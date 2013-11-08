---
layout: post
title: "Rewriting history"
description: ""
category: ""
tags: [git]
---
{% include JB/setup %}
Sometimes history has to be rewritten. In real life this is hopefully not possible yet. But in case of git this is possible. Some projecs start with a small repository which evolves over the time to a big bloated repository. Other ones start already bloated. In some cases the wish to split-up the repository in smaller chunks emerges. With some advice from [Stefan Seyfried](http://seife.kernalert.de/blog/) I learned how to rewrite history
<!--more-->
The goal was to extract the driver directory from the [tdt](https://gitorious.org/open-duckbox-project-sh4/tdt) repository like [Stefan](https://gitorious.org/neutrino-mp/tdt-driver) already did.


These are the steps to follow:

    $ cd /dev/shm/
    $ git clone --reference /net/transfer/spark71xx/tdt https://git.gitorious.org/open-duckbox-project-sh4/tdt.git tdt-driver
    $ cd tdt-driver/
    $ git filter-branch --subdirectory-filter tdt/cvs/driver

The first step speeds up the process of rewriting by doing it all in memory instead of writing everything onto a slow disk device. But please remember to push your work onto a less volatile memory. 
    
    $ cd /dev/shm/

In the second step the original repository is cloned. To again speedup things I used a local copy as reference. After cloning the directory structure looked like this

    $ tree -d -L 3 tdt/
    tdt/
    ├── custom
    ├── cvs
    │   ├── apps
    │   │   ├── dvb
    │   │   ├── enigma1-hd
    │   │   ├── enigma2
    │   │   ├── misc
    │   │   ├── neutrino
    │   │   ├── tuxbox
    │   │   └── vdr
    │   ├── boot
    │   │   └── u-boot-tufsbox
    │   ├── cdk
    │   │   ├── integrated_firmware
    │   │   ├── make
    │   │   ├── own_build
    │   │   ├── Patches
    │   │   ├── root
    │   │   ├── static
    │   │   └── tfinstaller
    │   ├── driver
    │   │   ├── adb_box_fan
    │   │   ├── avs
    │   │   ├── boxtype
    │   │   ├── bpamem
    │   │   ├── button
    │   │   ├── button_hs5101
    │   │   ├── cec
    │   │   ├── cec_adb_box
    │   │   ├── cic
    │   │   ├── compcache
    │   │   ├── cpu_frequ
    │   │   ├── dvbt
    │   │   ├── e2_proc
    │   │   ├── frontcontroller
    │   │   ├── frontends
    │   │   ├── i2c_spi
    │   │   ├── include
    │   │   ├── ipbox99xx_fan
    │   │   ├── led
    │   │   ├── logfs
    │   │   ├── multicom-3.2.2
    │   │   ├── multicom-3.2.4
    │   │   ├── multicom-3.2.4_rc3
    │   │   ├── multicom-4.0.6
    │   │   ├── old
    │   │   ├── player2_131
    │   │   ├── player2_179
    │   │   ├── player2_191
    │   │   ├── pti
    │   │   ├── rmu
    │   │   ├── sata_switch
    │   │   ├── siinfo
    │   │   ├── simu_button
    │   │   ├── smartcard
    │   │   ├── stgfb
    │   │   ├── tfswitch
    │   │   ├── ufs922_fan
    │   │   └── wireless
    │   └── hostapps
    │       ├── flash
    │       ├── mkfs.jffs2
    │       └── mklibs
    └── flash
        ├── at7500
        │   ├── extras
        │   ├── scripts
        │   ├── scripts_209
        │   └── scripts_extended
        ├── common
        │   ├── fup.src
        │   ├── mup.src
        │   └── pad.src
        ├── hs7810a
        │   ├── extras
        │   └── scripts
        ├── spark
        │   ├── extras
        │   └── scripts
        ├── tf7700hdpvr
        ├── ufc960
        │   ├── extra
        │   └── scripts
        ├── ufs910
        │   └── scripts
        ├── ufs912
        │   ├── extras
        │   └── scripts
        └── ufs913
            ├── extra
            ├── scripts
            └── test

The last step extracts just the directory  ''tdt/cvs/driver''

    $ git filter-branch --subdirectory-filter tdt/cvs/driver

After this step the filesystem looks like this:

    $ tree -d -L 1 tdt-driver/
    tdt-driver/
    ├── adb_box_fan
    ├── avs
    ├── boxtype
    ├── bpamem
    ├── button
    ├── button_hs5101
    ├── cec
    ├── cec_adb_box
    ├── cic
    ├── compcache
    ├── cpu_frequ
    ├── dvbt
    ├── e2_proc
    ├── frontcontroller
    ├── frontends
    ├── i2c_spi
    ├── include
    ├── ipbox99xx_fan
    ├── led
    ├── logfs
    ├── multicom-3.2.2
    ├── multicom-3.2.4
    ├── multicom-3.2.4_rc3
    ├── multicom-4.0.6
    ├── old
    ├── player2_131
    ├── player2_179
    ├── player2_191
    ├── pti
    ├── rmu
    ├── sata_switch
    ├── siinfo
    ├── simu_button
    ├── smartcard
    ├── stgfb
    ├── tfswitch
    ├── ufs922_fan
    └── wireless
        
There are more steps recommended in a [stackoverflow](http://stackoverflow.com/questions/359424/detach-subdirectory-into-separate-git-repository/1591174#1591174) article. These steps had not been necessary for my example.
