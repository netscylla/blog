---
layout: post
title:  "Whereis - The whois miner"
description: "A script for mining net-ranges from whois records given a specific keyword"
date:   2019-05-23 13:49:33 +0000
tags: [Threat Intel, Red Team, Pentest, DevSecOps]
---

![suspicious word doc](/assets/miner.png)

## The common problem
For a number of years during our red-teaming, and major infosec panics (such as eternal-blue, blue-keep etc), 
clients have asked us on their behalf to scan their entire range. However, some Infosec professionals and CISO's
struggle to know the extent of their publicly accessible networks, due to inherited infrastructure (M&A (Merges and Acquisitions),
ever expanded cloud infrastructure, change of providers, poor documentation etc). So identifying net-ranges can become a tricky task:
 * wrong subnets could be scanned
 * entire net-ranges could be missed
 * 3rd party providers/ISPs with dedicated infrastructure is missed.

## Our Solution
Netscylla's solution to the problem is to *'mine'*, the Whois registrars for the organisation's name(s). In an attempt to quickly obtain the breath of the network.
Now we said it was tricky... But today we release our in-house tool whereis, in a hope that it can make techies jobs easier in locating
the net-ranges of their publicly accessible infrastructure. 

### Example 1 - example keyword
```
$ ./whereis.py example
23.30.98.184/29
76.12.11.152/29
76.12.110.96/29
64.136.255.92/30
```
We can then perform a manual check of each CIDR to confirm example keyword is actually returned:
```
$ whois -a 23.30.98.184
NetRange:       23.30.98.184 - 23.30.98.191
CIDR:           23.30.98.184/29
NetName:        IMPORTEXAMPLES   <---- matches here

$ whois -a 76.12.11.152
NetRange:       76.12.11.152 - 76.12.11.159
CIDR:           76.12.11.152/29
NetName:        EXAMPLEESSAYS    <---- matches here

$ whois -a 64.136.255.92
NetRange:       64.136.255.92 - 64.136.255.95
CIDR:           64.136.255.92/30
NetName:        CTSC-S3362
NetHandle:      NET-64-136-255-92-1
Parent:         CTSTELECOM-BLK-1 (NET-64-136-224-0-1)
NetType:        Reassigned
OriginAS:       
Customer:       Digital Example (C06469865)   <---- matches here
```

**Warning** - as the script is only matching records by a given keyword, results could be gratuitous.  
When scanning for your own organisation name (as expected), its best to double check the results, as other organisations
with similar name could be included in the results.

### Example 2 - encapsulated string
```
$ ./whereis.py "private bank"
173.8.132.248/29
12.130.41.144/29
216.47.232.224/28
...
```
Again we perform a manual check, to confirm the results:
```
$ whois -a 173.8.132.248

NetRange:       173.8.132.248 - 173.8.132.255
CIDR:           173.8.132.248/29
NetName:        BORELPRIVATEBANKTRUS
```

## The code
We have hosted our code on github:
 * [https://github.com/netscylla/whereis](https://github.com/netscylla/whereis)

## Disclaimer
Netscylla or its staff cannot be held responsible for any abuse relating from this blog post. 
This post is to raise awareness in network mapping publicly accessible infrasture linked to specific organisations or keyword patterns. REMEMBER: It is illegal to attempt unauthorised access on any system you do not personally own, unless you have explicit permission in writing from the system owner!