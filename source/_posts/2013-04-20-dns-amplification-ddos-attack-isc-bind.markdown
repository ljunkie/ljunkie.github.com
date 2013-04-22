---
layout: post
title: "DNS Amplification DDoS Attack - ISC BIND"
date: 2013-04-20 18:17
comments: true
categories: 
---

** This example works for anyone running ISC BIND **

** dns attack isc.org any query **

I normally do not work with windows too much, but being on call this week I ended up having to fix a problem on a Windows 2008 server. I didn't find any documentation online, so I figured I'd add this post. 

For anyone running **Parallels Plesk** (unknown version, but I know our web admin always keeps these up to date) make sure you lock down your ISC BIND instance. If not, you will probably run into a DNS amplification attack which will cause **named.exe** to used ALL your memory and probably even crash. 

**2013-04-22 Update:** Plesk was set to only allow **_localnets_** recursion, however the built in **_localnets_** acl seems to be broken. 

`
"localnets" - matches all the IP address(es) and subnetmasks of the server on which BIND is running. For example, if the server has a single interface with an IP address of 192.168.2.3 and a netmask of 255.255.255.0 (or 192.168.2.2/24) then localnets will match 192.168.2.0 to 192.168.2.255 and 127.0.0.1 (the loopback is always present and has a single address, that is a netmask of 255.255.255.255). Some systems do not provide a way to determine the prefix lengths of local IPv6 addresses. In such a case, localnets only matches the local IP addresses, just like localhost though in this case it will apply to external and internal (same host) requests.
`

**Are you affected?** 

**tcpdump:**
```
12:28:00.121351 IP x.x.x.x.19135 > x.x.x.x.53: 10809+ [1au] ANY? isc.org. (36)
```

**bind logs:**
```
12:28:00.643 client x.x.x.x#49046: query: isc.org IN ANY +ED (x.x.x.x)
12:28:00.644 client x.x.x.x#25135: query: isc.org IN ANY +ED (x.x.x.x)
12:28:00.645 client x.x.x.x#19771: query: isc.org IN ANY +ED (x.x.x.x)
12:28:00.646 client x.x.x.x#44031: query: isc.org IN ANY +ED (x.x.x.x)
12:28:00.647 client x.x.x.x#31518: query: isc.org IN ANY +ED (x.x.x.x)
```

<!-- more -->

**test with dig:**

