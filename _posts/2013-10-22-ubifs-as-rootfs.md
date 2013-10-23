---
layout: post
title: "UBIFS as rootfs"
description: ""
category: ""
tags: [ubifs rootfs ]
---
{% include JB/setup %}
This Post describes how to get startet with an UBIFS root filesystem. I suppose you are using a image generated for the "spark" device. I have not done any tests with "spark7162". For the spark boxes there is automatically a ubifs image generated. 

<!--more-->

Generate a core-image-minimal for example. This assumes You already have set-up the yocto build environment.

    MACHINE=spark bitbake core-image-minimal

The resulting image can be found here
    
    tmp/deploy/images/core-image-minimal-spark.ubi

This is a ubivolume which can be burned to flash ...


The Linux Way
=============

For flashing from Linux [booting over USB](https://github.com/project-magpie/meta-stlinux/wiki/Boot-from-USB-Stick) is recommended. Be cause we do not want to bite the hand that feeds us. For debugging and configuration purposes a Nullmodem Cable and a Terminal Emulator is recommended.
 
Flashing the image
------------------
To get a list with the mtd devices just dump the content of "/proc/mtd"

    root@spark:~# cat /proc/mtd 
    dev:    size   erasesize  name
    mtd0: 00080000 00010000 "Boot firmware"
    mtd1: 00700000 00010000 "Kernel"
    mtd2: 00080000 00010000 "Reserve"
    mtd3: 00800000 00020000 "Spark Kernel"
    mtd4: 17800000 00020000 "Spark Rootfs"
    mtd5: 00800000 00020000 "E2 Kernel"
    mtd6: 05000000 00020000 "E2 RootFs"

We want to modify the Partion number "6" also known as "E2 RootFs". Be carefull to not erase Number "0" otherwise your bootloader is blown.
At first you have to transfer the image. In absence of any other tools I do use netcat again.

On the host (quadros) run this

    $ cat tmp/deploy/images/core-image-minimal-spark.ubi | nc -l -v 3333

On the target run this

    root@spark:~# nc  quadros 3333 > /tmp/image.ubi

Now we can burn the image onto the flash

    root@spark:~# flash_eraseall /dev/mtd6
    root@spark:~# ubiformat /dev/mtd6 -q -y -f /tmp/image.ubi

Choosing the right commandline
------------------------------

I had a some difficulties with the console. The default bootagrs point to "ttyAS1" but I have to use ttyAS0. So I have to check this. This is the command line which works for me.
 
    setenv bootargs 'console=ttyAS0,115200 rw init=/bin/devinit coprocessor_mem=4m@0x40000000,4m@0x40400000 printk=1 nwhwconf=device:eth0,hwaddr:00:80:E1:12:40:61 rw ip=172.100.100.249:172.100.100.174:172.100.100.174:255.255.0.0:LINUX7109:eth0:off bigphysarea=6000 stmmaceth=msglvl:0,phyaddr:2,watchdog:5000 ubi.mtd=6 rootfstype=ubifs root=ubi0:rootfs'

The spark u-boot
================

The spark bootloader has a nice menu integrated. With this you can also write images from a USB stick to the flash. Notice this is not the recommended way. Using ubiformat keeps all required information about the underlying flash device. To get started you need a USB-Stick which is supported by the spark u-noot. To boot the UBI image the commandline also have be tweaked to use ubi instead jffs2. Please have a look at the The Linux Way section for advise.


- Format USB-thumb drive with a single FAT32 partition
- Create an enigma2 folder on the USB-thumb drive
- Copy the file core-image-minimal-spark.ubi to the enigma2 folder and rename it to e2jffs2.img
- Copy the file uImage-spark.bin to the enigma2 folder as-well and rename it to uImage
- Turn the Set-Top-Box off by the power switch
- Put the USB-Stick into the USB-Slot at the back of the STB. Not the front one!
- Press the OK at the Front-Panel and keep it pressed.
- Turn on the box by the power switch.
- Wait until you read "Forc" on the display
- Release the OK Button. Now you can press the Button V+ -> (right).
- After this you can read "U LD" on the display.
- The flashing of the image may take some minutes. After successfull flashing "SUCC" is written on the display and the box reboots.


