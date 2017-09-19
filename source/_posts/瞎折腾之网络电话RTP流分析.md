---
title: 瞎折腾之网络电话RTP流分析
date: 2017-04-27 19:37:40
tags: [网络,网络电话,UDP,RTP,抓包,wireshark,scapy]
categories: Network
---

&#8195;&#8195;最近接到了一个要求分析VOIP网络电话通话质量的委托。之前一直不太了解网络电话，在公司实习的时候，虽然也配有网络电话，但也从来没用过。不过，牵扯到网络通讯，无外乎协议和包。花了不长的时间研究了这个陌生的领域，还是给了我在网络协议方面不少的启发。

<!-- more -->

# VoIP
VoIP(Voice over Internet Protocol)网络电话，将传统的电话迁移至网络。VoIP通过一系列协议，将发起通话、传输语音到结束通话等一系列过程，包装成一个个网络包，在网络上进行通信。

## SIP
SIP(Session Initiation Protocol)回话发起协议，正如其名，发起一个网络电话所使用的协议。并且网络电话的注册登录等也是使用这个协议完成的。

## RTP
网络电话的语音信息，被编码后以RTP流进网络传输。RTP是UDP上层的一个协议，不同于TCP，UDP的传输，不需要建立连接，适合连续传输大的流。而通话质量的分析，就是对传输语音的RTP流的分析。

## 使用wireshark观察网络电话流
初次接触网络电话、RTP等一些陌生的事物，不仅要从外部框架和理论上了解它们，还要从内部细节上去熟悉。如果我们能够看到一个网络电话从拨号到通话再到结束的全过程，那么就会对VoIP有一个直观的认识。[Wireshark](https://www.wireshark.org/)是一款对包进行封包分析的开源软件，使用它，可以轻松抓取各种协议的包并进行分析。  

当然还有另外一个问题，进行网络电话的情景也需要模拟。我们可以快速搭建一个网络电话框架，不过可能需要两个终端才能实现通话。Wireshark官网也有一些现成的数据，我们可以直接使用。  
[rtp_example_raw](https://wiki.wireshark.org/SampleCaptures?action=AttachFile&do=view&target=rtp_example.raw.gz)  

下载以后使用wireshark导入，就可以看到传说中的RTP包了  
![figure](http://ohvmg8dgt.bkt.clouddn.com/wiresharkRTP.jpg)
一个完整的VoIP流应该由SIP协议建立连接开始，再到RTP传输内容，这里我们只关注RTP流的分析。  

打开一个RTP包，可以看到它的层次结构  
![figure](http://ohvmg8dgt.bkt.clouddn.com/wiresharkRTPpkt.jpg)  
由最外层的网络层（以太），到IP、UDP再到最里面的RTP，一个包被分成了多层结构。RTP层里有有关音频流的一些信息，分别记录了RTP的版本，负载的格式，此包在流中的序列号、时间戳，和同步源标识，以及负载，也就是音频内容。在wireshark里，你可以看到包中每一位对应的信息。  

## RTP流分析
在wireshark上方菜单栏还有一个电话的RTP流分析选项，打开是这样的
![figure](http://ohvmg8dgt.bkt.clouddn.com/wiresharkRTPanalysis.jpg)  

可以看到，wireshark对RTP流分析得到了一些数据，包括
* delta(时延)
* jitter(抖动)
* skew(扭曲)  
* ......  
那么这些数据是怎么算出来的呢？  

作为一个程序猿，秉着能看代码就少读文档的思路（还好wireshark是开源的），我下载了wireshark源码，找到了这一部分的计算。  
```C++
// file: tap-rtp-common.c
void
rtp_packet_analyse(tap_rtp_stat_t *statinfo,
		       packet_info *pinfo,
		       const struct _rtp_info *rtpinfo)
{
    // ......
    /* Can only analyze defined sampling rates */
	if (clock_rate != 0) {
		statinfo->clock_rate = clock_rate;
		/* Convert from sampling clock to ms */
		nominaltime = nominaltime /(clock_rate/1000);

		/* Calculate the current jitter(in ms) */
		if (!statinfo->first_packet) {
			expected_time = statinfo->time + (nominaltime - statinfo->lastnominaltime);
			current_diff = fabs(current_time - expected_time);
			current_jitter = (15 * statinfo->jitter + current_diff) / 16;

			statinfo->delta = current_time-(statinfo->time);
			statinfo->jitter = current_jitter;
			statinfo->diff = current_diff;
		}
		statinfo->lastnominaltime = nominaltime;
		/* Calculate skew, i.e. absolute jitter that also catches clock drift
		 * Skew is positive if TS (nominal) is too fast
		 */
		statinfo->skew    = nominaltime - arrivaltime;
		absskew = fabs(statinfo->skew);
		if(absskew > fabs(statinfo->max_skew)){
			statinfo->max_skew = statinfo->skew;
		}
    }
}

```
时延比较简单，通过两个包发出的间隔就可以得出。而抖动的计算则稍微复杂。可以从源码看到，当前一帧的抖动，是上一帧抖动的15/16加上一个时间差。这个时间差就是理论上该包的到达时间与实际到达时间的差值的绝对值`current_diff`  

而理论到达时间的计算，又需要根据负载的类型，查询对应的时钟周期表，因为`timestamp`并非真正的时间戳，而是一个时钟周期。  
```C++
static guint32
get_clock_rate(guint32 key)
{
	size_t i;

	for (i = 0; i < NUM_CLOCK_VALUES; i++) {
		if (clock_map[i].key == key)
			return clock_map[i].value;
	}
	return 0;
}
//......
if (statinfo->pt < 96 ){
	clock_rate = get_clock_rate(statinfo->pt);
}
```
其实这个抖动的计算公式，是在rtp的[rfc1889](http://www.ietf.org/rfc/rfc1889.txt)中规定的:  

![figure](http://ohvmg8dgt.bkt.clouddn.com/rfc1889.jpg)  

# 实现一个网络电话监控分析的小程序
人生苦短，我用python。监控使用scapy，图形化界面用htmlPy+bootstrap+highCharts。  

## scapy
`scapy`是个可以侦听、解析甚至修改、发送网络报文的python库，使用它的`sniff`函数可以轻松抓取本机的指定网络包。  

需要注意的是，`scapy`支持了许多网络报文协议，其中就包括RTP，但是它不会自动把UDP中的RTP包解析出来，需要我们设置  

```python
bind_layers(UDP, RTP, sport=xxxx)
```

绑定指定`sport`（源端口）或者`dport`（目标端口）的所有UDP包，按照RTP包来解析。

## htmlPy
这是一个可以让你用python将网页内嵌在应用里做界面的库，官方文档不多但是足够了。

## 效果图
![figure](http://ohvmg8dgt.bkt.clouddn.com/7OPU8G~KOB%5BS$MLCSL~_~JG.png)
