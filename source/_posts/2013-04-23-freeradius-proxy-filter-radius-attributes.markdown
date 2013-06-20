---
layout: post
title: "FreeRADIUS Proxy - Filter Radius Attributes"
date: 2013-04-23 18:18
comments: true
categories: 
---

#Version
>* Requires 2.x
* freeradius2-2.1.12-5.el5


```bash
#centos 5.x (must specify freeradius2 otherwrite 1.1.x will be installed)
yum install freeradius2 freeradius2-utils

#centos 6.x (2.x branch is default)
yum install freeradius freeradius-utils

# ubuntu
apt-get install freeradius freeradius-utils
```




#Reason 
>* To allow an offsite vendor control of radius, but limit their ability to supply bad radius attribuites. 
* MAIN issue: '''Protect your network''' from disallowing the vendor to supply a misconfigured '''FRAMED-IP-ADDRESS''' and/or '''FRAMED-ROUTE''' that could be injected into OSPF or whatever routing protocol you might use.  


This is accomplished with the **_rlm_attr_filter_** [FreeRADIUS Module](http://freeradius.org/radiusd/man/rlm_attr_filter.html)


`The rlm_attr_filter module exists for filtering certain attributes and values in received ( or transmitted ) radius packets. It gives the server a flexible framework to filter the attributes we send to or receive from home servers or NASes. This makes sense, for example, in an out-sourced dialup situation to various policy decisions, such as restricting a client to certain ranges of Idle-Timeout or Session-Timeout.`





#Config files 
>* Vendor Name: '''rarforge.com'''   (we'll use that for the realm)
* Allowed Framed-IP-Address: '''10.0.0.x''' and '''192.168.5.x'''
* Allowed Framed-Netmask: '''255.255.255.255'''
* Allowed Framed-Route: '''NONE'''
* Framed-Filter-ID: '''NONE''' -- login will fail if access-list doesn't exist.



#####/etc/raddb/clients.conf
>* Update your clients secret - for now we will just be testing from localhost. 

```text /etc/raddb/clients.conf
client localhost {
...
secret = badsecret
...
}
```

<!--more-->

#####/etc/raddb/radiusd.conf
>* Listen on 21000 for auth
* Listen on 21001 for acct

```text /etc/raddb/radiusd.conf
...
### realm rarforge.com
listen {
        ipaddr = *
        port = 21000
        type = auth
}
listen {
        ipaddr = *
        port = 21001
        type = acct
}
...
```



#####/etc/raddb/attrs
>* This is where we remove/disallow radius attributes from the vendor sent to the client
* make sure to keep a close eye on your comments in the config. Remove them if you have parsing errors. Last rule must not end with a comma. 

```text /etc/raddb/attrs
...
rarforge.com
        Service-Type =* ANY,
        Login-Service =* ANY,
        Login-TCP-Port =* ANY,
        Framed-Protocol =* ANY,
        Framed-Compression =* ANY,
        Framed-MTU =* ANY,
        Reply-Message =* ANY,
        Proxy-State =* ANY,
        Session-Timeout =* ANY,
        Port-Limit =* ANY,
        Idle-Timeout =* ANY,
############ DENY BELOW ####################
# ; comments must begin with '#'-- NO SPACE
#       ; ONLY ALLOW 10.0.0.x and 192.168.5.X
        Framed-IP-Address =~ "10\.0\.0\.|192\.168\.5\.",
#       ; /32 ONLY
        Framed-IP-Netmask == 255.255.255.255,
#
#       ; LAST
        Framed-Filter-ID !* ANY
...
```



#####/etc/raddb/proxy.conf
>* enable the realm (rarforge.com) to be proxied to the vendors radius auth/acct server

```text /etc/raddb/proxy.conf
...
realm rarforge.com {
        type            = radius
        authhost        = vendor_radius_auth_ip:1645
        accthost        = vendor_radius_acct_ip:1646
        secret          = <radius secret>
}
...
```




#####/etc/raddb/sites-enabled/default
>* set realm to rarforge.com based on the Destination Port auth/acct port (21000/210001)
* This is optional if you require users to be user@realm. In my case, we had users authing with a realm.  

```text /etc/raddb/sites-enabled/default
...
authorize {
 ## rarforge realm
 if (Packet-Dst-Port == "21000") {
  update control {
    Proxy-To-Realm := "rarforge.com"
    }
 }
 if (Packet-Dst-Port == "21001") {
  update control {
    Proxy-To-Realm := "rarforge.com"
    }
  }
 ...
```


<br/>
---
<br/>

# Testing Proxy
```bash
radtest username <password> localhost:21000 1 badsecret

Sending Access-Request of id 85 to 127.0.0.1 port 21000
        User-Name = "username"
        User-Password = "<password>"
        NAS-IP-Address = 127.0.0.1
        NAS-Port = 1
        Message-Authenticator = 0x00000000000000000000000000000000
rad_recv: Access-Accept packet from host 127.0.0.1 port 21000, id=85, length=50
        Framed-IP-Address = 10.0.0.5
        Framed-Netmask = 255.255.255.255 
        Idle-Timeout = 600
        Session-Timeout = 18000
        Service-Type = Framed-User
        Port-Limit = 1

## SUCCESS!
```


<br/>
---
<br/>
# Troubleshooting

* Verify with radtest you can auth from the server running freeradius to the vendors radius server. It could be firewalled, not in their client list, etc...

* try appending your realm to the username username@yourrealmname. Maybe the section in '''/etc/raddb/sites-enabled/default''' is not working.

