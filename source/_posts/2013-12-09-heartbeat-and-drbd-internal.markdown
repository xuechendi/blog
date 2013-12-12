---
layout: post
title: "Heartbeat and DRBD internal"
date: 2013-12-09 10:20
comments: true
categories: Linux
---
This posting is talking about Heartbeat & DRBD internal

Every high−availability system needs two basic services: to be informed of cluster members joining and leaving the system, and to provide basic communication services for managing a cluster.

Below info is quoted from <a href="http://upcommons.upc.edu/pfc/bitstream/2099.1/4970/1/memoria.pdf" target="_blank">MASTER THESIS of Abraham Iglesias Aparicio</a>

##The Linux-HA project: Heartbeat

Linux-ha project (Linux High Availability project) [5] is an open source suite to build high available clusters. It can run on every known Linux flavor and in some Unices such as FreeBSD and Solaris.
Heartbeat software is the core component of the Linux-HA project and permits building high availability clusters from commodity servers, as it has no hardware dependencies.

The project started in 1998 and is maintained by an open developer’s community. The project is lead by some companies like Novell or IBM. 10 years of development, testing and more than 30.000 production clusters are numbers that would reflect project maturity.

###Heartbeat cluster styles
There are two different scopes when working with Heartbeat. Two styles of clusters can be configured. Let us say R1 cluster style and R2 cluster style.
R1 cluster style was the first kind of configurations that could be done with Heartbeat. In this configuration there were some limitations such as:
#####1. Limitation of nodes in the cluster (it can only accept 2 nodes)
#####2. Cannot perform monitor operations of a cluster resource
#####3. There were almost no options to express dependency information
<br>
First version of this R2 cluster style was 2.0.0. At the time of writing this document, Heartbeat latest stable version was 2.0.8. Therefore, this version will be the reference of study in this project. R2 is fully compatible with R1 cluster style configuration. It supports up to 16 nodes cluster and an asynchronous logging daemon was added. Moreover, there were some improvements on message architecture.
With R2 cluster style, or next generation of Heartbeat, these limitations were overridden and software architecture changed. Adding more components and functionalities lead to a more complex system but more complete than the R1 cluster style.
Some new components appeared with R2:
#####1. ClusterInformationBase
#####2. ClusterResourceManager
#####3. Modular PolicyEngine
#####4. LocalResourceManager
#####5. StonithDaemon
<br>
####ClusterInformationBase
ClusterInformationBase (aka CIB) is the component which stores information about the cluster. It is replicated on every node and stores two kind of information:
#####1. Static information. It includes definitions of cluster nodes, resources, monitor operations and constraints.
#####2. Dynamic information. CIB stores the current status of the cluster.
<br>
CIB is formatted in a XML document and must be built following DTD document included with Heartbeat specifications. Cib.xml files can be red in annex 2.1.

####ClusterResourceManager
The ClusterResourceManager (CRM) component consists on the following components:
#####1. ClusterInformationBase
#####2. ClusterResourceManagerDaemon
#####3. PolicyEngine
#####4. Transitioner
#####5. LocalResourceManager
<br>

The ClusterResourceManagerDaemon (CRMD) runs on every node and coordinates the actions of all other cluster resource managers. It exchanges information with the designated coordinator (DC) and it can be seen as a communication interface between DC and subsystem components such as CIB and LocalResourceManager.

The DC is a special instance of CRMD. It is elected with the help of the list of cluster nodes from the ClusterConsensusManager (CCM) and takes on the responsibilities of ResourceAllocation throughout the cluster. Therefore, DC is the one who delivers CIB information to all slave instances of CRMD.

The PolicyEngine (PE) module is the one who performs computation of the next state in the cluster. It is triggered by any event in the cluster, such as a resource changing state, a node leaving or joining, or CRM event such as a CRM parting or leaving. The PE, takes the input from CIB and runs locally and does not do any network I/O, it only runs an algorithm. It describes the actions and their dependencies necessary to go from the current cluster state to the target cluster state.
When PE returns an action it is passed to the Transitioner. The goal of this component is to make PE’s wishes become true. The new state computed by PE is passed to the Transitioner, which communicates with the LocalResourceManager on every node and informs about the actions decided by PE (start, stop resources).

###ClusterConsensusManager
The ClusterConsensusManager (CCM) is the one who establishes membership in the cluster. This membership layer is in the middle of Heartbeat, which is the messaging and infrastructure layer, and the new CRM which is the resource allocation layer. CCM information includes new additions to the membership, lost nodes and loss of quorum.

###Stonith daemon
R2 cluster style features include stonith capabilities. Stonith is an acronym meaning “Shot The Other Node In The Head”.
A stonith daemon is running on every node and receives requests from the CRM’s TE about what and when to reset. An example of when stonith daemon action is used is in a split brain situation. The DC will send a message to the Stonith daemon to reset the sick node. This node will reboot and then will join the cluster again in a healthy way.

###LocalResourceManager
Finally, the LocalResourceManager (LRM) has the responsibility for performing operations on resources, by using ResourceAgent scripts to carry out the work. ResourceAgent scripts define a set of start, stop or monitor operations, as directed by the CRM, which are used to operate on a resource instance.
LRM also provides information about resources. It can provide a list of resources that are currently primary and its current state.

###Configuration and manageability
Heartbeat GUI is a user interface to manage Heartbeat clusters. It is delivered in a separate package of Heartbeat core. It is an application developed with python and is able to add, remove nodes or cluster resources.
It is necessary to have connectivity to port 5560/tcp of the cluster node and password for hacluster user running on the cluster nodes.
Heartbeat GUI is executed from a shell and after being authenticated, the main screen of Heartbeat GUI appears. This is how GUI looks like:

