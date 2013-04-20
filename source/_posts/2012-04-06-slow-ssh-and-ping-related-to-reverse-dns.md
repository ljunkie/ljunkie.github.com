---
title: Slow SSH and PING related to Reverse DNS
author: rob
layout: post
categories:
  - Linux
comments: true
---
# 

** info: **: slow ssh authentication, pings slow/timeout when using fqdn

** issue: **  SSH just sits here for a while when trying to connect.
```bash
strace ssh myhostname ...
write(5, "RESOLVE-ADDRESS 192.168.1.1n", 27) = 27
...
```

**Reason:** mdns listed in nsswitch.conf doesn’t allow reverse dns to return failure immeditately when DNS lookup return NXDOMAIN.

**Resolution:**  modify  /etc/nsswitch.conf

```diff
- hosts:          files mdns4_minimal [NOTFOUND=return] dns mdns4
  hosts: files mdns4_minimal [NOTFOUND=return] dns [NOTFOUND=return] mdns4
```

