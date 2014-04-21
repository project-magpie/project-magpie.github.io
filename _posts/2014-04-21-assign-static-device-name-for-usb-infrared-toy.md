---
layout: post
title: "Assign Static Device Name for USB Infrared Toy"
description: ""
category: ""
tags: []
---
{% include JB/setup %}
The last few days I was playing arround with my [USB Infrared Toy v2](http://dangerousprototypes.com/docs/USB_Infrared_Toy) from Dangerous Prototypes. Due to the fact that I have attached some more devices which are detected as **/dev/ACM\[0-9\]** I decided to write an udev rule to assign a static device name.
<!--more-->

I stored the rule in the following file: **/etc/udev/rules.d/98-ir-toy-v2.rules**

    SUBSYSTEM=="tty", ATTRS{manufacturer}=="Dangerous Prototypes", ATTRS{idProduct}=="fd08", SYMLINK+="ir_toy" MODE="0666"

Maybe the field **"ATTRS{idProduct}=="fd08""** have to be tweaked for versions differnet of v2. With this rule the IR Toy is accessible via **/dev/ir_toy**

