---
title: 浅谈P2P与CDN
date: 2017-03-30 16:17:09
tags: [P2P,CDN,网络]
categories: Web
---

&#8195;&#8195;最近在调查学习CDN领域时，发现新兴CDN提供商大多对P2P情有独钟。有些以此为卖点，大力推销倡导；有些则用P2P作为产品的调料，为自己增色。P2P到底在CDN领域中扮演怎样的角色，能否在愈加激烈的CDN市场竞争中为提供商们带来优势吗？

<!-- more -->

# 引言
经常使用下载工具的人一定对P2P有印象。BitComet、电驴（eMule）以及在国内大名鼎鼎的迅雷，都算是P2P下载的知名软件。P2P(Peer-to-peer)的概念最早在1969的一份RFC文档中被提出，而它被用来做下载工具则是在2000年左右。这种去中心化的下载方式，能给用户带来极大的下载速度提升。P2P改变了传统网络内容提供商中心化服务器的结构，消除了服务器单一网络出入口的瓶颈，并且通过对文件的分片，大大提高了下载效率。  
而作为网络基础设施中一个很重要的部分——CDN，也越来越需要一些新的方式来提高自己的性能和性价比。P2P就是很多人想到的方法之一。

# CDN厂商
## 传统CDN
传统的CDN提供商如Akamai、Limelight国内的网速、蓝汛等等，在CDN市场已经纵横许多年，这种CDN的特点就是专业可靠。但同时也就意味着他们的价格普遍较高且一般不透明，主要针对大客户定制服务。  
![figure](http://ohvmg8dgt.bkt.clouddn.com/094S563I-4.png)  
CDN通过拉近与末端用户的地理距离来提高服务质量，这也就意味着一个可靠的CDN服务商，需要在世界各地拥有自己的节点，这给CDN厂商带来了较高的入门门槛。传统CDN厂商经历了多年的基础设施、技术和用户数据积累，在市场中占据主导位置也不足为奇。而正因为如此，传统CDN在很长一段时间在市场中占据近乎垄断的地位，其过高的定价却让许多中小型用户望而生畏。

## 云平台CDN
传统CDN主要针对网页静态资源、下载等类型请求分发，而近些年，随着网络地快速发展，普通用户的网络带宽大大提高，视频、直播等高流量的请求开始爆发增长。据统计，视频资源的流量已经达到了互联网流量的80%。而进入互联网行业做内容提供商也不再是大公司的专属，许多创业公司开始了自己的互联网之旅。传统CDN的高定价已经无法适应新的市场环境，而有了充足资源积累的一些互联网公司，将自己的资源与“云”这一概念融合，将云平台CDN开放给市场使用。  
云平台CDN相比传统CDN，价格较为低廉，且定价体系明确透明。而踏入CDN领域的云平台服务商，基本都是互联网行业的巨头：腾讯、阿里、百度、亚马逊等等，他们充分的资源积累也保证了其CDN的质量。而这些巨头的这一举动，也使得他们从传统CDN提供商的用户，变为了竞争对手，给CDN市场带来了新的革命。  

## 创新型专业CDN
不同于上面的云平台CDN，是由有一定资源和数据积累的互联网大公司开发的CDN服务，创新型专业CDN提供商希望通过新的技术、新的体系，参与到CDN市场的竞争中来。其中最典型的例子有：网心的“星域CDN”和云帆加速的“云帆CDN”。这两款CDN其实也颇有来历，星域其实源于迅雷的“赚钱宝”；而“云帆CDN”则有快播的技术前身。相信熟悉网络技术的读者已经发现，这两款CDN的前身，都与P2P有紧密的关系。迅雷和快播都是依靠自身强大的P2P技术，为用户提供高质量低延迟的服务。


# CDN融合P2P

# CDN市场调查

# 参考资料