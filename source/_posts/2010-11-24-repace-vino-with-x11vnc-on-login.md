---
title: Repace VINO with x11vnc on Login
author: rob
layout: post
categories:
  - Linux
  - Ubuntu
---
# 

Quick guide how to use VINO server and install x11vnc… 

```
sudo apt-get purge vino
sudo killall -9 vino-server
sudo apt-get install x11vnc
x11vnc -storepasswd
```

* edit file **/etc/gdm/custom.conf**
```
## add right below [daemon]
KillInitClients=false`
```

* edit file **/etc/gdm//Init/Default**
```
# add right before the end of the file "exit 0"
/usr/bin/x11vnc -rfbauth  -o /var/log/x11vnc.log -shared -forever -bg -rfbport 5900`
```