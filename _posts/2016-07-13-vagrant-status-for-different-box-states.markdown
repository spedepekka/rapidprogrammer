---
layout: post
title: "Vagrant status for different box states"
date: "2016-07-13 10:18:25 +0300"
permalink: /:title
---

Command `vagrant status` returns the state of the box. Here are the outputs for different states.

## Simple Python script to extract Vagrant status from the output

```python
import re
import sys

from subprocess import check_output, CalledProcessError

found_vm_state = False
try:
    vagrant_output_chars = check_output(["vagrant", "status"]).rstrip("\n")
    vagrant_output = []
    vagrant_output_line = ""
    for character in vagrant_output_chars:
        if character == "\n":
            if vagrant_output_line != "": # Append only lines which contain something
                vagrant_output.append(vagrant_output_line)
            vagrant_output_line = ""
        else:
            vagrant_output_line += character
    for line in vagrant_output:
        m = re.search("default\s*(.*)\s\(", line)
        if m:
            found_vm_state = True
            vagrant_status = m.group(1)
except CalledProcessError:
    pass # Supress non-existing Vagrantfile

if found_vm_state:
    print(vagrant_status)
else:
    print("Couldn't extract VM state")
```

## Vagrant status for different box states

### Folder without Vagrantfile

    A Vagrant environment or target machine is required to run this
    command. Run `vagrant init` to create a new Vagrant environment. Or,
    get an ID of a target machine from `vagrant global-status` to run
    this command on. A final option is to change to a directory with a
    Vagrantfile and to try again.

### The box is initialized, but not created yet (after vagrant init "mybox")

    default                   not created (vmware_fusion)

### The box is running (after vagrant up)

    default                   running (virtualbox)

### The box is suspended (after vagrant suspend)

For Virtualbox

    default                   saved (virtualbox)

For VMWare Fusion

    default                   suspended (vmware_fusion)

### The box is shut down

For Virtualbox

    default                   poweroff (virtualbox)

For VMWare Fusion

    default                   not running (vmware_fusion)
