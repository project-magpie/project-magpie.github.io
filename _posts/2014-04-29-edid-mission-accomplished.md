---
layout: post
title: "EDID Mission accomplished"
description: ""
category: ""
tags: []
---
{% include JB/setup %}
After my [HDMI EDID analisys](http://project-magpie.github.io/2014/04/26/pursue-hdmi-edid-bugs/) I decided getting more in touch with the hardware. I tried to trace the loose ends and checked where the onboard EEPROM is located. The next step was disabling the EEPROM and connecting the STB I2C Bus to the HDMI side.As in my previous [post](2014/04/26/pursue-hdmi-edid-bugs) expected the missing SOT-23 parts near the HDMI output form a level shifter which convert the 3.3V I2C STB level to the 5V level of the HDMI side an vice versa.

<!--more-->

A detailed description of such an level shifter is described in an application note from [Phillips](http://www.adafruit.com/datasheets/an97055.pdf)/[NXP](http://www.nxp.com/documents/application_note/AN10441.pdf).

I started with the trace of the EEPROM. I finally found the EEPROM.

![EDID EEPROM]({{ site.url }}/assets/edid/edid_eeprom.jpg)

There are two serial resistors between I2C SCL and I2C SDA. Which can be more or less easily removed. 

![EDID EEPROM disabled]({{ site.url }}/assets/edid/edid_eeprom_disabled.jpg)

After soldering my very first sot-23 transistor which made me feeling like my soldering iron is too huge for decent jobs. I cursed myself for not taking the box to work where we do have a more professional equipement at least when it comes to soldering irons.  

The result looks ugly but seems to work.

![EDID Transistor]({{ site.url }}/assets/edid/edid_transistor.jpg)

As Transistor I've chosen some  N-Channel Field Effect Transistor. Anything like a [2N7002](http://www.fairchildsemi.com/ds/2N/2N7000.pdf) or a [BSS138](http://www.fairchildsemi.com/ds/BS/BSS138.pdf) should do the job.

For me the final result was a working EDID readout of the HDMI monitor's EDID EEPROM. And the first time the STB took the correct HDMI-CEC address. I had not been able to check levels and edges of the 5V signal and the ultimate HDMI-CEC test also have to wait some time. Because my test lab HDMI monitor have no HDMI-CEC support.