---
layout: post
title: "Android Studio \"Include C++ Support\" checkbox is missing"
date: "2016-10-25 14:17:41 +0300"
permalink: /:title
---

I'm playing around with OpenCV and Android. I began to create a new project with C++/Native support. I was going to trough [add native code -tutorial][native-code], but then I stuck to [Create a New Project with C/C++ Support][new-cpp-project] step 1, because my **"Include C++ Support"** checkbox was missing. I dug around, but couldn't find any help so the next step was to update everything.

I had Android Studio 2.1.3 and version 2.2.2 was available in the stable channel.

![Android Studio update notification](assets/android-studio-update-notification.png)

After the update I can see the checkbox when I create a new project.

!["Include C++ Support" checkbox is visible](assets/include-cpp-support-checkbox-is-visible.png)


[native-code]: https://developer.android.com/studio/projects/add-native-code.html#new-project
[new-cpp-project]: https://developer.android.com/studio/projects/add-native-code.html#new-project
