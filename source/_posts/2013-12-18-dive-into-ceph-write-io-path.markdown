---
layout: post
title: "Dive into Ceph Write IO Path"
date: 2013-12-18 14:41
comments: true
categories: Ceph
---

*After talking with a friend of mine who is really intelligent and awesome and intelligent, just thinking about to post this article to get through the Ceph OSD Write IO path. There I am sorry to say must exists some misunderstanding in this article, but I hope it will still means something for people who read this article...*

*Ok, start here.*

======

There is a IO path graph on ceph.com, pls click <code>[OSD Internals](http://ceph.com/docs/master/dev/osd_internals/osd_throttles/?highlight=wbthrottle)</code>, I find the graph is a bit difficult to see, so redraw it in a vertical way

<div style="float:left;">
<div style="float:left; width:58%;">
	<div style="border-left:red solid 5px; padding-left:5%; ">
	<p>**Pipe::Reader** call the function DispatchQueue::enqueue, put the msg into mqueue</p>
	<p>**DispatchThread** loops in getting msg from mqueue and dispatch the msg to who is able to handle( msgr->ms_deliver_dispatch(m), in this case, OSD::ms_dispatch(m) will be called then )<p>
	</div>
	<br>
	<div style="border-left:green solid 5px; padding-left:5%;">
	<p>**OSD::_dispatch** switch (msg->type), in this case, would be "CEPH_MSG_OSD_OP", call handle_op(op) </p>
	<p>**handle_op** do some preparation before actually applying this op, includes refresh the map, check the obj name and data, remaining space, then it cal the pgid and lock the pg, call enqueue_op to put the op into osd->op_wq with pgid and op. </p>
	<p>**worker wq** ThreadPool::worker loops all the time to join_old_threads, then dequeue any items in the workqueue, by now, that would be our write op. Then work_queue start to process this op request by OSD::OpWQ::_process function, then OSD::dequeue_op function, and do_request. After dequeue, the pg lock is then released.</p>
	</div>
</div>

<img src="{{ root_url }}/images/ceph_write_path/io_path_1.png" style="margin-left:5%; width:35%; border:5px solid #222; float:left;">
</div>
<br>
<div style="width:100%; float:left; border-left:#00FFFF solid 5px; padding-left:5%; font-size:10px;">
	<p>**do_request** Seems it calls the do_request implemented in ReplicatedPG.cc, then calls do_op(op) also implemented in ReplicatedPG.cc.</p>
	<p>**do_op** 
</p>
</div>
<br>
<br>
Below are the logs :

<div style="float:left; font-size:12px;  border: solid #999 1px; background-color: #eee; width:100%; overflow: scroll; padding:5px; white-space:nowrap;text-overflow:scroll;">
<br>root@MPXDELL2:~# tail -f /var/log/ceph/osd.15.log  | grep "dispatch\|queue_op\|<red>worker</red>"
<br>2013-12-18 19:42:51.999692 7f2c9e790700 20 osd.15 267 <red>_dispatch</red> 0x23d6000 osd_op(client.5722.0:4 rbd_header.10126b8b4567 [call rbd.get_size,call rbd.get_object_prefix] 3.fa956e77 e267) v4
<br>2013-12-18 19:42:51.999748 7f2c9e790700 15 osd.15 267 <red><red>enqueue_op</red></red> 0x2aa9bc40 prio 63 cost 39 latency 0.000151 osd_op(client.5722.0:4 rbd_header.10126b8b4567 [call rbd.get_size,call rbd.get_object_prefix] 3.fa956e77 e267) v4
<br>2013-12-18 19:42:51.999814 7f2c8d76e700 12 OSD::op_tp <red>worker</red> wq OSD::OpWQ start processing 0x1 (1 active)
<br>2013-12-18 19:42:51.999824 7f2c8d76e700 10 osd.15 267 <red>dequeue_op</red> 0x2aa9bc40 prio 63 cost 39 latency 0.000228 osd_op(client.5722.0:4 rbd_header.10126b8b4567 [call rbd.get_size,call rbd.get_object_prefix] 3.fa956e77 e267) v4 pg pg[3.e77( v 267'732 (0'0,267'732] local-les=267 n=2 ec=15 les/c 267/267 266/266/266) [15] r=0 lpr=266 mlcod 267'732 active+clean]
<br>2013-12-18 19:42:52.000519 7f2c8d76e700 10 osd.15 267 <red>dequeue_op</red> 0x2aa9bc40 finish
<br>2013-12-18 19:42:52.000535 7f2c8d76e700 15 OSD::op_tp <red>worker</red> wq OSD::OpWQ done processing 0x1 (0 active)
<br>2013-12-18 19:42:52.001161 7f2c9e790700 20 osd.15 267 <red>_dispatch</red> 0x23d6480 osd_op(client.5722.0:5 rbd_header.10126b8b4567 [call rbd.get_stripe_unit_count] 3.fa956e77 e267) v4
<br>2013-12-18 19:42:52.001188 7f2c9e790700 15 osd.15 267 <red><red>enqueue_op</red></red> 0x2aa9bb60 prio 63 cost 24 latency 0.000107 osd_op(client.5722.0:5 rbd_header.10126b8b4567 [call rbd.get_stripe_unit_count] 3.fa956e77 e267) v4
<br>2013-12-18 19:42:52.001213 7f2c8af69700 12 OSD::op_tp <red>worker</red> wq OSD::OpWQ start processing 0x1 (1 active)
<br>2013-12-18 19:42:52.001230 7f2c8af69700 10 osd.15 267 <red>dequeue_op</red> 0x2aa9bb60 prio 63 cost 24 latency 0.000148 osd_op(client.5722.0:5 rbd_header.10126b8b4567 [call rbd.get_stripe_unit_count] 3.fa956e77 e267) v4 pg pg[3.e77( v 267'732 (0'0,267'732] local-les=267 n=2 ec=15 les/c 267/267 266/266/266) [15] r=0 lpr=266 mlcod 267'732 active+clean]
<br>2013-12-18 19:42:52.001615 7f2c8af69700 10 osd.15 267 <red>dequeue_op</red> 0x2aa9bb60 finish
<br>2013-12-18 19:42:52.001627 7f2c8af69700 15 OSD::op_tp <red>worker</red> wq OSD::OpWQ done processing 0x1 (0 active)
<br>2013-12-18 19:42:52.002200 7f2c9e790700 20 osd.15 267 <red>_dispatch</red> 0x23d6240 osd_op(client.5722.0:6 rbd_header.10126b8b4567 [watch add cookie 1 ver 0] 3.fa956e77 e267) v4
<br>2013-12-18 19:42:52.002223 7f2c9e790700 15 osd.15 267 <red><red>enqueue_op</red></red> 0x2aa9ba80 prio 63 cost 0 latency 0.000099 osd_op(client.5722.0:6 rbd_header.10126b8b4567 [watch add cookie 1 ver 0] 3.fa956e77 e267) v4
<br>2013-12-18 19:42:52.002275 7f2c8ff73700 12 OSD::op_tp <red>worker</red> wq OSD::OpWQ start processing 0x1 (1 active)
<br>2013-12-18 19:42:52.002293 7f2c8ff73700 10 osd.15 267 <red>dequeue_op</red> 0x2aa9ba80 prio 63 cost 0 latency 0.000169 osd_op(client.5722.0:6 rbd_header.10126b8b4567 [watch add cookie 1 ver 0] 3.fa956e77 e267) v4 pg pg[3.e77( v 267'732 (0'0,267'732] local-les=267 n=2 ec=15 les/c 267/267 266/266/266) [15] r=0 lpr=266 mlcod 267'732 active+clean]
<br>2013-12-18 19:42:52.002800 7f2c8ff73700 10 osd.15 267 <red>dequeue_op</red> 0x2aa9ba80 finish
<br>2013-12-18 19:42:52.002805 7f2c8ff73700 15 OSD::op_tp <red>worker</red> wq OSD::OpWQ done processing 0x1 (0 active)
<br>2013-12-18 19:42:52.004577 7f2ca26c5700 12 FileStore::op_tp <red>worker</red> wq FileStore::OpWQ start processing 0x2aa1dd80 (1 active)
<br>2013-12-18 19:42:52.005139 7f2ca26c5700 15 FileStore::op_tp <red>worker</red> wq FileStore::OpWQ done processing 0x2aa1dd80 (0 active)
<br>2013-12-18 19:42:52.005334 7f2c9e790700 20 osd.15 267 <red>_dispatch</red> 0x2ad1b480 osd_op(client.5722.0:7 rbd_header.10126b8b4567 [call rbd.get_size,call rbd.get_features,call rbd.get_snapcontext,call rbd.get_parent,call lock.get_info] 3.fa956e77 e267) v4
<br>2013-12-18 19:42:52.005373 7f2c9e790700 15 osd.15 267 <red><red>enqueue_op</red></red> 0x2aa9b9a0 prio 63 cost 111 latency 0.000124 osd_op(client.5722.0:7 rbd_header.10126b8b4567 [call rbd.get_size,call rbd.get_features,call rbd.get_snapcontext,call rbd.get_parent,call lock.get_info] 3.fa956e77 e267) v4
<br>2013-12-18 19:42:52.005444 7f2c97f83700 12 OSD::op_tp <red>worker</red> wq OSD::OpWQ start processing 0x1 (1 active)
<br>2013-12-18 19:42:52.005454 7f2c97f83700 10 osd.15 267 <red>dequeue_op</red> 0x2aa9b9a0 prio 63 cost 111 latency 0.000205 osd_op(client.5722.0:7 rbd_header.10126b8b4567 [call rbd.get_size,call rbd.get_features,call rbd.get_snapcontext,call rbd.get_parent,call lock.get_info] 3.fa956e77 e267) v4 pg pg[3.e77( v 267'733 (0'0,267'733] local-les=267 n=2 ec=15 les/c 267/267 266/266/266) [15] r=0 lpr=266 mlcod 267'733 active+clean]
<br>2013-12-18 19:42:52.006585 7f2c97f83700 10 osd.15 267 <red>dequeue_op</red> 0x2aa9b9a0 finish
<br>2013-12-18 19:42:52.006601 7f2c97f83700 15 OSD::op_tp <red>worker</red> wq OSD::OpWQ done processing 0x1 (0 active)
<br>2013-12-18 19:42:52.010185 7f2c9e790700 20 osd.15 267 <red>_dispatch</red> 0x2ad1b240 osd_op(client.5722.0:9 rbd_header.10126b8b4567 [watch remove cookie 1 ver 0] 3.fa956e77 e267) v4
<br>2013-12-18 19:42:52.010207 7f2c9e790700 15 osd.15 267 <red><red>enqueue_op</red></red> 0x2aa9b8c0 prio 63 cost 0 latency 0.000168 osd_op(client.5722.0:9 rbd_header.10126b8b4567 [watch remove cookie 1 ver 0] 3.fa956e77 e267) v4
<br>2013-12-18 19:42:52.010259 7f2c96780700 12 OSD::op_tp <red>worker</red> wq OSD::OpWQ start processing 0x1 (1 active)
<br>2013-12-18 19:42:52.010269 7f2c96780700 10 osd.15 267 <red>dequeue_op</red> 0x2aa9b8c0 prio 63 cost 0 latency 0.000230 osd_op(client.5722.0:9 rbd_header.10126b8b4567 [watch remove cookie 1 ver 0] 3.fa956e77 e267) v4 pg pg[3.e77( v 267'733 (0'0,267'733] local-les=267 n=2 ec=15 les/c 267/267 266/266/266) [15] r=0 lpr=266 mlcod 267'733 active+clean]
<br>2013-12-18 19:42:52.010692 7f2c96780700 10 osd.15 267 <red>dequeue_op</red> 0x2aa9b8c0 finish
<br>2013-12-18 19:42:52.010696 7f2c96780700 15 OSD::op_tp <red>worker</red> wq OSD::OpWQ done processing 0x1 (0 active)
<br>2013-12-18 19:42:52.011222 7f2ca1ec4700 12 FileStore::op_tp <red>worker</red> wq FileStore::OpWQ start processing 0x2aa1dd80 (1 active)
<br>2013-12-18 19:42:52.011701 7f2ca1ec4700 15 FileStore::op_tp <red>worker</red> wq FileStore::OpWQ done processing 0x2aa1dd80 (0 active)
<br>2013-12-18 19:42:56.918744 7f2c9e790700 20 osd.15 267 <red>_dispatch</red> 0x2ad94380 pg_stats_ack(1 pgs tid 18) v1
</textarea>
