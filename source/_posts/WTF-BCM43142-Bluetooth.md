---
title: WTF BCM43142 Bluetooth
date: 2015-11-15 15:33:04
tags:
  - linux
---
This shit is not supported by any version of stock Linux.

`rfkill` showed this device but cannot work at all.

I found [this post](https://outhereinthefield.wordpress.com/2014/03/01/ubuntu-13-10-and-bluetooth-on-broadcom-bcm43142-wifibt-combo-adapter/) and [this bug](https://bugs.launchpad.net/ubuntu/+source/bluez/+bug/1366418/comments/12), using hex file in windows, but seems that i must rebuild kernel. F**k.