![heartbeat_gui]( {{ root_url }}/images/HA/heartbeat_gui.png)

##HA overview 

![heartbeat_gui]( {{ root_url }}/images/HA/pcmk-overview.png)

##HA(Pacemaker) Internal Components
Pacemaker itself is composed of four key components (illustrated below in the same color scheme as the previous diagram):

#####Heartbeat：将原来的消息通信层独立为heartbeat项目，新的heartbeat只负责维护集群各节点的信息以及它们之前通信；
#####Cluster Glue：相当于一个中间层，它用来将heartbeat和pacemaker关联起来，主要包含2个部分，即为LRM和STONITH。
#####Resource Agent：用来控制服务启停，监控服务状态的脚本集合，这些脚本将被LRM调用从而实现各种资源启动、停止、监控等等。
#####Pacemaker:也就是Cluster Resource Manager （简称CRM），用来管理整个HA的控制中心，客户端通过pacemaker来配置管理监控整个集群。
#####Corosync:a Group Communication System， helps Pacemaker to support popular open source cluster filesystems.
	what is Corosync:
		Corosync包含OpenAIS的核心框架用来对Wilson的标准接口的使用、管理。它为商用的或开源性的集群提供集群执行框架。Corosync执行高可用应用程序的通信组系统，它有以下特征：
		一个封闭的程序组（A closed process group communication model）通信模式，这个模式提供一种虚拟的同步方式来保证能够复制服务器的状态。
		一个简单可用性管理组件（A simple availability manager），这个管理组件可以重新启动应用程序的进程当它失败后。
		一个配置和内存数据的统计（A configuration and statistics in-memory database），内存数据能够被设置，回复，接受通知的更改信息。
		一个定额的系统（A quorum  syste）,定额完成或者丢失时通知应用程序。

 
![heartbeat_gui]( {{ root_url }}/images/HA/pcmk-stack.png)

#####1. CIB (aka. Cluster Information Base)
The CIB uses XML to represent both the cluster’s configuration and current state of all resources in the cluster. The contents of the CIB are automatically kept in sync across the entire cluster and are used by the PEngine to compute the ideal state of the cluster and how it should be achieved.

#####2. CRMd (aka. Cluster Resource Management daemon)
This list of instructions is then fed to the DC (Designated Co-ordinator). Pacemaker centralizes all cluster decision making by electing one of the CRMd instances to act as a master. Should the elected CRMd process, or the node it is on, fail… a new one is quickly established.

#####3. PEngine (aka. PE or Policy Engine)
The DC carries out the PEngine’s instructions in the required order by passing them to either the LRMd (Local Resource Management daemon) or CRMd peers on other nodes via the cluster messaging infrastructure (which in turn passes them on to their LRMd process).

The peer nodes all report the results of their operations back to the DC and based on the expected and actual results, will either execute any actions that needed to wait for the previous one to complete, or abort processing and ask the PEngine to recalculate the ideal cluster state based on the unexpected results.

#####4. STONITHd
In some cases, it may be necessary to power off nodes in order to protect shared data or complete resource recovery. For this Pacemaker comes with STONITHd. STONITH is an acronym for Shoot-The-Other-Node-In-The-Head and is usually implemented with a remote power switch. In Pacemaker, STONITH devices are modeled as resources (and configured in the CIB) to enable them to be easily monitored for failure, however STONITHd takes care of understanding the STONITH topology such that its clients simply request a node be fenced and it does the rest.


![heartbeat_gui]( {{ root_url }}/images/HA/pcmk-internals.png)


A heartbeat is a periodic communication attempt to check for updated policies. 

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

###heartbeat.conf

#####心跳方式：ucast,mcast,serial
#####debugfile /var/log/ha-debug:该文件保存 heartbeat 的调试信息
#####logfile /var/log/ha-log:heartbeat 的日志文件
#####keepalive 2:心跳的时间间隔,默认时间单位为秒
#####deadtime 30:超出该时间间隔未收到对方节点的心跳,则认为对方已经死亡。
#####warntime 10:超出该时间间隔未收到对方节点的心跳,则发出警告并记录到日志中。
#####initdead 120:在某些系统上,系统启动或重启之后需要经过一段时间网络才能正常工作,
#####该选项用于解决这种情况产生的时间间隔。取值至少为 deadtime 的两倍。
#####udpport 694:设置广播通信使用的端口,694 为默认使用的端口号。
#####baud 19200:设置串行通信的波特率。
#####serial /dev/ttyS0:选择串行通信设备,用于双机使用串口线连接的情况。如果双机使用以太网连接,则应该关闭该选项。
#####bcast eth0:设置广播通信所使用的网络接口卡。
#####auto_failback on:heartbeat 的两台主机分别为主节点和从节点。主节点在正常情况下占用资源并运行所有的服务,遇到故障时把资源交给从节点并由从节点运行服务。在该选项设为 on 的情况下,一旦主节点恢复运行,则自动获取资源并取代从节点,否则不取代从节点。
#####ping ping-node1 ping-node2:指定 ping node,ping node 并不构成双机节点,它们仅仅用来测试网络连接。
#####respawn hacluster /usr/lib/heartbeat/ipfail:指定与 heartbeat 一同启动和关闭的进程,该进程被自动监视,遇到故障则重新启动。最常用的进程是 ipfail,该进程用于检测和处理网络故障,需要配合ping 语句指定的 ping node 来检测网络连接。
