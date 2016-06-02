---
layout: post
title: "How to install additional components to XCode from commandline/shell"
date: "2016-06-02 15:20:13 +0300"
permalink: /:title
---

To avoid the two popups on first XCode launch you can install the additional components like this

```
sudo installer -pkg /Applications/<YOUR_XCODE>/Contents/Resources/Packages/MobileDevice.pkg -target /
sudo installer -pkg /Applications/<YOUR_XCODE>/Contents/Resources/Packages/MobileDeviceDevelopment.pkg -target /
```

Replace the `<YOUR_XCODE>` with your XCode path. The default is `Xcode.app`.

Check out my other post [How to agree XCode license from commandline/shell]({% post_url 2016-06-02-how-to-agree-xcode-license-from-commandline-shell %}) for more XCode automation.
