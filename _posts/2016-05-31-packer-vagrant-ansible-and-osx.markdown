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

