---
layout: post
title: "How to fix clipboard with iTerm and Vim/NeoVim on Mac"
date: "2016-06-13 09:26:16 +0300"
permalink: /:title
---

Add this to your Vim configuration file (`.vimrc`)

    set clipboard=unnamed

This makes yanking work with default Terminal, but to make it work with iTerm
you need to check *Allow clipboard access to terminal apps* in iTerm
preferences.