```bash NOTE: x.x.x.x is YOUR dns server ip
$ dig ANY isc.org @x.x.x.x +edns=0

; <<>> DiG 9.8.1-P1 <<>> ANY isc.org @x.x.x.x +edns=0
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 8809
;; flags: qr rd ra; QUERY: 1, ANSWER: 30, AUTHORITY: 4, ADDITIONAL: 13

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 4096
;; QUESTION SECTION:
;isc.org.                       IN      ANY

;; ANSWER SECTION:
isc.org.                300     IN      RRSIG   SPF 5 2 7200 20130515233253 20130415233253 50012 isc.org. rxkc3Rv5XIuARc+jiKh2DQIc3osweF+a7Db1OA8bbaEPfVW6eje0CrWM 1+D9gU0ghbNp4On4G4jfClcWTpefqhcGIzODMIwKXlGDcp1Y4t7f1Xt8 sAt0iDj7k4qoUOXEtcLoo3fhi0/HfjmJojNmNfzRuIy0q4VBPLvXDzY1 wio=
isc.org.                300     IN      SPF     "v=spf1 a mx ip4:204.152.184.0/21 ip4:149.20.0.0/16 ip6:2001:04F8::0/32 ip6:2001:500:60::65/128 ~all"
isc.org.                300     IN      RRSIG   DNSKEY 5 2 7200 20130515230132 20130415230132 12892 isc.org. By1JLKWH4p/NijvP7TO40IkAokI2o5w2tZlw1d92Iv7chSKkhnBlS0jh Hpo5IySLsr3yYmKnb5rv/lIMhlPVF5TUC3+ToY7hz6aouS5P4JYA1bIB SZlzxS5HAAPl3UddF4cwf5Dp3JON3E6VIzA588PMjUBD666A27JRNqut EbHI2WxnZBR9inxwDnEf5JPagEYgNMlADottLSa3PKxwtmWUS3OLZaOo 4+wMgbL+bqTI5h5y6IpOipz3rUWurFbYteTIy5RjC+uaLcazEM94G41Z YoXQP+LodcZTqiYnfbT0Cp3ahr3n+Kx3OHLglW/V5GoqyTDFjRrHtObc j1dA5A==
isc.org.                300     IN      RRSIG   DNSKEY 5 2 7200 20130515230132 20130415230132 50012 isc.org. GRdD8E7BJR+sD7V0MkBzWKeonk97axGU7sinBrc6szaons7LY1TWyn3T 5cllz850C6o5dlK22zykRjrwCI6wiuJVzuJbyAOjrwM3TtEjFv7ePAaK ad3VGofZeb0klGEtvG9L/5rMkF2bbAwxeFGuD7SPz1gsGKDurmbCQ8YD diw=
isc.org.                300     IN      DNSKEY  257 3 5 BEAAAAOhHQDBrhQbtphgq2wQUpEQ5t4DtUHxoMVFu2hWLDMvoOMRXjGr hhCeFvAZih7yJHf8ZGfW6hd38hXG/xylYCO6Krpbdojwx8YMXLA5/kA+ u50WIL8ZR1R6KTbsYVMf/Qx5RiNbPClw+vT+U8eXEJmO20jIS1ULgqy3 47cBB1zMnnz/4LJpA0da9CbKj3A254T515sNIMcwsB8/2+2E63/zZrQz Bkj0BrN/9Bexjpiks3jRhZatEsXn3dTy47R09Uix5WcJt+xzqZ7+ysyL KOOedS39Z7SDmsn2eA0FKtQpwA6LXeG2w+jxmw3oA8lVUgEf/rzeC/bB yBNsO70aEFTd
isc.org.                300     IN      DNSKEY  256 3 5 BQEAAAABwuHz9Cem0BJ0JQTO7C/a3McR6hMaufljs1dfG/inaJpYv7vH XTrAOm/MeKp+/x6eT4QLru0KoZkvZJnqTI8JyaFTw2OM/ItBfh/hL2lm Cft2O7n3MfeqYtvjPnY7dWghYW4sVfH7VVEGm958o9nfi79532Qeklxh x8pXWdeAaRU=
isc.org.                300     IN      RRSIG   NSEC 5 2 3600 20130515233253 20130415233253 50012 isc.org. oBP67HE1s2DNfMZ7z/MR6aY2Ujhq7PNY4eMuw6Y+i+qXbUbQ48T2O6Kj ATc9heRYWryRFQNj/JXvhDZvB4/4WsdgGqy7GdbNtBM5tSTKyp+/omc2 Bzg2y70TQgwoXyMnqCTUh/L5OAvLO8kk0xY1mbDSiv7l+gSAqQws876R zmI=
isc.org.                300     IN      NSEC    _adsp._domainkey.isc.org. A NS SOA MX TXT AAAA NAPTR RRSIG NSEC DNSKEY SPF
isc.org.                300     IN      RRSIG   NAPTR 5 2 7200 20130515233253 20130415233253 50012 isc.org. k+sm/1z+7Tp+cYeZL/IZHfGT4gVXG3Lto7n1bxCU34hh1DtuKYWXYra4 UdSpZ8lrFu+y4BMTVtR9eoDI2azQCbwJwU9E+btAsf7ZYRYONY7YkttZ l3iqckDHfNPb80/o25QRDX1VejYq9oSfOiKRVNjCvR9YvUMptPZJEn06 iYQ=
isc.org.                300     IN      NAPTR   20 0 "S" "SIP+D2U" "" _sip._udp.isc.org.
isc.org.                300     IN      RRSIG   AAAA 5 2 7200 20130515233253 20130415233253 50012 isc.org. M3h/PV6Fq3U0g45Z3FiwG+xcsnAok2T/nJwis4x/5MCuGk1wPRj1uUI+ h8nPPXkG9fq40uWV8hd7zKvZ5mSO0sDgdFDdkZLyPtG4jv67nw7/vUwb Pm3cqxTcMCSdKpPQvFXhq4X+bWdYJoNwNHGtviimMse8fdUER6dZqhmB Cqc=
isc.org.                300     IN      AAAA    2001:4f8:0:2::d
isc.org.                300     IN      RRSIG   TXT 5 2 7200 20130515233253 20130415233253 50012 isc.org. bdioDXnXaXz6+DRzq0WSEihIBcs7/5xbS7YzcEeCNdeq7c+dpD3SBNxj mG40ic0YH0ADYzXSi+MrgESZoQZmzUmBvw84FJcxtv1OHP1RYun9Scny acpx0U6SGhqvNFs4UaiJmPDYyJFaJcbGk47bQgTfbKDzDyCjhmbuEaVf 1fM=
isc.org.                300     IN      TXT     "v=spf1 a mx ip4:204.152.184.0/21 ip4:149.20.0.0/16 ip6:2001:04F8::0/32 ip6:2001:500:60::65/128 ~all"
isc.org.                300     IN      TXT     "$Id: isc.org,v 1.1793 2013-04-09 00:33:46 bind Exp $"
isc.org.                300     IN      RRSIG   MX 5 2 7200 20130515233253 20130415233253 50012 isc.org. HdmWOHdVRLEiKIgkeGAueVl5IhUVL2b8VddQwUAatNGIidhPHkSuylE4 E8+uh2qN/sTeYPI6DN7exvKvz+4i1dsO8lUXJqY2JY8JRizRb6oBPhgP wySTiM6mAfOzB6LFMy6bqb3IYiOApuToT/785I2WxzNPK0vmxGh29nH7 Tsc=
isc.org.                300     IN      MX      10 mx.pao1.isc.org.
isc.org.                300     IN      MX      10 mx.ams1.isc.org.
isc.org.                300     IN      RRSIG   SOA 5 2 7200 20130515233253 20130415233253 50012 isc.org. Qw/kiSPp8fgeYDucYo/tJP+SuQEWMoXJM9XPAgvuUj4ek44hUxjNUo2p 1VpLXaTIUnv6WdBvgo7X8wdPJckRag56y2zJSVbrAS3PrIALj9j90pbB 55RRc0xaGNusBU27orI4acFEZCpIH8yNv7zLx1MX3cWohoCisWaQMBwB Syo=
isc.org.                300     IN      SOA     ns-int.isc.org. hostmaster.isc.org. 2013041600 7200 3600 24796800 3600
isc.org.                300     IN      RRSIG   NS 5 2 7200 20130515233253 20130415233253 50012 isc.org. HUXmb89gB4pVehWRcuSkJg020gw2d8QMhTrcu1ZD7nKomXHQFupXl5vT iq5VUREGBQtnT7FEdPEJlCiJeogbAmqt3F1V5kBfdxZLe/EzYZgvSGWq sy/VHI5d+t6/EiuCjM01UXCH1+L0YAqiHox5gsWMzRW2kvjZXhRHE2+U i1Q=
isc.org.                300     IN      RRSIG   A 5 2 7200 20130515233253 20130415233253 50012 isc.org. LU6oTkxGAxOlX6zGFzwW+OSOXDZOi0NbPQGnF/evVZ+4N5FUN+7uApXP cTiwiXnX41JOqCpQyb/5zkUNyrl99/g5quVeOan06gBzsJfK8EoOSrOK UwbOKoJC1uWisJvIVS9+sb729LJ3bnAOFw0SDc22KLSaFKiwuKeTCtV6 Tuk=
isc.org.                300     IN      A       149.20.64.42
isc.org.                297     IN      RRSIG   DS 7 2 86400 20130508155535 20130417145535 31380 org. Fnk6ZJwwnJXTyIfgpFsax5dOeLFsEVg08GVeAgAaKiX/h2Oo4GZ/iVWu X9EAVCiL+hRCcpVsKED3Le+hzV4nazBtITsxTso3UTtWdJ49jJSRfsld 9Tp+5/Slf5dzwU4ST0d5VO4V5XN7mxTbIuAn9hpQ5ujVdZHjQYw/FkAU Bek=
isc.org.                297     IN      DS      12892 5 2 F1E184C0E1D615D20EB3C223ACED3B03C773DD952D5F0EB5C777586D E18DA6B5
isc.org.                297     IN      DS      12892 5 1 982113D08B4C6A1D9F6AEE1E2237AEF69F3F9759
isc.org.                297     IN      NS      ord.sns-pb.isc.org.
isc.org.                297     IN      NS      sfba.sns-pb.isc.org.
isc.org.                297     IN      NS      ams.sns-pb.isc.org.
isc.org.                297     IN      NS      ns.isc.afilias-nst.info.

;; AUTHORITY SECTION:
isc.org.                297     IN      NS      ams.sns-pb.isc.org.
isc.org.                297     IN      NS      ord.sns-pb.isc.org.
isc.org.                297     IN      NS      sfba.sns-pb.isc.org.
isc.org.                297     IN      NS      ns.isc.afilias-nst.info.

;; ADDITIONAL SECTION:
mx.ams1.isc.org.        300     IN      A       199.6.1.65
mx.ams1.isc.org.        300     IN      AAAA    2001:500:60::65
mx.pao1.isc.org.        300     IN      A       149.20.64.53
mx.pao1.isc.org.        300     IN      AAAA    2001:4f8:0:2::2b
ns.isc.afilias-nst.info. 299    IN      A       199.254.63.254
ns.isc.afilias-nst.info. 299    IN      AAAA    2001:500:2c::254
ams.sns-pb.isc.org.     297     IN      A       199.6.1.30
ams.sns-pb.isc.org.     297     IN      AAAA    2001:500:60::30
ord.sns-pb.isc.org.     297     IN      A       199.6.0.30
ord.sns-pb.isc.org.     297     IN      AAAA    2001:500:71::30
sfba.sns-pb.isc.org.    297     IN      A       149.20.64.3
sfba.sns-pb.isc.org.    297     IN      AAAA    2001:4f8:0:2::19

;; Query time: 4300 msec
;; SERVER: x.x.x.x#53(x.x.x.x)
;; WHEN: Sat Apr 20 18:20:04 2013
;; MSG SIZE  rcvd: 3628
```


