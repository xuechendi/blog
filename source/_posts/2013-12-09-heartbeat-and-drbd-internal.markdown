---
layout: post
title: "My Understanding of High Availability"
date: 2013-12-09 10:20
comments: true
categories: [Linux, HA]
---
> Get known of HA, that's what I was been requested week ago, and I just thought what I need to do is knowing how to use something called heartbeat and pacemaker, maybe knowing the protocol and model underneath these two projects. Then you definitely get what happened, so many nouns pop out, "corosync", "CMAN", "OpenAIS", "rgmanager", and protocols like "SRP", "RRP", "quiescent algorithms", "termination detection algorithms", etc.

>I *TRIED REALLY HARD* here to figure out the relationship among all these nouns, and what gives me a relief is that I just found this posting in <a href="http://schlutech.com/2011/07/demystifying-high-availability-linux-clustering-technologies/" target="_blank" ><code>which</code> some guy gets my feeling </a>. ^_^ , Just dig dig dig...

>I put everything in the way I thought will be much clear <code>below in the SLIDES</code>, and will update in following days

{% reveal /slides/get-known-of-ha 100 800 %}


<h2>Referrence</h2>
<h6><a href="http://upcommons.upc.edu/pfc/bitstream/2099.1/4970/1/memoria.pdf" target="_blank">1. MASTER THESIS of Abraham Iglesias Aparicio</a></h6>
<h6><a href="http://www.linux-ha.org/wiki/Documentation" target="_blank">2. The Linux-HA project: Heartbeat</a></h6>
<h6><a href="http://hi.baidu.com/leolance/item/26e1943543e873342e0f8142" target="_blank">3. http://hi.baidu.com/leolance/item/26e1943543e873342e0f8142</a></h6>
<h6><a href="http://www.ultramonkey.org/3/linux-ha.html" target="_blank">4. http://www.ultramonkey.org/3/linux-ha.html</a></h6>
<h6><a href="http://doc.opensuse.org/products/draft/SLE-HA/SLE-ha-guide_sd_draft/book.sleha.html" target="_blank">5. SUSE Linux Enterprise High Availability Extension High Availability Guide</a></h6>
<h6><a href="http://linux-ha.org/_cache/TechnicalPapers__UKUUG-WinterConf-2004-SCRAT-Paper.pdf" target="_blank">6. 《A new Cluster Resource Manager for heartbeat》Writen by SUSE</a></h6>
