---
title: 'VMware errors &#8211; FC6 and/or 2.6.19 Kernel'
author: rob
layout: post
categories:
  - Fedora
  - Linux
---
# 

If you are using 2.6.19 kernel, then you will notice some errors with the vmware-config.pl script. If… actually, you better be using the 2.6.19 kernel now, or whatever the latest release is as of now

```
 Error runninging vmware-config.pl :  
 /tmp/vmware-config2/vmnet-only/userif.c: In function ‘VNetCopyDatagramToUser’:  
 /tmp/vmware-config2/vmnet-only/userif.c:629: error: ‘CHECKSUM_HW’ undeclared (first use in this function)  
 /tmp/vmware-config2/vmnet-only/userif.c:629: error: (Each undeclared identifier is reported only once  
 /tmp/vmware-config2/vmnet-only/userif.c:629: error: for each function it appears in.)  
 make[2]:   [/tmp/vmware-config2/vmnet-only/userif.o] Error 1  
 make[1]:   [\_module\_/tmp/vmware-config2/vmnet-only] Error 2  
 make[1]: Leaving directory `/usr/src/kernels/2.6.19-1.2911.fc6′  
 make: [vmnet.ko] Error 2  
 make: Leaving directory `/tmp/vmware-config2/vmnet-only’  
 Unable to build the vmnet module.
```


It seems CHECKSUM\_HW has been remove from 2.6.19 and replaced with CHECKSUM\_PARTIAL. You can either replace any CHECKSUM\_HW lines with CHECKSUM\_PARTION, or you can use the patch that Robin Kearney (usefulthings.org.uk) has writtenHow to use the Patch:

<!-- more -->


**FIX** – As root:

*   cd /tmp
*   wget http://usefulthings.org.uk/wp-content/uploads/vmnet-only-2.6.19.patch
*   cd /usr/lib/vmware/modules/source/
*   cp vmnet.tar vmnet.tar.orig
*   tar xf vmnet.tar
*   patch -p0 < /tmp/vmnet-only-2.6.19.patch
*   tar cf vmnet.tar vmnet-only
*   vmware-config.pl (if OS is FC6, this will probably fail again)**  
    **

**Fedora Core 6 will most likely have another error we need to fix:**

```
 Using 2.6.x kernel build system.  
 make: Entering directory `/tmp/vmware-config1/vmnet-only’  
 make -C /lib/modules/2.6.19-1.2911.fc6/build/include/.. SUBDIRS=$PWD SRCROOT=$PWD/. modules  
 make[1]: Entering directory `/usr/src/kernels/2.6.19-1.2911.fc6-i686′  
 CC [M] /tmp/vmware-config1/vmnet-only/driver.o  
 CC [M] /tmp/vmware-config1/vmnet-only/hub.o  
 CC [M] /tmp/vmware-config1/vmnet-only/userif.o  
 CC [M] /tmp/vmware-config1/vmnet-only/netif.o  
 CC [M] /tmp/vmware-config1/vmnet-only/bridge.o  
 CC [M] /tmp/vmware-config1/vmnet-only/procfs.o  
 /tmp/vmware-config1/vmnet-only/procfs.c:33:26: error: linux/config.h: No such file or directory  
 make[2]: \***| [/tmp/vmware-config1/vmnet-only/procfs.o] Error 1  
 make[1]: \***| [\_module\_/tmp/vmware-config1/vmnet-only] Error 2  
 make[1]: Leaving directory `/usr/src/kernels/2.6.19-1.2911.fc6-i686′  
 make: \***| [vmnet.ko] Error 2  
 make: Leaving directory `/tmp/vmware-config1/vmnet-only’  
 Unable to build the vmnet module.
```

**FIX** – as root:

*   cd /usr/src/kernels/2.6.19-1.2911.fc6-i686/include/linux (or whatever kernel version you are using)
*   touch config.h
*   vmware-config.pl (should actually complete this time)