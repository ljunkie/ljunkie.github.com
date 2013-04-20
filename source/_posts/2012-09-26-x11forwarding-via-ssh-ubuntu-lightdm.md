---
title: 'X11forwarding via SSH &#8211; Ubuntu &#038; lightdm'
author: rob
layout: post
categories:
  - Linux
  - Ubuntu
comments: true
---
# 

**Some of the errors I ran into**

> - DISPLAY is not set  
- Failed to allocate internet-domain X11 display socket  
- X11 forwarding request failed on channel 0



1) **Make sure you have the X11forwarding enabled.**

Server: **/etc/ssh/sshd_config**  
```text /etc/ssh/sshd_config
X11Forwarding yes
X11DisplayOffset 10
AllowTcpForwarding yes`  
```

<!-- more -->


Client: **/etc/ssh/ssh_config**  
```
ForwardX11 yes
ForwardX11Trusted yes
```

2) **Enable lightdm to listen on TCP sockets**

Modify and add line below to: **/etc/lightdm/lightdm.conf**  
```
xserver-allow-tcp=true
```

3) **If you have disabled IPv6**, one more update to sshd_config is needed  
Modify and add line below to: **/etc/ssh/sshd_config**  
```
AddressFamily inet
```

4) **Restart lightdm and sshd – or – Reboot**