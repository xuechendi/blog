---
layout: post
title: "AIO-STRESS vs. FIO"
date: 2013-11-15 01:05
comments: true
categories: [Ceph, Storage]
---
There is no big gap to me in producing workload by AIO-STREE and FIO.

Just because I haven't using much more advanced feature in FIO like replay trace, and I just using libaio as the driver.

But here, I just wanna show some interesting test to see some behavior by different setting of io_depth and io_batch. And <code>sadly</code> that is just some wrong setting I did before and produced some strange data then I just realized how these settings mean. T_T

##Just follow my lead to see how I found this wrong setting. ^_^

![findings]( {{ root_url }}/images/AIO_vs_FIO/1.png)

After a loadline test of io_depth setting, I just found the figures followed some rules, so using qemu logs, I draw above graph. This is a graph recording the io request intervals sending from qemu to librbd, and catched in librbd logs.

As we can see, when we set the queue_depth as 8, the first io request has quite small interval(0.075ms), but the interval between the 8th and 9th io request is pretty long(1.6ms). And if we set the queue_depth as 16, or 32. The burst interval happened just between the 16th to 17th, and 32th to 33th respectively.

So there comes the deduction:
Queue will send out all requests, then wait the last request completes . After that it starts to send out new queue.

##How to prove it?

###Methodology:
   1. Set queue depth = 64
   2. make the 42th request sleep 3s in completion thread

###--> If RBD waits 3s then receives the 65th requests, we can confirm the deduction. 

###How to do this?

There is a function in librbd named C_AioRead::finish in AioCompletion.cc, so what we need to do is just randomly block this function for 3s, and see what happens.

![proving]( {{ root_url }}/images/AIO_vs_FIO/2.png)

Aha! The results just show as what we expect.

###But there are still 3 possibilities can produce result like this.
1.Wrong workload setting?
2.Ceph RBD design ?  if there is some lock in finish thread that blocks the submit thread
3.Virtio ? 

The situation produced by FIO

![proving]( {{ root_url }}/images/AIO_vs_FIO/3.png)

From this graph, we can see the result is really different by using aio-stress, which can help us to exclude last two possibilities. That means we must do some wrong setting in aio-stress or aio-stress just have some wierd strategy design.

Also we can see the difference in logs.

![proving]( {{ root_url }}/images/AIO_vs_FIO/4.png)

<code>Here I will add some aio-stress codes digging into soon, just not now~ Sorry</code>
<hr>
The right setting for random io in aio-stress to act more meet our demand (Here I said random io is because this setting may result in no io merge in sequential io)

``` console
./aio-stress -O -o 3 -i 1 -d 64 -r4k -s 4m /dev/vdb
```

![proving]( {{ root_url }}/images/AIO_vs_FIO/5.png)

![proving]( {{ root_url }}/images/AIO_vs_FIO/6.png)

 
