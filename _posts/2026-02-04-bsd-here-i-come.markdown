---
layout: post
title: "BSD here I come"
date: "2026-02-04 09:08:00 +0300"
permalink: /:title
---

I somehow stumbled to BSD-world again and thought that maybe I would have some use cases for it. I've installed some BSD-variant long time ago, but not really used it for anything.

## Backstory

This time I had an oops-moment, when I configured Wireguard and accidentally opened router/firewall to world. I saw failed login attempt from router logs, which should not have been there, because it was external IP-address. My heart skipped a beat, because I didn't have too much logs for my router. First I thought the router has been open to world a long time and I would have to assume it was compromised and reinstall everything. Then I thought a moment and realized this might be because I updated firmware and configured Wireguard. Of course...I disabled one rule from firewall and it exposed too much to the world. So it was like tens of minutes open with good password, so perhaps I can rule out hacking.

I was cathing my breath while fixing the rules. What went wrong?

First of all, I didn't have proper logging propagation in place and the firmware upgrades removed old logs, so if I would have had to find out traces from hacking I wouldn't have anything to dig. So I needed a way to save the logs to other place from the router. Of course I learned a lot from firewalls and Wireguard on the way, but this post is about BSD. This would be the perfect use case to try it out.

## Which variant to choose?

There are two main BSD variants: [OpenBSD](https://www.openbsd.org/) and [FreeBSD](https://www.freebsd.org/). Of course there are more, but those are the ones suitable for my needs. It seems that OpenBSD is more security oriented and it uses UFS as filesystem. FreeBSD seems to have a bit more "modern stuff" in it like [ZFS](https://en.wikipedia.org/wiki/ZFS). This is not really about which variant is better or what filesystem is better. It's more about what I want to do and learn. So I went for FreeBSD to make things rapid.

## Fixing the logging problem

The installation of FreeBSD was quite pleasant experience. Nothing too fancy and everything worked smoothly. The only hiccups were my knowdledge base on BSD and mainly the commands etc.

I decided to run syslog-ng in the BSD to catch the logs from router and save the logs to safe place. There are many things to improve here like automate this with Ansible and make it possible to easily add a new host that can send logs to my logging server. My VM-setup doesn't really have too much free space or processing power so I decided not to go for ElasticSearch this time. I just want to fix this problem quickly and move to other things I want to do.

Setting up FreeBSD configurations was a bit hard, because things are configured a bit differently what I'm used to, but at the same time things were more organized. This got my attention. It seems that in BSD there are more single files in BSD than Linux that configure the system, like `/etc/rc.conf` and I like it. I have to mention that [BSD has very cool manual or handbook](https://docs.freebsd.org/en/books/handbook/) where you need to start instead of ChatGPT or similar. The page for [rc.conf](https://man.freebsd.org/cgi/man.cgi?rc.conf) is very long and I assume it covers pretty much everything. Also `pf` ([BSD firewall](https://man.freebsd.org/cgi/man.cgi?pf)) syntax was something to learn, but it is much easier that `iptables` rules.

The final boss was `Address already in use` problem. First I made sure my firewall rules and router configurations were correct with Netcat. But strangely, I was able to run Netcat in logging host without getting `Address already in use`. It seemed that my syslog-ng wasn't running at all. Configuring services and init-stuff is also quite pleasant in BSD. One doesn't have to select what init-system to use. There is just the BSD-way and that's it. It is also very easy to run a service with default configurations.

I restarted the service and it still kept giving me the problem. Maybe syslog-ng is conflicting with syslogd, which have similar capabilities? That wasn't the case, because the default setting was to run syslogd with `-ss` which disabled UDP-listener on port 514. Finally I figured out that my configurations were wrong. I had the default `udp()` earlier in the configuration and I tried to open it again. The easy fix was to remove the default one and go on.

Now I get my logs from the router to FreeBSD's syslog-ng at least. I could continue improving this system, but it is good enough for now.

P.s. I heard that there are "own" beard styles for Linux and BSD people. I don't know about that, but yeah, I have beard.

