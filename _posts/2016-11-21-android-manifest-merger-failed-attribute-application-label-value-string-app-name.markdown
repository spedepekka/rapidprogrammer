---
layout: post
title: "Android: Manifest merger failed: Attribute application@label value=(@string/app_name)"
date: "2016-11-21 14:37:29 +0200"
permalink: /:title
---

## Manifest merger failed

I got this error when I used Android library in my Android project

```
Android Manifest merger failed: Attribute application@label value=(@string/app_name)
```

## Solution

This means that you have multiple `AndroidManifes.xml` files where the files cannot be automatically merged. For more see [Android developer][android-dev-manifest-merge].

The error also says

```
Suggestion: add 'tools:replace="android:label"' to <application> element at AndroidManifest.xml:A:B-C:D to override.
```

I modified the library project from

```java
<?xml version="1.0" encoding="utf-8"?>
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    package="com.example">

    <application
        android:allowBackup="true"
        android:label="@string/app_name"
        android:supportsRtl="true">
        <activity android:name=".SomeActivity"/>
    </application>
</manifest>
```

To

```java
<?xml version="1.0" encoding="utf-8"?>
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    package="com.example"
    xmlns:tools="http://schemas.android.com/tools">

    <application
        android:allowBackup="true"
        android:label="@string/app_name"
        android:supportsRtl="true"
        tools:replace="android:label">
        <activity android:name=".SomeActivity"/>
    </application>
</manifest>
```

And now the `android:label` is replaced by the label from the parent application.


[android-dev-manifest-merge]: https://developer.android.com/studio/build/manifest-merge.html

