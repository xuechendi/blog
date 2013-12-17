---
layout: post
title: "Heartbeat and DRBD internal"
date: 2013-12-09 10:20
comments: true
categories: Linux
---
This posting is talking about Heartbeat & DRBD internal

Every high−availability system needs two basic services: to be informed of cluster members joining and leaving the system, and to provide basic communication services for managing a cluster.


{% reveal /slides/get-known-of-ha 100 800 %}


<h2>Referrence</h2>
<h6><a href="http://upcommons.upc.edu/pfc/bitstream/2099.1/4970/1/memoria.pdf" target="_blank">1. MASTER THESIS of Abraham Iglesias Aparicio</a></h6>
<h6><a href="http://www.linux-ha.org/wiki/Documentation" target="_blank">2. The Linux-HA project: Heartbeat</a></h6>
<h6><a href="http://hi.baidu.com/leolance/item/26e1943543e873342e0f8142">3. http://hi.baidu.com/leolance/item/26e1943543e873342e0f8142</a></h6>
<h6><a href="http://www.ultramonkey.org/3/linux-ha.html">4. http://www.ultramonkey.org/3/linux-ha.html</a></h6>
<h6><a href="http://doc.opensuse.org/products/draft/SLE-HA/SLE-ha-guide_sd_draft/book.sleha.html">5. SUSE Linux Enterprise High Availability Extension High Availability Guide</a></h6>



Heartbeat仅仅是个HA软件，它仅能完成心跳监控和资源接管，不会监视它控制的资源或应用程序。要监控资源和应用程序是否运行正常，必须使用第三方的插件，例如ipfail、Mon和Ldirector等。Heartbeat自身包含了几个插件，分别是ipfail、Stonith和Ldirectord，介绍如下。

ipfail的功能直接包含在Heartbeat里面，主要用于检测网络故障，并做出合理的反应。为了实现这个功能，ipfail使用ping节点或者ping节点组来检测网络连接是否出现故障，从而及时做出转移措施。Linux内核不仅为各种不同类型的watchdog硬件电路提供了驱动。

Stonith插件可以在一个没有响应的节点恢复后，合理接管集群服务资源，防止数据冲突。当一个节点失效后，会从集群中删除。如果不使用Stonith插件，那么失效的节点可能会导致集群服务在多于一个节点运行，从而造成数据冲突甚至是系统崩溃。因此，使用Stonith插件可以保证共享存储环境中的数据完整性。

Ldirector是一个监控集群服务节点运行状态的插件。Ldirector如果监控到集群节点中某个服务出现故障，就屏蔽此节点的对外连接功能，同时将后续请求转移到正常的节点提供服务。这个插件经常用在LVS负载均衡集群中。关于Ldirector插件的使用，将在后续章节详细讲述。

针对系统自身出现问题导致heartbeat无法监控的问题，就需要在Linux内核中启用一个叫watchdog的模块。watchdog是一个Linux内核模块，它通过定时向/dev/watchdog设备文件执行写操作，从而确定系统是否正常运行。如果watchdog认为内核挂起，就会重新启动系统，进而释放节点资源。

在Linux中完成watchdog功能的软件叫softdog。softdog维护一个内部计时器，此计时器在一个进程写入/dev/watchdog设备文件时更新。如果softdog没有看到进程写入/dev/watchdog文件，就认为内核可能出了故障。watchdog超时周期默认是一分钟，可以通过将watchdog集成到Heartbeat中，从而通过Heartbeat来监控系统是否正常运行。

####Linux下watchdog的工作原理

Watchdog在实现上可以是硬件电路也可以是软件定时器，能够在系统出现故障时自动重新启动系统。在Linux 内核下, watchdog的基本工作原理是：当watchdog启动后(即/dev/watchdog 设备被打开后)，如果在某一设定的时间间隔内/dev/watchdog没有被执行写操作, 硬件watchdog电路或软件定时器就会重新启动系统。

/dev/watchdog 是一个主设备号为10， 从设备号130的字符设备节点。 Linux内核不仅为各种不同类型的watchdog硬件电路提供了驱动，还提供了一个基于定时器的纯软件watchdog驱动。 驱动源码位于内核源码树drivers\char\watchdog\目录下。

####硬件与软件watchdog的区别

硬件watchdog必须有硬件电路支持, 设备节点/dev/watchdog对应着真实的物理设备， 不同类型的硬件watchdog设备由相应的硬件驱动管理。软件watchdog由一内核模块softdog.ko 通过定时器机制实现，/dev/watchdog并不对应着真实的物理设备，只是为应用提供了一个与操作硬件watchdog相同的接口。

硬件watchdog比软件watchdog有更好的可靠性。 软件watchdog基于内核的定时器实现，当内核或中断出现异常时，软件watchdog将会失效。而硬件watchdog由自身的硬件电路控制, 独立于内核。无论当前系统状态如何，硬件watchdog在设定的时间间隔内没有被执行写操作，仍会重新启动系统。

一些硬件watchdog卡如WDT501P 以及一些Berkshire卡还可以监测系统温度，提供了 /dev/temperature接口。 对于应用程序而言, 操作软件、硬件watchdog的方式基本相同：打开设备/dev/watchdog, 在重启时间间隔内对/dev/watchdog执行写操作。即软件、硬件watchdog对应用程序而言基本是透明的。

在任一时刻， 只能有一个watchdog驱动模块被加载，管理/dev/watchdog 设备节点。如果系统没有硬件watchdog电路，可以加载软件watchdog驱动softdog.ko。


When heartbeat is configured, a master node is selected. When heartbeat starts up this node sets up an interface for a virtual IP address, that will be accessed by external end users. If this node fails then another node in the heartbeat cluster will start up an interface for this IP address and use gratuitous ARP to ensure that all traffic bound for this address is received by this machine. This method of fail-over is called IP Address Takeover. Unless the auto_failback directive is set to off in the ha.cf file, once the master node becomes available again resources will fail-over again so they are once again owned by the master node.

Each virtual IP address is considered to be a resource. Resources are encapsuated as programes that work similarly to unix init scripts, that is the resource can be started and stopped, and its can be asked if it is running or not. In this way ~eartbeat is able to start and stop resources depending on the status of the other node that it is communicating with using the heartbeat-protocol. As the resources are encapsulated as programmes, arbitary resources may be used.

