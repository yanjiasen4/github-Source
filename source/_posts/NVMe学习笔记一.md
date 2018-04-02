---
title: NVMe学习笔记一
date: 2018-03-27 20:01:40
tags: [PCI-e, NVMe]
---

NVMe (Non-Volatile Memory Express) 是SSD与主机通信的一种新协议，它针对PCI-e和SSD的特性来设计，提高了主机对SSD的读写速度。最近开始进行NVMe虚拟化的研究，恶补了一波相关知识。

<!-- more -->

# 前言

硬盘的数据如何被CPU读取，用户的数据又是如何被写入到硬盘的。这一看似简单的动作，却经历了一段不简单的过程。硬盘设备和电脑主机的数据通信协议栈大致可以分为这么几层，硬盘在几层协议的帮助下才实现了与CPU的交互。

![figure](http://ohvmg8dgt.bkt.clouddn.com/layer.png)

在SSD刚刚问世的时候，只有下层的硬件变了，上层的协议包括接口都没来得及更新。这也就导致了读写更快、延迟更低的SSD只能走HDD使用的SATA接口和AHCI协议。后来一些SSD开始支持PCI-e接口，但是针对HDD指定的协议仍然严重制约了SSD的数据传输速率。于是，英特尔集合了许多厂商，为SSD制定了一套新的上层协议——NVMe。

![AHCI、NVMe对比](http://ohvmg8dgt.bkt.clouddn.com/AHCI-NVMe.png)

> AHCI协议中的队列深度，无法缓存的寄存器访问，中断，并行多线程等特性都远远不如NVMe协议

# 概述

NVMe (Non-Volatile Memory Express) 是SSD与主机通信的一种新协议，它针对PCI-e和SSD的特性来设计，提高了主机对SSD的读写速度。

# PCI-e

首先明确NVMe的目标设备：PCI-e接口的SSD设备。NVMe限定的PCI-e接口也是显卡一般会使用的接口，可以想象到它的速度会有多快。但是在PCI-e被应用到SSD之前，绝大多数硬盘都使用SATA接口，SATA也被叫做“串行ATA”(Serial ATA)，它拥有更快的速度和更小的体积，取代了PATA(Paralel ATA)，成为了硬盘接口的新标准。大家可能会疑惑，为什么串行的SATA会在各个方面优于并行的PATA。其实这和PCI-e替代PCI的原因类似：
> 并行线路的信号同步问题会限制传输的时钟周期大小

设时钟周期为t，并行线路同时能传输的数据量为n，则传输的速率v=t*v

- PCI: t=33MHz, n=4B, v=33MHz*4B=132MB/s
- PCI-e 1.0x1: t=2.5GHz, n=1b, v=2.5GHz*1b=312.5MB/s

当然我们没有考虑数据编码校验等带来的额外开销，但是也可以感受到PCI-e在1条lane的情况下就已经超过PCI。这主要就是因为PCI-e采用串行传输，拥有超高的时钟周期。

# NVMe总体设计

NVMe协议的大体框架和流程如下图所示
![figure](http://ohvmg8dgt.bkt.clouddn.com/NVMe-Structure.png)

Host就是搭载NVMe设备的主机，NVMe Controller则是NVMe设备上的控制器。在整个NVMe协议中，最重要的几个部分就是sq(Submission Queue)，cq(Completion Queue)和对应的db(Doorbell)。

其中，sq和cq处在host的内存空间中，而对应的db在NVMe设备控制器的寄存器空间中。sq和cq是环形队列，它们可能有很多个（根据不同设备的支持，最多可达64K），并且sq与cq的关系实在NVMe设备初始化的时候就确定的。一般来说，一个sq对应一个cq,但是也允许多个sq对应一个cq。

这两个队列负责host与NVMe设备的指令交换。host通过写sq传达新的指令，并通过写db告诉NVMe控制器。NVMe控制器根据db的值读取sq的指令，并将指令的完成情况写到cq上,触发一个终端,最后通过写db通知host。

除了sq和cq，在host的内存中还有一个admin queue，它负责host对NVMe的管理，包括对sq和cq（统称为IO queue）的初始化。

由于NVMe支持多大64K个IO队列，使得SSD的速度被发挥到极致。这是如此多的队列控制，也对host和NVMe控制器有较高的要求。另外，如何对大量db进行轮询，也是host面临的一大问题。
