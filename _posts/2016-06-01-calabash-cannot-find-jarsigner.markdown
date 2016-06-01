---
layout: post
title: "Calabash cannot find jarsigner"
date: "2016-06-01 10:04:40 +0300"
permalink: /:title
---

I tried to resign one Android apk with `calabash-android resign app.apk` in
OS X El Capitan. I use Java 8, but the version was not the problem.
First I updated calabash-android gem from 0.5.15 to 0.7.3, but it didn't
help.

```
macrotik:builds hackbox$ calabash-android resign app.apk
Unable to locate an executable at "/Library/Java/JavaVirtualMachines/jdk1.8.0_65.jdk/Contents/Home/jre/bin/jarsigner" (-1)
/Library/Ruby/Gems/2.0.0/gems/calabash-android-0.7.3/lib/calabash-android/java_keystore.rb:60:in `sign_apk': Could not sign app: /var/folders/gt/_pk0f_sd2hb_3jmjth9fm8_00000gn/T/d20160601-29261-pn66qb/unsigned.apk (RuntimeError)
        from /Library/Ruby/Gems/2.0.0/gems/calabash-android-0.7.3/lib/calabash-android/helpers.rb:163:in `sign_apk'
        from /Library/Ruby/Gems/2.0.0/gems/calabash-android-0.7.3/lib/calabash-android/helpers.rb:118:in `block in resign_apk'
        from /System/Library/Frameworks/Ruby.framework/Versions/2.0/usr/lib/ruby/2.0.0/tmpdir.rb:88:in `mktmpdir'
        from /Library/Ruby/Gems/2.0.0/gems/calabash-android-0.7.3/lib/calabash-android/helpers.rb:112:in `resign_apk'
        from /Library/Ruby/Gems/2.0.0/gems/calabash-android-0.7.3/bin/calabash-android:127:in `<top (required)>'
        from /usr/local/bin/calabash-android:23:in `load'
        from /usr/local/bin/calabash-android:23:in `<main>'
```

Let's find `jarsigner`

```
macrotik:builds hackbox$ find /Library/Java/JavaVirtualMachines/jdk1.8.0_65.jdk/ -name "jarsigner"
/Library/Java/JavaVirtualMachines/jdk1.8.0_65.jdk//Contents/Home/bin/jarsigner
```

Ok so it is there, but for some reason Calabash adds word *jre* to the path.
I took a look at the sources and there was no *jre* in the path.

```
macrotik:builds hackbox$ echo $JAVA_HOME
/Library/Java/JavaVirtualMachines/jdk1.8.0_65.jdk/Contents/Home/jre
```

`JAVA_HOME` was incorrectly set up. I fixed the env var to `~/.bash_profile` and
now it works.

```
export JAVA_HOME=/Library/Java/JavaVirtualMachines/jdk1.8.0_65.jdk/Contents/Home
```

and reload the changes

```
. ~/.bash_profile
```
