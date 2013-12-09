---
layout: post
title: "ObjectCacher: RBD Cache Codes"
date: 2013-11-15 00:58
comments: true
categories: [Ceph, Storage] 
---

There is some cache mechanism in Ceph RBD.
And it is now in memory cache which supports write through and write back.

In this post, I would post what I have learnt from ObjectCacher codes.

##How to deploy RBD Cache

there is a really clear instruction in ceph.com, just linked here <code>[Ceph RBD Cache setting](http://ceph.com/docs/master/rbd/rbd-config-ref/?highlight=rbd%20cache)</code>

##How to record log from RBD

There is a little tricky to record log in RBD sides, because if you simply add the log settings in ceph.conf, you can only get logs when you do something using "rbd" command, (I just cannot remember what is the exact command, will add this later ), etc. So simply adding log settings in ceph.conf(client side) can not help you to get logs from QEMU to librbd, the reason is unknown to me, but I just find a way to walk around this.

All you need to do is to add log setting in you instance.xml, then using libvirt to boot this instance or also you can just attach a new disk by using xml like below.

``` xml disk.xml
    <disk type='network' device='disk'>
      <driver name='qemu' type='raw' cache='none'/>
      <source protocol='rbd' name='xcd_8osd/volume-3:debug_rbd=20:debug_objectcacher=20:log_file=/tmp/qemu-rbd.log'/>
      <target dev='vdb' bus='virtio'/>
      <serial>12f70341-d199-4fca-9270-56e5d6b80061</serial>
      <alias name='virtio-disk1'/>
      <address type='pci' domain='0x0000' bus='0x00' slot='0x06' function='0x0'/>
    </disk>
``` 

After attach this disk, you can find logs in /tmp/qemu-rbd.log or you can just write to qemu log.

``` xml write log to /var/libvirt/qemu/xxxxx(instance_name).log
      <source protocol='rbd' name='xcd_8osd/volume-3:debug_rbd=20:debug_objectcacher=20:log_to_stderr=true'/>
```
There is a really clear codes path we can find from logs, so just using log to debug or hack into the rbd codes.

##Dig into the codes

If we enable rbd cache in ceph.conf(client side), the ObjectCacher object is created when this rbd image attach to the vm.
Here is the overview.

<img src="{{ root_url }}/images/RBD_Cache/overview.png" width=60%>

From above we can see each RBD has a object to store all information name ImageCtx(image context), so there is only one ObjectCacher in one RBD cache and also different RBD images can not share their cache till now.

ObjectCacher uses poolid, oid(objectid), then offset and length to index cache in memory.

Then here is a graph to show how ceph using ObjectCacher.

<img src="{{ root_url }}/images/RBD_Cache/workflow_send_req.png" width=100%>

<img src="{{ root_url }}/images/RBD_Cache/workflow_recv_req.png" width=50%>


