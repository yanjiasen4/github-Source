---
title: Squid入门使用
date: 2017-04-12 19:15:23
tags: [proxy,CDN,网络]
categories: Web
---

![pic](http://ohvmg8dgt.bkt.clouddn.com/img4.jpg)  
Squid是一款开源的代理缓存服务器，可以作为个人代理使用，也可以用来作为搭建CDN的重要组件。最近在试着自己实现一个类似CDN功能的网络，所以对Squid做了一些了解。

<!-- more -->

# 安装
Squid的多平台支持还不错，linux、CentOS和Windows都能安装运行，并且使用方法基本相同。
## Linux/CentOS
可以通过包管理工具直接安装  
1. Linux  
`sudo apt-get install squid`  
2. CentOS  
`yum install squid`  

但是通过这种方式安装的可能不是最新版本，如果想要安装最新版本的Squid，可以在官网选择指定版本的安装包，或是下载源码编译的方式  
[squid安装包下载](http://wiki.squid-cache.org/SquidFaq/BinaryPackages)

## Windows
下载windows安装程序进行安装  
[squid for windows](http://squid.acmeconsulting.it/)  

# 配置
## 指南
(squid官网文档)[http://www.squid-cache.org/Doc/config/]有所有config文件指令的解释，但是有些解释还不够详细。

## 默认配置解读
安装成功以后，squid会用默认的配置文件自动运行，但是想要达到我们想要的效果，还需要自己对squid进行配置。以下以Ubuntu16系统为例：  

squid的配置文件在`\etc\squid\squid.conf`中，打开以后看到一些配置信息。
```
#
# Recommended minimum configuration:
#

# Example rule allowing access from your local networks.
# Adapt to list your (internal) IP networks from where browsing
# should be allowed
acl localnet src 10.0.0.0/8	# RFC1918 possible internal network
acl localnet src 172.16.0.0/12	# RFC1918 possible internal network
acl localnet src 192.168.0.0/16	# RFC1918 possible internal network
acl localnet src fc00::/7       # RFC 4193 local private network range
acl localnet src fe80::/10      # RFC 4291 link-local (directly plugged) machines

acl SSL_ports port 443
acl Safe_ports port 80		# http
acl Safe_ports port 21		# ftp
acl Safe_ports port 443		# https
acl Safe_ports port 70		# gopher
acl Safe_ports port 210		# wais
acl Safe_ports port 1025-65535	# unregistered ports
acl Safe_ports port 280		# http-mgmt
acl Safe_ports port 488		# gss-http
acl Safe_ports port 591		# filemaker
acl Safe_ports port 777		# multiling http
acl CONNECT method CONNECT
```  

一堆以`acl`开头的声明，`acl`全称是`Access Control List`，它可以让你通过不同的方式，定义一些组，为下面的访问控制提供方便。比如默认的配置文件中，将请求按照端口号分为了不同的组。 

```
#
# Recommended minimum Access Permission configuration:
#
# Deny requests to certain unsafe ports
http_access deny !Safe_ports

# Deny CONNECT to other than secure SSL ports
http_access deny CONNECT !SSL_ports

# Only allow cachemgr access from localhost
http_access allow localhost manager
http_access deny manager

# We strongly recommend the following be uncommented to protect innocent
# web applications running on the proxy server who think the only
# one who can access services on "localhost" is a local user
#http_access deny to_localhost

#
# INSERT YOUR OWN RULE(S) HERE TO ALLOW ACCESS FROM YOUR CLIENTS
#

# Example rule allowing access from your local networks.
# Adapt localnet in the ACL section to list your (internal) IP networks
# from where browsing should be allowed
http_access allow localnet
http_access allow localhost

# And finally deny all other access to this proxy
http_access deny all

# Squid normally listens to port 3128
http_port 3128
```  

下面是一些`http_access`的条目，这种指令指定了接收或是拒绝某一种请求。例如  
```
http_access deny !Safe_ports
```  
这一条就是拒绝所有不是Safe_ports类型的请求，而Safe_ports就是上面的`acl`指令定义的那些通过特定端口的请求。  

但是要注意一点的是，squid在运行时，对请求的过滤判断是根据配置文件的配置，由上至下的，如果上面的某条规则匹配成功，则按照这条指令对请求进行接受或拒绝。所以可以看到配置文件`http_access`的最后一条是`deny all`。  

最后一行的`http_port`规定了squid在3128端口进行监听。  

```
# Uncomment and adjust the following to add a disk cache directory.
#cache_dir ufs /var/cache/squid 100 16 256

# Leave coredumps in the first cache dir
coredump_dir /var/cache/squid

#
# Add any of your own refresh_pattern entries above these.
#
refresh_pattern ^ftp:		1440	20%	10080
refresh_pattern ^gopher:	1440	0%	1440
refresh_pattern -i (/cgi-bin/|\?) 0	0%	0
refresh_pattern .		0	20%	4320
```  

被注释的一行指定了squid的磁盘缓存配置，默认是关闭只使用内存缓存的。  

下面一行的`coredump_dir`是squid出错时的核心转储存放位置。  

最后一行的`refresh_pattern`指定了相应内容的缓存刷新规则，四个参数分别为
```
regex min percent max [option]
```  
具体的刷新方法比较复杂，可以在源码的`refresh.cc`中找到使用方法。  

## 配置一个反向代理
squid跟varnish、nginx等通常被作为CDN的部署组件使用，下面我们就看一下如何使用squid配置一个反向代理。  

找到官网的[Example](http://wiki.squid-cache.org/ConfigExamples/Reverse/BasicAccelerator)。将squid的监听窗口改为80，覆盖默认的http端口，并且设置`accel`模式和指定默认的访问网站。
```
http_port 80 accel defaultsite=yanjiasen4.tech no-vhost
```  

下面设置缓存应该从哪里找到源服务器。
```
cache_peer 120.24.71.4 parent 80 0 no-query originserver name=myAccel
```  

然后设置访问控制
```
acl our_sites dstdomain yanjiasen4.tech
http_access allow our_sites
cache_peer_access myAccel allow our_sites
cache_peer_access myAccel deny all
```  

设置完成以后重启squid
```
cd \etc\init.d
./squid restart
```  

访问`localhost:80`，发现进入了自己设置的网站`yanjiasen4.tech`。  

再将DNS的`yanjiasen4.tech`解析A记录设置为squid的ip地址，使用浏览器或用`curl url -I`指令访问加速网站`yanjiasen4.tech`。可以看到返回的包中多了一些关于squid的信息  
![figure](http://ohvmg8dgt.bkt.clouddn.com/QQ%E6%88%AA%E5%9B%BE20170413194300.jpg)  
再查看squid的log文件，可以确认反向代理设置成功 
```
vim \var\log\squid\access.log
```  

```
1492084035.268     96 ::1 TCP_MISS/200 7593 GET http://cdnbroker.tech/ - FIRSTUP_PARENT/120.24.71.4 text/html
```  
清理浏览器缓存以后再次访问，可以看到，一些对象的`X-Cache`属性已经变为了HIT，但是一些对象仍然是MISS。这就和网站服务器的一些设置有关了。
![figure1](http://ohvmg8dgt.bkt.clouddn.com/QQ%E6%88%AA%E5%9B%BE20170413195343.jpg)  
```
1492084189.003    108 ::1 TCP_REFRESH_MODIFIED/200 7593 GET http://cdnbroker.tech/ - FIRSTUP_PARENT/120.24.71.4 text/html
......
1492084192.386      0 ::1 TCP_MEM_HIT/200 8232 GET http://cdnbroker.tech/images/avatar.jpg - HIER_NONE/- image/jpeg
```