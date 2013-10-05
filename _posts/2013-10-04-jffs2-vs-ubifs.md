---
layout: post
title: "JFFS2 vs. UBIFS"
description: ""
category: ""
tags: []
---
{% include JB/setup %}

After a successfull merge of the [UBIFS patches](https://github.com/project-magpie/linux-sh4-2.6.32.y/tree/ubifs-merge)  I have made some benchmarks. In both cases I have flashed the same image. It was a fresh build of core-image-minimal from the [dylan branch](https://github.com/project-magpie/meta-stlinux/tree/dylan). Maybe these benachmarks have to be repeated with a more representative scenario. More Information regarding [UBIFS](http://en.wikipedia.org/wiki/UBIFS) can be found on [eLinux](http://elinux.org/UBIFS). If you are interested in more flash file system Benchmarks you can also have a look a [eLinux](http://elinux.org/Flash_Filesystem_Benchmarks).

<!--more-->

Mounting a JFFS2 Volume
-----------------------

    
    root@spark:~# time mount /dev/mtdblock6  -tjffs2 /mnt/
    real    0m 2.06s
    user    0m 0.00s
    sys     0m 2.06s

Mounting a UBIFS volume
-----------------------

    
    root@spark:~# time ubiattach -m6 && time mount -tubifs ubi0:rootfs /mnt/
    [  663.748000] UBI: attaching mtd6 to ubi0
    [  663.752000] UBI: physical eraseblock size:   131072 bytes (128 KiB)
    [  663.760000] UBI: logical eraseblock size:    129024 bytes
    [  663.764000] UBI: smallest flash I/O unit:    2048
    [  663.768000] UBI: sub-page size:              512
    [  663.776000] UBI: VID header offset:          512 (aligned 512)
    [  663.780000] UBI: data offset:                2048
    [  663.940000] UBI: max. sequence number:       28
    [  663.960000] UBI: attached mtd6 to ubi0
    [  663.972000] UBI: MTD device name:            "E2 RootFs"
    [  663.976000] UBI: MTD device size:            80 MiB
    [  663.980000] UBI: number of good PEBs:        638
    [  663.988000] UBI: number of bad PEBs:         2
    [  663.992000] UBI: number of corrupted PEBs:   0
    [  663.996000] UBI: max. allowed volumes:       128
    [  664.000000] UBI: wear-leveling threshold:    4096
    [  664.008000] UBI: number of internal volumes: 1
    [  664.012000] UBI: number of user volumes:     1
    [  664.016000] UBI: available PEBs:             0
    [  664.020000] UBI: total number of reserved PEBs: 638
    [  664.024000] UBI: number of PEBs reserved for bad PEB handling: 6
    [  664.028000] UBI: max/mean erase counter: 1/0
    [  664.032000] UBI: image sequence number:  1037100788
    [  664.040000] UBI: background thread "ubi_bgt0d" started, PID 553
    UBI device number 0, total 638 LEBs (82317312 bytes, 78.5 MiB), available 0 LEBs (0 bytes), LEB size 129024 bytes (126.0 KiB)
    real    0m 0.31s
    user    0m 0.00s
    sys     0m 0.28s
    [  664.092000] UBIFS: background thread "ubifs_bgt0_0" started, PID 556
    [  664.160000] UBIFS: mounted UBI device 0, volume 0, name "rootfs"<NULL>
    [  664.164000] UBIFS: LEB size: 129024 bytes (126 KiB), min./max. I/O unit sizes: 2048 bytes/2048 bytes
    [  664.168000] UBIFS: FS size: 79607808 bytes (75 MiB, 617 LEBs), journal size 9033728 bytes (8 MiB, 71 LEBs)
    [  664.172000] UBIFS: reserved for root: 0 bytes (0 KiB)
    [  664.176000] UBIFS: media format: w4/r0 (latest is w4/r0), , small LPT model
    real    0m 0.14s
    user    0m 0.00s
    sys     0m 0.11s

