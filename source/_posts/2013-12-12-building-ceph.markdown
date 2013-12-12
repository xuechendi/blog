---
layout: post
title: "Building Ceph"
date: 2013-12-12 11:17
comments: true
categories: Ceph 
---

Actually, I had bulid ceph from codes for many times, just each time, I have to struggle a little bit, for I always forget install some libraries.

So, record things here

I found ceph.com have also detailed its "Building from Codes" doc, indeed, much detailed than previous version

But seems I still missing some packages when doing "make"

<code><http://ceph.com/docs/master/install/build-ceph/></code>

```console pre-make-ceph-packages
sudo apt-get install pkg-config autotools-dev autoconf automake cdbs gcc g++ git libboost-dev libedit-dev libssl-dev libtool libfcgi libfcgi-dev libfuse-dev linux-kernel-headers libcrypto++-dev libcrypto++ libexpat1-dev libboost-thread-dev uuid-dev libkeyutils-dev libgoogle-perftools-dev libatomic-ops-dev libaio-dev libgdata-common libgdata13 libsnappy-dev libleveldb-dev libboost-program-options-dev
cd ceph
./autogen.sh
./configure
make
make install

```
