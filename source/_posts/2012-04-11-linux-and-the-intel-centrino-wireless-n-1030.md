---
title: Linux and the Intel Centrino Wireless-N 1030
author: rob
layout: post
categories:
  - Linux
comments: true
---
# 

These changes are stilled needed as of Linux 3.0.0-17. The driver in use is iwlagn and I have seen other posts using the iwlwifi driver (untested). 

**Turn off Power Managment for wireless**  
Info: This should already be off when using AC, however we will want to turn this feature off for battery use. 

To check you status run the command below and look for Power Management  
- check this output on **AC** and **Battery** to verify it’s **off**.

<!-- more -->
```
$ iwconfig wlan0
wlan0     IEEE 802.11bg  ESSID:"------------"
          Mode:Managed  Frequency:2.462 GHz  Access Point: 00:1C:B3:AE:FF:A1
          Bit Rate=54 Mb/s   Tx-Power=15 dBm
          Retry  long limit:7   RTS thr:off   Fragment thr:off
          Encryption key:off
          Power Management:off
          Link Quality=55/70  Signal level=-55 dBm
          Rx invalid nwid:0  Rx invalid crypt:0  Rx invalid frag:0
          Tx excessive retries:0  Invalid misc:396   Missed beacon:0
```

**To disable permanently** edit (as root) edit /usr/lib/pm-utils/power.d/wireless
```
# /usr/lib/pm-utils/power.d/wireless
Change line 39 from "power on” to “power off”.
```

**Turn off Wireless-N features of the card** 

I know, this sounds odd since we all want our N to work, but sadly N currently does not does not work correctly under linux.

This will unload the wifi module and load without wireless-N.  

```bash
sudo rmmod iwlwifi
sudo modprobe iwlwifi 11n_disable=1`
```

* If `11n_disable=1` causes the module to not load, try changing  it to `11n_disable50=1` 
* Persistent by adding to `/etc/modprobe.d/iwlwifi.conf`


** Finding which Kernel Module you are using  **-- iwlagn or iwlwifi --
```bash
Info: refer to the lines "Kernel driver in use" and " Kernel modules"  
`sudo lspci -v  | grep -A10 "Centrino Wireless"
03:00.0 Network controller: Intel Corporation Centrino Wireless-N 1030 (rev 34)
        Subsystem: Intel Corporation Centrino Wireless-N 1030 BGN
        Flags: bus master, fast devsel, latency 0, IRQ 53
        Memory at f1b00000 (64-bit, non-prefetchable) [size=8K]
        Capabilities: [c8] Power Management version 3
        Capabilities: [d0] MSI: Enable  Count=1/1 Maskable- 64bit 
        Capabilities: [e0] Express Endpoint, MSI 00
        Capabilities: [100] Advanced Error Reporting
        Capabilities: [140] Device Serial Number bc-77-37-ff-ff-0a-43-bc
        Kernel driver in use: iwlagn
        Kernel modules: iwlagn`
```

  
