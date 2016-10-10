---
layout: post
title: "Android - How To Call Native Java Methods From Webview Javascript"
date: "2016-10-10 11:39:15 +0300"
permalink: /:title
---

In this post I'll show how to call native Java methods from webview Javascript. Complete [source code in Github][github].

## Background

I bumped into application which is an webview application where some of the native functionality has been exposed to webview. I wanted to learn how to do it by recreating it by myself. The code is pretty much copied from [this Google tutorial][google-doc].

## Add permissions

Add network permission to `AndroidManifest.xml`.

```xml
<uses-permission android:name="android.permission.INTERNET" />
```

## Add the webview

I created the project with Android Studio and used Basic Activity as the main activity so the file to add the webview is `content_main.xml`.

```xml
<WebView  xmlns:android="http://schemas.android.com/apk/res/android"
    android:id="@+id/webview"
    android:layout_width="fill_parent"
    android:layout_height="fill_parent"
/>
```

## Load some page in the webview

I want to get something visible as soon as possible so let's load http://rapidprogrammer.com

```java
WebView myWebView = (WebView) findViewById(R.id.webview);
myWebView.loadUrl("http://rapidprogrammer.com");
```

And the screen should look something like this (btw: see this [oneliner][screenshot] to take a screenshot).

<p style="text-align: center">
<img alt="Rapid Programmer site in webview on Android device" src="assets/rapid-programmer-webview.png" style="width: 40%"/>
</p>

## Enable Javascript in the webview

Don't forget to enable JS in the webview.

```java
WebSettings webSettings = myWebView.getSettings();
webSettings.setJavaScriptEnabled(true);
```

## Prepare the backend

I decided to serve HTML and JS to webview from Python Bottle. The server is self-explanatory and a bit out of the scope of this post so I won't go into details with that. The point is that it doesn't matter how the files get into the webview as long as they are fed into the webview somehow.

## The bridge/glue between Java and Javascript

### Java

The Java methods are exposed to Javascript with a special class. Here is an example. The exposed methods are annotated with `@JavascriptInterface` annotation. The documentation says the annotation is mandatory for API level 17 and above. I'm not sure, but I think before SDK 17 all the methods are exposed.

```java
public class WebAppInterface {
    Context mContext;

    /** Instantiate the interface and set the context */
    WebAppInterface(Context c) {
        mContext = c;
    }

    /** Show a toast from the web page */
    @JavascriptInterface
    public void showToast(String toast) {
        Toast.makeText(mContext, toast, Toast.LENGTH_SHORT).show();
    }
}
```

Add the class to webview with

```java
myWebView.addJavascriptInterface(new WebAppInterface(this), "Android");
```

The first parameter is the interface class with the context and the latter is the variable name to be used in Javascript.

### Javascript

```html
<input type="button" value="Say hello" onClick="showAndroidToast('Hello Android!')" />

<script type="text/javascript">
    function showAndroidToast(toast) {
        if(typeof Android !== "undefined" && Android !== null) {
            android.showToast(toast);
        } else {
            alert("Not viewing in webview");
        }
    }
</script>
```

In Javascript there is a global object called `Android`. The object name comes from the `addJavascriptInterface` method. That object has all the methods annotated in the interface class, in this example `showToast(String toast)`. The if statement is there to check if the object is set. With this if statement the site can be used from other browsers as well where the Android interface doesn't exist.

## Update the URL to your server

Update your IP address to webview, for example

```java
myWebView.loadUrl("http://10.113.24.171:8080");
```

Complete [source code in Github][github].


[github]: https://github.com/spedepekka/jsjava
[google-doc]: https://developer.android.com/guide/webapps/webview.html
[screenshot]: http://blog.shvetsov.com/2013/02/grab-android-screenshot-to-computer-via.html
