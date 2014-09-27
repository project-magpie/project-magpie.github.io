---
layout: post
title: "Building a Yocto image for the A20 OLinuXino LIME"
description: ""
category: "yocto"
tags: olimex,yocto,A20,Lime,sunxi
---
{% include JB/setup %}

Some time ago I ordered a [A20-OLinuXino-LIME-4GB](https://www.olimex.com/Products/OLinuXino/A20/A20-OLinuXino-LIME-4GB/open-source-hardware) board. This was planed as a replacement for my not so beloved Raspberry PI. I am not that big fan of the PI because of the wacky SD-Card holder and the USB stability. I hope the 4GB NAND of the [A20-OLinuXino-LIME-4GB](https://www.olimex.com/Products/OLinuXino/A20/A20-OLinuXino-LIME-4GB/open-source-hardware) Will improve the stability of my target application.
<!--more-->
As a big fan of the [Yocto Project](https://www.yoctoproject.org/) I decided to run poky on my OLinuXino.

# Preparation

Clone the git repositories and switch to the daisy branch.

    mkdir /data/src/yocto
    cd /data/src/yocto
    git clone git://git.yoctoproject.org/poky
    git clone git@github.com:linux-sunxi/meta-sunxi.git
    cd poky
    git checkout -b daisy origin/daisy
    cd .. meta-sunxi
    git checkout -b daisy origin/daisy

Prepare a warm and cozy environment for the build

    cd /data/src/yocto/poky
    mkdir -p ../build/a20-lime
    source oe-init-build-env ../build/a20-lime

Add the [sunxi meta-layer](https://github.com/linux-sunxi/meta-sunxi) to the build environment

    --- a/conf/bblayers.conf
    +++ b/conf/bblayers.conf
    @@ -9,6 +9,7 @@ BBLAYERS ?= " \
       /data/src/yocto/poky/meta \
       /data/src/yocto/poky/meta-yocto \
       /data/src/yocto/poky/meta-yocto-bsp \
    +  /data/src/yocto/meta-sunxi \
       "
     BBLAYERS_NON_REMOVABLE ?= " \
       /data/src/yocto/poky/meta \

Set the correct target machine and tuning parameters

    diff --git a/conf/local.conf b/conf/local.conf
    index bcb0864..6a793e6 100644
    --- a/conf/local.conf
    +++ b/conf/local.conf
    @@ -55,7 +55,7 @@ PARALLEL_MAKE ?= "-j ${@oe.utils.cpu_count()}"
     #MACHINE ?= "edgerouter"
     #
     # This sets the default machine to be qemux86 if no other machine is selected:
    -MACHINE ??= "qemux86"
    +MACHINE ??= "olinuxino-a20"

     #
     # Where to place downloads
    @@ -125,7 +125,7 @@ DISTRO ?= "poky"
     #  - 'package_rpm' for rpm style packages
     # E.g.: PACKAGE_CLASSES ?= "package_rpm package_deb package_ipk"
     # We default to rpm:
    -PACKAGE_CLASSES ?= "package_rpm"
    +PACKAGE_CLASSES ?= "package_ipk"

     #
     # SDK/ADT target architecture
    @@ -134,7 +134,19 @@ PACKAGE_CLASSES ?= "package_rpm"
     # you can build the SDK packages for architectures other than the machine you are
     # running the build on (i.e. building i686 packages on an x86_64 host).
     # Supported values are i686 and x86_64
    -#SDKMACHINE ?= "i686"
    +SDKMACHINE ?= "i686"
    +
    +#
    +# The default machine settings are meant to be the lowest common denominator,
    +# maximizing generality. Significantly better performance (2x-3x) can be achieved
    +# with the following settings:
    +#
    +# Allwinner A20
    +#
    +# For Allwinner A20 (Cubieboard2/CubieTruck), the following tuning options are recommended:
    +#
    +# Enable hardfloat, thumb2 and neon capabilities
    +DEFAULTTUNE = "cortexa7hf-neon-vfpv4"

     #
     # Extra image configuration defaults

# Building the image

Now the image can be build, lean back and grap some coffee, beer, ...

    bitbake core-image-base

    [...]
    Build Configuration:
    BB_VERSION        = "1.22.0"
    BUILD_SYS         = "x86_64-linux"
    NATIVELSBSTRING   = "Ubuntu-12.04"
    TARGET_SYS        = "arm-poky-linux-gnueabi"
    MACHINE           = "olinuxino-a20"
    DISTRO            = "poky"
    DISTRO_VERSION    = "1.6.1"
    TUNE_FEATURES     = "armv7a vfp neon callconvention-hard vfpv4 cortexa7"
    TARGET_FPU        = "vfp-vfpv4-neon"
    meta
    meta-yocto
    meta-yocto-bsp    = "daisy:a4d8015687cf9ddd6ef563e29cf840698f81c099"
    meta-sunxi        = "daisy:41596163b46f51e43dc7b351132cc0bf1f9d6ed3"
    [...]

After a successful build all images are located in the folder

    /data/src/yocto/build/a20-lime/tmp/deploy/images/olinuxino-a20/

# Building the SDK

Sometimes it is handy to have a toolchain apart from the whole yocto/poky environment. So lets build some

    bitbake -cpopulate_sdk core-image-base

This will take mostly as long as the image build so again grab some coffee, beer or .....
The result is located here:

    /data/src/yocto/build/a20-lime/tmp/deploy/sdk/poky-eglibc-i686-core-image-base-cortexa7hf-vfp-vfpv4-neon-toolchain-1.6.1.sh

# Installation of the Toolchain

You may need root access to install in /opt

    sudo  tmp/deploy/sdk/poky-eglibc-i686-core-image-base-cortexa7hf-vfp-vfpv4-neon-toolchain-1.6.1.sh
    Enter target directory for SDK (default: /opt/poky/1.6.1):
    You are about to install the SDK to "/opt/poky/1.6.1". Proceed[Y/n]?y
    Extracting SDK...
    Setting it up...done
    SDK has been successfully set up and is ready to be used.

# Using the SDK

To use the SDK we have to source the set-up script

    source  /opt/poky/1.6.1/environment-setup-cortexa7hf-vfp-vfpv4-neon-poky-linux-gnueabi

After this some variables should be set like this

    echo $CC
    arm-poky-linux-gnueabi-gcc -march=armv7-a -mthumb-interwork -mfloat-abi=hard -mfpu=neon-vfpv4 -mtune=cortex-a7 --sysroot=/opt/poky/1.6.1/sysroots/cortexa7hf-vfp-vfpv4-neon-poky-linux-gnueabi

