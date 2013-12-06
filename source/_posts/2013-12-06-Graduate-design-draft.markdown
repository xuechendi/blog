---
layout: post
title: "Graduate design draft"
date: 2013-12-06 11:55
comments: true
categories: Storage 
---
##The paper contents:
	1.overview
	2.Cache and Consistency(ceph, vertical and horizonal)
	3.The atomic transaction(leveldb internal)
	4.The design and architecture
	5.The implementation
	6.The evaluation
	7.The wanna do(cache for vm snapshot?)
	



###Coherence: Read the right data.
###Consistency: Write at the right sequence and location, to garantee the right reading data.

##How to achieve cache consistency?

{% blockquote %}
所有的一致性方案都要求通过某种方式来实现同一CACHE块的<b>串行化</b>访问，不论是通过串行化对通信媒介的访问，还是通过串行化对其他共享结构的访问。
{% endblockquote %}

Consistency only can be acheived by maintain the sequentialty of accessing devices. 

##Vertical data consistency and Horizonal data consistency

###Vertical data consistency: Require data consistency among memory, persistency cache device(ssd), and also storage backend(ceph cluster).
	1. How ceph cache pool plan to design and architect.
	2. System register, memory design.
	3. Other Tiering storage.

###Horizonal data consistency: When the same backend attach to different VM and even different VM on diffent host, how to keep all cache being data consistent and coherent.
	1. Splay? Ceph Metadata subtree updating mechanism?.
	2. Multi core system with shared memory design.
	3. ???
