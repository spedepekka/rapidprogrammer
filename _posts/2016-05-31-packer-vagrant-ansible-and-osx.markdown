---
layout: post
title: "Packer, Vagrant, Ansible and OSX"
date: "2016-05-31 12:30:28 +0300"
permalink: /:title
---

## Issues

### Ansible (remote) Packer provider fails to connect to the box

Here is the complete log snippet

```
2016/05/30 15:50:26 ui: ==> vmware-iso: starting sftp subsystem
==> vmware-iso: starting sftp subsystem
2016/05/30 15:50:26 packer: 2016/05/30 15:50:26 opening new ssh session
2016/05/30 15:50:26 packer: 2016/05/30 15:50:26 starting remote command: /usr/lib/sftp-server -e
2016/05/30 15:50:26 packer: 2016/05/30 15:50:26 remote command exited with '127': /usr/lib/sftp-server -e
2016/05/30 15:50:26 packer: 2016/05/30 15:50:26 [INFO] RPC endpoint: Communicator ended with: 127
2016/05/30 15:50:26 [INFO] 54 bytes written for 'stderr'
2016/05/30 15:50:26 [INFO] 0 bytes written for 'stdout'
2016/05/30 15:50:26 [INFO] RPC client: Communicator ended with: 127
2016/05/30 15:50:26 [INFO] RPC endpoint: Communicator ended with: 127
2016/05/30 15:50:26 [INFO] 9 bytes written for 'stdin'
2016/05/30 15:50:26 packer: 2016/05/30 15:50:26 [INFO] 0 bytes written for 'stdout'
2016/05/30 15:50:26 packer: 2016/05/30 15:50:26 [INFO] 54 bytes written for 'stderr'
2016/05/30 15:50:26 packer: 2016/05/30 15:50:26 [INFO] RPC client: Communicator ended with: 127
2016/05/30 15:50:26 packer: 2016/05/30 15:50:26 [INFO] 9 bytes written for 'stdin'
2016/05/30 15:50:26 ui:     vmware-iso: fatal: [default]: UNREACHABLE! => {"changed": false, "msg": "ERROR! SSH Error: data could not be sent to the remote host. Make sure this host can be reached over ssh", "unreachable": true}
    vmware-iso: fatal: [default]: UNREACHABLE! => {"changed": false, "msg": "ERROR! SSH Error: data could not be sent to the remote host. Make sure this host can be reached over ssh", "unreachable": true}
2016/05/30 15:50:26 ui:     vmware-iso:
    vmware-iso:
2016/05/30 15:50:26 ui:     vmware-iso: PLAY RECAP *********************************************************************
    vmware-iso: PLAY RECAP *********************************************************************
2016/05/30 15:50:26 ui:     vmware-iso: default                    : ok=0    changed=0    unreachable=1    failed=0
    vmware-iso: default                    : ok=0    changed=0    unreachable=1    failed=0
2016/05/30 15:50:26 ui:     vmware-iso:
    vmware-iso:
2016/05/30 15:50:26 ui: ==> vmware-iso: shutting down the SSH proxy
```

I tried to figure this long time

```
2016/05/30 15:50:26 ui:     vmware-iso: fatal: [default]: UNREACHABLE! => {"changed": false, "msg": "ERROR! SSH Error: data could not be sent to the remote host. Make sure this host can be reached over ssh", "unreachable": true}
    vmware-iso: fatal: [default]: UNREACHABLE! => {"changed": false, "msg": "ERROR! SSH Error: data could not be sent to the remote host. Make sure this host can be reached over ssh", "unreachable": true}
```

The problem was actually couple lines before

```
2016/05/30 15:50:26 packer: 2016/05/30 15:50:26 remote command exited with '127': /usr/lib/sftp-server -e
```

This could mean a lot of things, but in this case the issue was `/usr/lib/sftp-server` command did not found.

**Solution**

Provide correct path with `sftp_command` to the provider. In OSX El Capitan the path is `/usr/libexec/sftp-server`.

Here is my provider json in the template

```json
    {
      "type": "ansible",
      "playbook_file": "./ansible/playbook.yaml",
      "sftp_command": "/usr/libexec/sftp-server -e",
      "ansible_env_vars": [
        "ANSIBLE_HOST_KEY_CHECKING=False"
      ]
    }
```

### Network drops for big downloads

I provision Packer with Ansible and I download big files like XCode from S3.
The download freezes in few moments with `read timeout` error.

I began to debug the issue by shooting up the box with packer
and adding a script which waits for `/tmp/continue.txt` file. I don't know
other way to leave the box running.

This is my `wait-for-file.sh` and I added it to my `template.json` with
other bash provision scripts.

