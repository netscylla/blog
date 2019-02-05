---
layout: post
title:  "Arris Cable Modem SNMP Enumeration"
description: "A walkthrough on analysing an Arris Cable Modem via SNMP"
date:   2019-02-04 11:49:33 +0000
tags: [Malware, Incident Response, Pentesting, Red Team, SNMP]
---
![Virginmedia Hub3 or Arris Cable Modem](/blog/assets/VMHH3.jpg)

## Intro
This blog post was inspired by recent problems with the new Virgin Media Hub v3.0. Specifically changing our network 
range to the non-default range.  

The default range is 192.168.0.1/24. However, this causes routing issues when we try to VPN
back to home-base when the client's range matches our native home-base range (despite having a specific VPN range); this
is why we usually configure an insane internal network range to avoid these possible collisions (e.g. our internal range may be 
192.168.38.1/24 - this is just an example). 

The Virgin Media interface is a simplified cut-down version compared to the original Arris and previously available 
versions and a lot of the more flexible features  (such as deciding your own network lan range).  
Looking at the community forums for other opinions and findings on this matter, revealed that some of these issues could be solved using SNMP.

So we started looking in Arris SNMP MIBS and found the following page useful in our research:
 * [https://mibs.observium.org/mib/ARRIS-ROUTER-DEVICE-MIB/](https://mibs.observium.org/mib/ARRIS-ROUTER-DEVICE-MIB/)

## SNMP Controls
The Virgin Hub is re-skinned Arris cable-modem router, but by using any modern browsers Web-developer toolset (Chrome, Firefox, Safari, Edge),
we can observed the AJAX (XHR) calls underneath to reveal the API calls we can utilise to access additional information:

![VHH XHR Calls](/blog/assets/VMHH3_xhr.png)

We extracted the following URL's, while clicking around and changing settings:
 * http://192.168.0.1/walk?oids=xxx&_n=xxx&_=xxx
 * http://192.168.0.1/snmpGet?oid=xxx&_n=xxx&_=xxx
 * http://192.168.0.1/snmpSet?oid=xxx&_n=xxx&_=xxx

We can use the above URLs to obtain, query and change settings on our router.

## Is this a vulnerability?
Sadly no, as to gain access or modify this information you need to be first authenticated with the device.

### Important
The following tokens/variables are obtained after a successful login:
 * _n
 * _

For best results, we recommend first logging into the HomeHub and copying these tokens from the underlying AJAX calls.

## Intersting Calls

### Router Configuration
 * http://192.168.0.1/walk?oids=.3.6.1.4.1.4115.1.20.1.1.5
 
![screenshot](/blog/assets/VMHH3_sc.png) 

### Read Firewall log 
 * http://192.168.0.1/walk?oids=1.3.6.1.4.1.4115.1.20.1.1.5.19.1.1.1.3;

### Router Logs 
 * http://192.168.0.1/walk?oids=1.3.6.1.4.1.4115.1.20.1.1.5.19.1.4.1.3;
 
### LAN Configuration
 * http://192.168.0.1/walk?oids=1.3.6.1.4.1.4115.1.20.1.1.2.2.1;
 
### Firewall Enabled True/False
 * http://192.168.0.1/walk?oids=1.3.6.1.4.1.4115.1.20.1.1.2.2.1.22
 
### UPnP Enabled True/False
 * http://192.168.0.1/walk?oids=1.3.6.1.4.1.4115.1.20.1.1.2.2.1.23
 
### LAN Client Information
 * http://192.168.0.1/walk?oids=1.3.6.1.4.1.4115.1.20.1.1.2.4
 
### Reveal WiFi Passwords
 * http://192.168.0.1/walk?oids=1.3.6.1.4.1.4115.1.20.1.1.3.26
 
## snmpGet
If you want to query an individual setting you can use snmpGet

Get arrisRouterSerialNumber : 
 * http://192.168.0.1/snmpGet?oid=1.3.6.1.4.1.4115.1.20.1.1.5.8.0;&_n=xxx&_=xxxxxxxx
```
{
"1.3.6.1.4.1.4115.1.20.1.1.5.8.0":"AAAP71111007"
}
```

## walk
Query DNS Settings
 * http://192.168.0.1/snmpGet?oid=1.3.6.1.4.1.4115.1.20.1.1.1.11.2.1.3;&_n=xxx&_=xxxxxxxx
```
"1.3.6.1.4.1.4115.1.20.1.1.1.11.2.1.3.1":"$c2a80464",
"1.3.6.1.4.1.4115.1.20.1.1.1.11.2.1.3.2":"$c2a80864",
```
The returned strings, e.g. $c2a80464 are hex-notation of the IP address, these can easily be decoded to the default Virgin Media DNS servers e.g. 194.168.4.100.

## snmpSet
Change the guest wifi name to "aaaaa"; Note you have to URL encode the string and start with %24:
* http://192.168.0.1/snmpSet?oid=1.3.6.1.4.1.4115.1.20.1.1.3.22.1.2.10004=%24616161616;&_n=xxx&_=xxxxxxxx

Before this setting is changed, we need to send an **Apply** signal:
* http://192.168.0.1/snmpSet?oid=1.3.6.1.4.1.4115.1.20.1.1.9.0=1;2;&_n=xxx&_=xxxxxxxx
If you tried these strings on your own Arris cable modem, your guest WiFi SSID should read "aaaaa"

Attempt Changing the DNS to 1.1.1.1:
* http://192.168.0.1/snmpSet?oid=1.3.6.1.4.1.4115.1.20.1.1.1.11.2.1.3.1=%2401010101;4&_n=xxx&_=xxxxxxxx
* http://192.168.0.1/snmpSet?oid=1.3.6.1.4.1.4115.1.20.1.1.9.0=1;2;&_n=xxx&_=xxxxxxxx

Despite trying to change the DNS settings to 1.1.1.1 - unfortunately they remain unchanged from the default settings. Hints on the Virgin community
forums are that this specific issue has been previously reported, and now a number of settings have been changed to read-only values.

This is backed up by the log entry (again using the snmpwalk API to obtain this info)

```
"1.3.6.1.4.1.4115.1.20.1.1.5.19.1.4.1.3.3":"[ERROR] [DOCSIS.SNMP(pid=538)]: OID: 1.3.6.1.4.1.4115.1.20.1.1.1.11.2.1.3.1 NOT WRITABLE"
```

## Changing the default network range
Set Gateway IP:
 * http://192.168.0.1/snmpSet?oid=1.3.6.1.4.1.4115.1.20.1.1.2.2.1.5.200=%24c0a802fe;4;&_n=XXXX

Set DHCP Start (192.168.2.1):
 * http://192.168.0.1/snmpSet?oid=1.3.6.1.4.1.4115.1.20.1.1.2.2.1.11.200=%24C0A80201;4;&_n=XXXX

Set DHCP End (192.168.2.50):
 * http://192.168.0.1/snmpSet?oid=1.3.6.1.4.1.4115.1.20.1.1.2.2.1.13.200=%24c0a80232;4;&_n=XXXX

Apply:
 * http://192.168.0.1/snmpSet?oid=1.3.6.1.4.1.4115.1.20.1.1.9.0=1;2;&_n=XXXX

## Conclusion
SNMP is a simple and powerful protocol for network management. It looks like Arris and Virgin Media are aware of the strengths
and weaknesses of this protocol and the API interface, and have made several settings read-only; Hopefully, to prevent attackers
from hijacking the cable modems for nefarious purposes; Or simply to stop customs messing with settings and bricking their modems.
