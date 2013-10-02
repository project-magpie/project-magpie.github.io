---
layout: post
title: "That was a close shave!"
description: "Last night I almost bricked my spark box"
category: ""
tags: []
---
{% include JB/setup %}

While fiddling arround with the ubifs port I recently did. I misstyped the MTD Partion and completly erased mtd0. This sounds not that tragically, but on mtd0 u-boot is located. Without a bootloader there is only a small chance that the box will boot again. To reboot the box in such a situation is a absolut no go. Now you have to start brain work ...

<!--more-->

For the ubifs tests I have choosen the fresh dylan port of my [meta-stlinux](https://github.com/project-magpie/meta-stlinux/tree/dylan) layer. This was a first draft and so I just build the ,,core-image-minimal''. I included the mtd-utils but I've forgotten to include a ssh server aswell. This was not a big issue due to the fact that I used the serial line for connection. For testing I booted the box from USB. The USB was connect through a USB extension cable. Which cut me off from installing a fresh u-boot by inserting a USB Stick. 

What are the options?
---------------------

 - Transfer the u-boot by zmodem
 - Transfer the u-boot by ethernet.

The name minimal definitly means minimal. There was no serial command I did know to transer data. On the ethernet no SSH no telnet and of course no FTP. Okay lets start thinking...

BANG!!! my good old friend ,,netcat'' one little test and yes there is a ,,nc'' command. The rest was easy going.

How I solved the issue
----------------------

On the host PC start the server

    cat ~/uboot-alien.bin | nc -v -l 3333

Yes I do know alien is not the correct name for a golden media GM990 Box. But this was the only image I have found.

On the box I used this commands

    # nc 192.168.24.157 3333 > u-boot.bin    
    # du u-boot.bin 
    520     u-boot.bin
    # flash_eraseall -j /dev/mtd0
    flash_eraseall has been replaced by `flash_erase <mtddev> 0 0`; please use it
    Erasing 64 Kibyte @ 0 --  0 % complete flash_erase:  Cleanmarker written at 0
    Erasing 64 Kibyte @ 10000 -- 12 % complete flash_erase:  Cleanmarker written at 10000
    Erasing 64 Kibyte @ 20000 -- 25 % complete flash_erase:  Cleanmarker written at 20000
    Erasing 64 Kibyte @ 30000 -- 37 % complete flash_erase:  Cleanmarker written at 30000
    Erasing 64 Kibyte @ 40000 -- 50 % complete flash_erase:  Cleanmarker written at 40000
    Erasing 64 Kibyte @ 50000 -- 62 % complete flash_erase:  Cleanmarker written at 50000
    Erasing 64 Kibyte @ 60000 -- 75 % complete flash_erase:  Cleanmarker written at 60000
    Erasing 64 Kibyte @ 70000 -- 87 % complete flash_erase:  Cleanmarker written at 70000
    Erasing 64 Kibyte @ 70000 -- 100 % complete
    # flashcp u-boot.bin /dev/mtd0
    # reboot

Now it was time to to keep one's fingers crossed. One deep breath later the box bootet like every time before.

    Board: STx7111-Mboard (MB618)  [32-bit mode]
    info: Disregarding any EPLD
    
    
    U-Boot 1.3.1 (Oct 19 2010 - 18:08:50) - stm23_0043 - YW 1.0.017 Rel
    
    DRAM:  128 MiB
    NOR:     8 MiB
    NAND:  512 MiB
    *** Warning - bad CRC, using default environment
    
    In:    serial
    Out:   serial
    Err:   serial
    IdentID : 09 00 07 00 00 46 d1 
    0  ESC to stop autoboot:  3  

Except of the little message ,,*** Warning - bad CRC, using default environment'' this was caused because by writing 512kb I have destroyed u-boot environment. I am not sure If this is the right u-boot. I would be happy If someone with a Golden Media Spark reloaded Box can send me a copy of the u-boot.
