---
layout: post
title: "Vagrant and filesystem watchers"
date: "2017-01-13 14:25:04 +0200"
permalink: /:title
---

I like to set up a Vagrant box for development and run stuff inside the box. Usually I
do the source code modifications in the host and I have some sort of file watcher
f.e. `truffle serve` running in the VM. At least if the host is Mac and the VM is Ubuntu,
the VM does not notice if the file is changed from the host. Here is a fix for that.

Install `vagrant-notify-forwarder` plugin and reload the box.

```
vagrant plugin install vagrant-notify-forwarder
vagrant reload
```

Then add this to `Vagrantfile`

```
  if Vagrant.has_plugin?("vagrant-notify-forwarder")
    config.notify_forwarder.enable = true
  else
    puts "WARNING: Install vagrant-notify-forwarder for better watch support"
  end
```

For more details about the plugin go to [https://github.com/mhallin/vagrant-notify-forwarder](https://github.com/mhallin/vagrant-notify-forwarder)
