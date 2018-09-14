---
layout: post
title:  "Cracking WPA1/2/Enterprise with HCXTools"
date:   2018-08-08 13:49:33 +0000
tags: [WiFi, pentest, redteam, blueteam]
---
![](/blog/assets/WiFi.png)

A new technique to crack WPA PSK (Pre-Shared Key) passwords.

In order to make use of this new attack you need the following tools:
* [hcxdumptool v4.2.0 or higher](https://github.com/ZerBea/hcxdumptool)
* [hcxtools v4.2.0 or higher](https://github.com/ZerBea/hcxtools)
* [hashcat v4.2.0 or higher](https://github.com/hashcat/hashcat)

This attack was discovered accidentally while looking for new ways to attack the new WPA3 security standard. However, WPA3 is not vulnerable to this specific attack! WPA3 will be much harder to attack because of its modern key establishment protocol called “Simultaneous Authentication of Equals” (SAE).

The main difference from existing attacks is that in this attack, capture of a full EAPOL 4-way or half EAPOL 2-way handshake is not required. The new attack is performed on the RSN IE (Robust Security Network Information Element) of a single EAPOL frame.

Currently there is no list of vendors or routers that are vulnerable to this attack. But theoretically it should work against all 802.11i/p/q/r networks with roaming functions enabled (most modern routers).

The main advantages of this attack are as follow:
* No more regular users required — because the attacker directly communicates with the AP (aka “client-less” attack)
* No more waiting for a EAPOL handshake between the regular user and the AP
* No more eventual retransmissions of EAPOL frames (which can lead to uncrackable results)
* No more eventual invalid passwords sent by the regular user
* No more lost EAPOL frames when the regular user or the AP is too far away from the attacker
* No more fixing of nonce and replaycounter values required (resulting in slightly higher speeds)
* No more special output format (pcap, hccapx, etc.) — final data will appear as regular hex encoded string

## Usage
### General WPA/WPA2 Exploit
Dump the packet
```
./hcdumptool -o test.pcapng -i <interface> --enable_status 1
```
Target specific AP:
```
echo 112233445566 > /tmp/filter.txt
./hcdumptool -o test.pcapng -i <interface> --filtermode 2 --filerlist /tmp/filter.txt --enable_status 1
```
### Cracking Enterprise Mode
**Note**: While not required it is recommended to use options -E -I and -U with hcxpcaptool. We can use these files to feed hashcat. They typically produce good results.
* -E retrieve possible passwords from WiFi-traffic (additional, this list will include ESSIDs)
* -I retrieve identities from WiFi-traffic
* -U retrieve usernames from WiFi-traffic
```
$ ./hcxpcaptool -E essidlist -I identitylist -U usernamelist -z test.16800 test.pcapng
```
### Hash Cracking
Extract the hash
```
./hcxpcaptool -z test.16800 test.pcapng
```
Crack with hashcat
```
./hashcat -m 16800 test.16800 -a3 -w 3 '?|?|?|?|?|?|t!'
```

## References
* https://hashcat.net/forum/thread-7717.html
