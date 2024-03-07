---
layout: post
title: "Ubuntu or Kubuntu 23.10 ADB connection to OnePlus 8T"
date: "2024-03-07 10:36:00 +0300"
permalink: /:title
---

The first time I did this udev rules fix for Android ADB connection was around 2012. Today I had to do it again for my Kubuntu 23.10.

## 1. Enable developer mode

Go to *Settings / About Device / Version* and tap Build number 10 times.

Go to *Settings / System Settings / Developer options* and make sure USB debugging is turned on.

For some reason I wasn't developer in my main device so I had to do this to make the device even visible into `adb devices`.

## 2. Plug the device in

After connecting the device and running `adb devices` I got following:

```
List of devices attached
c6342f8c        no permissions (missing udev rules? user is in the plugdev group); see [http://developer.android.com/tools/device.html]
```

This indicates that the udev rules are not properly set up.

## 3. Fix udev rules

Instead of trying to find the correct values from search engines just use `lsusb`. It should tell the vendor and product ID for your device.

```
Bus 001 Device 012: ID 2abc:4def OnePlus Technology ...
```

Open udev rules

```
sudo vim /etc/udev/rules.d/51-android.rules
```

and put the values from lsusb into it like this

```
SUBSYSTEM=="usb", ATTR{idVendor}=="2abc", ATTR{idProduct}=="4def", MODE="0666", GROUP="plugdev"
```

Remember to make sure your user is part of group `plugdev`.

Then reload the rules.

```
sudo udevadm control --reload-rules
```

