---
layout: post
title: "Android EditText password and UI Automator"
date: "2016-10-18 10:46:05 +0300"
permalink: /:title
---

I just figured out that if you have EditText field in Android application and you try to check if the password is set correctly with UI Automator, `getText()` will return empty string.

Here is my password field

```xml
<EditText
    android:id="@+id/eb_et_password"
    android:layout_width="0dp"
    android:layout_height="wrap_content"
    android:layout_weight="1"
    android:imeOptions="actionGo"
    android:inputType="textPassword" />
```

And here is the UI Automation test code

```java
String password = mDevice.findObject(new UiSelector().resourceId(getId("eb_et_password"))).getText();
```

The password string above will be empty.

If you remove `android:inputType="textPassword"` from the EditText, the test code will work and return the password string in the EditText field.

I haven't found a solution for this yet, if I find it I'll update this post.
