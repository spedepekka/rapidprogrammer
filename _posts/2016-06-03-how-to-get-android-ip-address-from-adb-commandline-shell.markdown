---
layout: post
title: "How to get Android IP address from adb/commandline/shell"
date: "2016-06-03 10:01:28 +0300"
permalink: /:title
---

Older devices have *netcfg*

    adb shell netcfg

If the device doesn't have netcfg installed try

    adb shell ip addr show wlan0

Also this is possible

    adb shell ip route

Oneliners

This works only for Wifi. It does not show the public IP of the device
when connected through 3G/4G.

    adb shell ip route | awk '{print $9}'

Or

    adb shell ip addr show wlan0 | grep "inet\s" | awk '{print $2}' | awk -F'/' '{print $1}'