** To fix this and lock down BIND  ** edit: C:\Parallels\Plesk\dns\etc\named.user.conf


```text Disable recursion for the DNS service 
options {
    recursion no;
};
```

```text ONLY Allow queries from known networks
options {
    allow-query {192.168.1.0/24;};
};
```

* Optionally, you can also use ACL's for a _cleaner_/_easier_ config

```text notice we also allow the trusted acl recursion
acl trusted {
        127.0.0.1;
        127.0.0.0/24;
        172.16.0.0/16;
        192.168.1.0/24;
}

options {
    allow-recursion	 {trusted; }; 
    allow-query		 {trusted; }; 
};
```

** 2013-04-22 Update: ** Plesk will overwrite "allow-recursion { trusted; };" with the selection you have set in the web gui dns configuration with  any, localnets or none. If you choose localnets (good choice), this feature may not work properly, so you will have to set the named.user.conf file immutable (read-only). If you do not, your changes *will revert. 


** Now with this in place, here is the query again ** 
```
dig ANY isc.org @x.x.x.x +edns=0

; <<>> DiG 9.8.1-P1 <<>> ANY isc.org @x.x.x.x +edns=0
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: REFUSED, id: 53084
;; flags: qr rd; QUERY: 1, ANSWER: 0, AUTHORITY: 0, ADDITIONAL: 1
;; WARNING: recursion requested but not available

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 4096
;; QUESTION SECTION:
;isc.org.                       IN      ANY

;; Query time: 50 msec
;; SERVER: x.x.x.x#53(x.x.x.x)
;; WHEN: Sat Apr 20 18:08:19 2013
;; MSG SIZE  rcvd: 36
```

```text nslookup tool... I know
$ nslookup isc.org x.x.x.x
Server:         x.x.x.x
Address:        x.x.x.x#53

** server can't find isc.org: REFUSED
```


There you have it. You should take a look at [http://blog.cloudflare.com/deep-inside-a-dns-amplification-ddos-attack](http://blog.cloudflare.com/deep-inside-a-dns-amplification-ddos-attack) if you want a more detailed break down of this attack. I have better things to do now... like have a beer. 
