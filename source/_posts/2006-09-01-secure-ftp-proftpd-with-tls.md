---
title: Secure FTP (ProFTPD with TLS)
author: rob
layout: post
categories:
  - Fedora
  - Linux
---
# 

I always wondered how hard it might be to get ftp running with encryption (preferably proFTPd since that has always been my choice ftp server).

I did realize how easy it was. Of course again this documentation really only refers to Fedora Core 5. I am sure you can take this info and put it towards any other linux os.

**proFTPd TLS extremely mini howto**

*  edit /etc/proftpd.conf  
```
    ## Add the Following lines.. or uncomment them# TLS  
    # Explained at http://www.castaglia.org/proftpd/modules/mod_tls.html  
    TLSEngine on  
    TLSRequired on  
    TLSRSACertificateFile /etc/pki/tls/certs/proftpd.pem  
    TLSRSACertificateKeyFile /etc/pki/tls/certs/proftpd-key.pem  
    TLSCipherSuite ALL:!ADH:!DES  
    TLSOptions NoCertRequest  
    TLSVerifyClient off  
    TLSLog /var/log/proftpd/tls.log
```
<!-- more -->

*  Create the proftp.pem and proftpd-key.pem files  
```   
     ## Run this command as root  
    openssl req -new -x509 -days 365 -nodes -out /etc/pki/tls/certs/proftpd.pem -keyout /etc/pki/tls/certs/proftpd-key.pem
```

*  Start the proFTPd service  
```
    ## Run the command as root  
    service restart proftpd
```

Continue below if your FTP server is behind a firewall or is part of a NAT

** Use the IP Nat Helper module **


/etc/sysconfig/iptables-config
```diff
-IPTABLES_MODULES=""
+IPTABLES_MODULES="ip_nat_ftp"
```


** OLD way **

* Allow passive ftp to work… need to specify what passive ports to use  
```
    edit /etc/proftpd.conf  
    ## Add the line below  
    PassivePorts 60000 60500
```
* Add the Passive ports through your firewall  
```
    edit /etc/sysconfig/iptables  
    ## Add the line below (may need some changing to fit the format)  
    -A RH-Firewall-1-INPUT -p tcp -m tcp –dport 60000:60050 -j ACCEPT
```