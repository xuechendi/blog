---
layout: post
title: "How to do Ceph Latency Breakdown"
date: 2013-12-05 10:38
comments: true
categories: Ceph 
---

##The Ceph Latency Breakdown process includes 3 steps:

###1. Enable Log Configuration or Other performance count mechanism to record the runtime infomation.

###2. Produce the workload and meanwhile dump the perfcounter

###3. Parse the perfcounter dump data


##Details:

In ceph, a perfcounter mechanism has been adopted to do the latency breakdown, and the runtime info can be dumped by the admin_socket is an internal implementation in Ceph.

example:

<1> codes
``` c git diff
diff --git a/src/osd/OSD.cc b/src/osd/OSD.cc
index d34bb9c..6c308d3 100644
--- a/src/osd/OSD.cc
+++ b/src/osd/OSD.cc
@@ -7055,10 +7055,10 @@ PGRef OSD::OpWQ::_dequeue()
 
 void OSD::OpWQ::_process(PGRef pg, ThreadPool::TPHandle &handle)
 {
-  utime_t pg_lock_lat = ceph_clock_now(g_ceph_context);
+  utime_t start = ceph_clock_now(g_ceph_context);
   pg->lock_suspend_timeout(handle);
-  pg_lock_lat = ceph_clock_now(g_ceph_context) - pg_lock_lat;
-  osd->logger->tinc(l_osd_pg_lock_lat, pg_lock_lat);
+  utime_t end = ceph_clock_now(g_ceph_context);
+  osd->logger->tinc(l_osd_pg_lock_lat, end - start);
 
   OpRequestRef op;
   {
```

Current Ceph Version only enabled admin_socket in Ceph Cluster Side, so we need to enable it in RBD side.


<2> dump the data

``` bash ceph.conf
#Enable the admin_socket in QEMU RBD
check ceph.conf
perf = false
->
#perf = false
```

```xml QEMU disk.xml
    <disk type='network' device='disk'>
      <driver name='qemu' type='raw' cache='none'/>
      <source protocol='rbd' name='xcd_8osd/volume-3:admin_socket=/var/run/ceph/node5_1.asok'/>
      <target dev='vdb' bus='virtio'/>
      <serial>12f70341-d199-4fca-9270-56e5d6b80061</serial>
      <alias name='virtio-disk1'/>
      <address type='pci' domain='0x0000' bus='0x00' slot='0x06' function='0x0'/>
    </disk>
```
``` console
virsh attach instance1 disk.xml

ceph --admin-daemon /var/run/ceph/${osd} perf dump >> ${osd}.txt
```
<hr>
##Below is the summary of all scripts:
<a href ="{{ root_url }}/downloads/Ceph_Latency_Breakdown_Script.zip"><code style="font-family: 'Fjalla One','Georgia','Helvetica Neue',Arial,sans-serif;font-weight:900;font-size:12px;">download the script</code></a>

###Script to mkceph:(in the dir "mkceph")

mkceph_with_pg.sh



###Script to produce workload:(all put in the dir "run the test")

<b>run.sh</b>: aim to change the <code>ceph.conf</code> op thread number setting, then restart ceph, vm, and call <code>test.sh</code> to start the test. The test result will then be stored in the path "/data/xcd/ceph-CephXCD/run_${id}"

<b>test.sh</b>: aim to clean run a vm_num loadline test, call <code>mhost-volume-test-xcd.sh</code> to run the actual test

<b>mhost-volume-test-xcd.sh</b>: calls <code>clean_cache_on_ceph.sh</code>,<code>check_readahead.sh</code>, <code>volume-test_in_vm.sh</code>, <code>all.fio</code>, <code>volume-test_in_pc.sh</code>, <code>ceph_perf_counter_dump.sh</code>, <code>volume-test_in_ceph.sh</code>, <code>check_cephHealth.sh</code> to run the test(producing workload and collecting runtime system info & ceph info), then collects all the data back to the Node1.
```console 
#example:
./mhost-volume-test-xcd.sh 1 $vm_num 3 4k "vmNum_"$vm_num ${qd} node13_net
```
tip: There is a VM list stored in file "node13_net" 


###Script to parse the data:

<b>getLatbreakdown.sh</b>: aim to parse latency data from <code>*.asok</code> files using <code>post_perf.py</code>

<b>getResult.sh</b>: aim to parse iostat, mpstat data. It calls <code>post_volume_xcd.sh</code>, <code>post_osd.sh</code>, <code>post_cpu.sh</code>, <code>iostat.sh</code>