```bash
#!/bin/bash

WAIT_FILE="/tmp/continue.txt"

while [ ! -f ${WAIT_FILE} ]
do
    sleep 2
    done
rm "${WAIT_FILE}"
```

Then I connected to the machine with SSH. I tried to download big file
with `curl -O <BIG_FILE>`. This freezes as well. Looks like it is a network
issue. The problem might be in VMWare Fusion NAT system.

I added another interface to the box with this snippet in my template.

```json
"ethernet0.present": "TRUE",
"ethernet0.connectionType": "nat",
"ethernet1.present": "true",
"ethernet1.startConnected": "true",
"ethernet1.virtualDev": "e1000",
"ethernet1.connectionType": "bridged"
```

Now `curl` still fails, because it uses `en0` by default which is the NAT
interface. Let's use `en1` with `curl`

```
curl --interface en1 -O <BIG_FILE>
```

### *shrink.sh* times out

The output looks like this

```
2016/06/03 13:14:32 [INFO] RPC client: Communicator ended with: 0
2016/06/03 13:14:32 [INFO] RPC endpoint: Communicator ended with: 0
2016/06/03 13:14:32 packer: 2016/06/03 13:14:32 [INFO] RPC client: Communicator ended with: 0
2016/06/03 13:14:32 packer: 2016/06/03 13:14:32 opening new ssh session
2016/06/03 13:14:32 packer: 2016/06/03 13:14:32 starting remote command: chmod +x /tmp/script_8825.sh; sudo PACKER_BUILD_NAME='vmware-iso' PACKER_BUILDER_TYPE='vmware-iso' /tmp/script_8825.sh
2016/06/03 13:14:33 ui:     vmware-iso: Please disregard any warnings about disk space for the duration of shrink process.
    vmware-iso: Please disregard any warnings about disk space for the duration of shrink process.
    vmware-iso: Progress: 100 [===========>]
    vmware-iso:
2016/06/03 13:15:02 ui:     vmware-iso: Progress: 100 [===========>]
2016/06/03 13:15:02 ui:     vmware-iso:
2016/06/03 13:15:48 packer: 2016/06/03 13:15:48 remote command exited with '0': chmod +x /tmp/script_8825.sh; sudo PACKER_BUILD_NAME='vmware-iso' PACKER_BUILDER_TYPE='vmware-iso' /tmp/script_8825.sh
2016/06/03 13:15:48 packer: 2016/06/03 13:15:48 [INFO] RPC endpoint: Communicator ended with: 0
2016/06/03 13:15:48 [INFO] 173372 bytes written for 'stdout'
2016/06/03 13:15:48 [INFO] 1 bytes written for 'stderr'
2016/06/03 13:15:48 [INFO] RPC client: Communicator ended with: 0
2016/06/03 13:15:48 [INFO] RPC endpoint: Communicator ended with: 0
2016/06/03 13:15:48 packer: 2016/06/03 13:15:48 [INFO] 173372 bytes written for 'stdout'
2016/06/03 13:15:48 packer: 2016/06/03 13:15:48 [INFO] RPC client: Communicator ended with: 0
2016/06/03 13:15:48 packer: 2016/06/03 13:15:48 [INFO] 1 bytes written for 'stderr'
2016/06/03 13:15:48 packer: 2016/06/03 13:15:48 opening new ssh session
2016/06/03 13:15:48 packer: 2016/06/03 13:15:48 ssh session open error: 'read tcp 192.168.56.1:59263->192.168.56.138:22: read: operation timed out', attempting reconnect
2016/06/03 13:15:48 packer: 2016/06/03 13:15:48 reconnecting to TCP connection for SSH
2016/06/03 13:16:03 packer: 2016/06/03 13:16:03 reconnection error: dial tcp 192.168.56.138:22: i/o timeout
2016/06/03 13:16:04 packer: 2016/06/03 13:16:04 Retryable error: Error removing temporary script at /tmp/script_8825.sh: dial tcp 192.168.56.138:22: i/o timeout
```

I've installed XCode to the box with Ansible provisioner. The shrink operation
seems to take too long. I can see the animated VMWare icon doing something
in the Dock. I took a look at timestamps and it seems to wait around 5 minutes.
`start_retry_timeout` is 5 minutes by default. First I removed the XCode.dmg
from the box after the installation and then I increased this timeout to
10 minutes.

```json
    {
      "execute_command": "chmod +x {{ .Path }}; sudo {{ .Vars }} {{ .Path }}",
      "scripts": [
        "../scripts/shrink.sh"
      ],
      "environment_vars": [
      ],
      "start_retry_timeout": "10m",
      "type": "shell"
    }
```

See the [Packer sources][packer-retry].

[packer-retry]: https://github.com/mitchellh/packer/blob/master/provisioner/shell/provisioner.go#L334-356
