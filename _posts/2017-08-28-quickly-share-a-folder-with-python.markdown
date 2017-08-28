---
layout: post
title: "Quickly share a folder with Python"
date: "2017-08-28 09:45:37 +0300"
permalink: /:title
---

## How to share a folder quickly to your colleague in the same local network (LAN)?

I found a way to share a folder quickly in the local network. There is no need to use Dropbox, One Drive, Google Drive or any other network share.

The command is here

```
python -m SimpleHTTPServer
```

The output looks similar to this

```
Serving HTTP on 0.0.0.0 port 8000 ...
127.0.0.1 - - [28/Aug/2017 09:49:06] "GET / HTTP/1.1" 200 -
127.0.0.1 - - [28/Aug/2017 09:50:14] "GET /my-test-file.txt HTTP/1.1" 200 -
```

And when someone goes to http://\<YOUR IP ADDRESS\>:8000 they will see something like this

<img alt="The folder share looks like this in the browser" src="assets/share-folder-with-python-simplehttpserver.png" style="width: 50%"/>
