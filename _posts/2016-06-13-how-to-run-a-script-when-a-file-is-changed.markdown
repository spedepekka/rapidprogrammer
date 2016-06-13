---
layout: post
title: "How to run a script when a file is changed"
date: "2016-06-13 13:38:54 +0300"
permalink: /:title
---

There are many ways to achieve this, but I prefer **fswatch** for simple stuff.

    fswatch -o <FILE> | xargs -n1 <SCRIPT>

For example if I edit a file and want to deploy it on save

    fswatch -o example.py | xargs -n1 ./deploy-example.py
