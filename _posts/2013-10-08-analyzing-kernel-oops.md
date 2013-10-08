---
layout: post
title: "Analyzing Kernel Oops"
description: ""
category: ""
tags: []
---
{% include JB/setup %}

There was a little hickup in the kernel while shutting down or rebooting the system. The result of the hickup was a Kernel Oops. Because it had no big influence on the behaviour of the system I did not spend a lot of time on investigation. Yesterday I did some tests with the yocto workflow for kernel development and tried to fix the hickup.

<!--more-->

The hickup
----------

The Kernel Oops looked like this one

    
    Unmounting local filesystems...
    [ 2135.692000] umount: sending ioctl 4c01 to a partition!
    [ 2135.696000] umount: sending ioctl 4c01 to a partition!
    Rebooting... [ 2137.744000] ------------[ cut here ]------------
    [ 2137.744000] Badness at 80a859a0 [verbose debug info unavailable]
    [ 2137.744000] 
    [ 2137.744000] Pid : 855, Comm:                 reboot
    [ 2137.744000] CPU : 0                  Not tainted  (2.6.32.59_stm24_0211 #2)
    [ 2137.744000] 
    [ 2137.744000] PC  : 80a859a0 SP  : 8928be48 SR  : 400080f1 TEA : c113ae28
    [ 2137.744000] R0  : 00000000 R1  : 00000000 R2  : 80c22e7c R3  : 00002000
    [ 2137.744000] R4  : 80c22ba4 R5  : 80a85610 R6  : 00000000 R7  : 00003fff
    [ 2137.744000] R8  : 80c229cc R9  : 80a855fe R10 : 89ec440c R11 : 00000000
    [ 2137.744000] R12 : fffff000 R13 : 80a859e0 R14 : 00000000
    [ 2137.744000] MACH: 000000de MACL: 00000014 GBR : 296c1470 PR  : 80a859cc
    [ 2137.744000] 
    [ 2137.744000] Call trace:
    [ 2137.744000]  [<80a85a04>] 0x80a85a04
    [ 2137.744000]  [<80a5347a>] 0x80a5347a
    [ 2137.744000]  [<809db332>] 0x809db332
    [ 2137.744000]  [<8081f610>] 0x8081f610
    [ 2137.744000]  [<8081f636>] 0x8081f636
    [ 2137.744000]  [<8081f780>] 0x8081f780

For me under yocto the best way to get in touch with the source is to start a devshell.

    bitbake -cdevshell virtual/kernel

While the occurence of the Oops the PC (program counter) was at address 0x80a859a0. So my first approach was loading the vmlinux in gdb. This was not that succefull because the kernel had no debug symbols included. The next source of information is the Sytem.map file:

    cat cat System.map | grep 80a859 
    80a85904 T _clk_disable 

This looks like the cause of the problem is located in this function. To get a closer look I decided to compile the kernel with debug symbols. So I added the following config options:

    CONFIG_DEBUG_KERNEL=y
    CONFIG_DETECT_SOFTLOCKUP=y
    CONFIG_BOOTPARAM_SOFTLOCKUP_PANIC_VALUE=0
    CONFIG_DETECT_HUNG_TASK=y
    CONFIG_BOOTPARAM_HUNG_TASK_PANIC_VALUE=0
    CONFIG_SCHED_DEBUG=y
    CONFIG_DEBUG_PREEMPT=y
    CONFIG_DEBUG_BUGVERBOSE=y
    CONFIG_DEBUG_INFO=y
    
And recompiled the kernel

    bitbake -ccompile -f virtual/kernel
    bitbake -cdeploy -f virtual/kernel

And finally copied the kernel onto the USB pendrive. Because I do the most of my testing while booting from USB.

    cp  tmp/deploy/images/uImage-spark.bin  /media/spark/uImage

After unmounting the pendrive I tested on the box with the follwing results:

    Unmounting local filesystems...
    [   38.764000] umount: sending ioctl 4c01 to a partition!
    [   38.768000] umount: sending ioctl 4c01 to a partition!
    Rebooting... [   40.816000] ------------[ cut here ]------------
    [   40.816000] Badness at drivers/stm/clk.c:190
    [   40.816000] 
    [   40.816000] Pid : 779, Comm:                 reboot
    [   40.816000] CPU : 0                  Not tainted  (2.6.32.59_stm24_0211 #2)
    [   40.816000] 
    [   40.816000] PC  : 80a8b7e4 SP  : 88e59e44 SR  : 400080f1 TEA : c16772ac
    [   40.816000] R0  : 00000000 R1  : 00000000 R2  : 80c2fe58 R3  : 00002000
    [   40.816000] R4  : 80c2fb80 R5  : 80a8b438 R6  : 00000000 R7  : 00003fff
    [   40.816000] R8  : 80c2f9a8 R9  : 80a8b426 R10 : 89e2048c R11 : 00000000
    [   40.816000] R12 : fffff000 R13 : 80a8b824 R14 : 00000000
    [   40.816000] MACH: 000000de MACL: 00000014 GBR : 296c1470 PR  : 80a8b810
    [   40.816000] 
    [   40.816000] Call trace:
    [   40.816000]  [<80a8b84a>] 0x80a8b84a
    [   40.816000]  [<80a58f9e>] 0x80a58f9e
    [   40.816000]  [<809e055a>] 0x809e055a
    [   40.816000]  [<8082188c>] 0x8082188c

The Oops now tells me where I do have to look at:

    [   40.816000] Badness at drivers/stm/clk.c:190

The next test is to try what gdb tells me the address varies, because this is the kernel with the fix applied. But the mechanism is the same.

    # sh4-poky-linux-gdb vmlinux
    GNU gdb (GDB) 7.5.1
    Copyright (C) 2012 Free Software Foundation, Inc.
    License GPLv3+: GNU GPL version 3 or later <http://gnu.org/licenses/gpl.html>
    This is free software: you are free to change and redistribute it.
    There is NO WARRANTY, to the extent permitted by law.  Type "show copying"
    and "show warranty" for details.
    This GDB was configured as "--host=x86_64-linux --target=sh4-poky-linux".
    For bug reporting instructions, please see:
    <http://www.gnu.org/software/gdb/bugs/>...
    Reading symbols from vmlinux...done.
    (gdb) list *(0x80a8b460)
    0x80a8b460 is in _clk_disable (drivers/stm/clk.c:187).
    182		return ret;
    183	}
    184	EXPORT_SYMBOL(clk_enable);
    185	
    186	void _clk_disable(struct clk *clk)
    187	{
    188		int ret;
    189		
    190	
    191		if (clk_is_always_enabled(clk)) {
    (gdb) 

With this information gdb can help to narrow the source of the problem. In my case I just googled the keywords stm and \_clk_disable and found the following patch:
[linux-sh4-fix-crash-usb-reboot_stm24_0211.diff](http://code.google.com/p/tdt-amiko/source/browse/tdt/cvs/cdk/Patches/linux-sh4-fix-crash-usb-reboot_stm24_0211.diff?spec=svnbf027b5e899cd26fbc20ac0745385c69ab385923&r=bf027b5e899cd26fbc20ac0745385c69ab385923)




