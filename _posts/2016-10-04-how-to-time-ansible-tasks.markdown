---
layout: post
title: "How to time Ansible tasks"
date: "2016-10-04 09:41:30 +0300"
permalink: /:title
---

It is very helpful to see how long it takes to execute an Ansible task ans this is how I do it.

Write this to your `ansible.cfg`

```ini
[defaults]
callback_whitelist = profile_tasks
```

The location of the configuration file will be processed in the following order

1. ANSIBLE_CONFIG (an environment variable)
2. ansible.cfg (in the current directory)
3. .ansible.cfg (in the home directory)
4. /etc/ansible/ansible.cfg
