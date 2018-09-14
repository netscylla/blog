---
layout: post
title:  "Hacking Digital Signs for Fun (and no profit!)"
date:   2018-05-12 13:49:33 +0000
tags: [hardware, IoT, pentest, redteam, blueteam]
---

![](/blog/assets/sign_hack.jpeg)

You can usually find digital information signs in hotels, conference centres, sport stadiums, shopping centres/malls and other places where people might collectively meet.

With the increasing popularity of IOT, more and more of these signs are popping up all over the place. But hacking these signs is nothing new; considering past events (see below). Recently we have seen a flurry of new posts on social media (Twitter/Facebook/Linkedin), and we have been asked by our friends and colleagues on how such a hack is possible? Having been in the industry for close to 20 years, we have encountered digital signs and hacked (with explicit client permission) them successfully.

## A history of public exploits
* April 2018 http://www.abc.net.au/news/2018-04-06/porn-site-pornhub-displayed-on-perth-yagan-square-touchscreen/9624428
* May 2017 http://www.bbc.co.uk/news/technology-40092037

## Attacking Digital Signs
There are often two types of digital signs we have encountered in the past; and here is our rough methodology of cracking them:

### Type 1 — Embedded Windows
* Open WiFi AccessPoint / or crackable weak key?
* DHCP may not be enabled, so you need an understanding of IP networking
* RDP/VNC service with default/weak credentials
* Simply load applications from the embedded Windows desktop (e.g. Notepad/Word/IE/VLC)

### Type 2 — IP-based Printers
* Staff often forget to configure these signs and may leave them in a default state:

### Open WiFi AccessPoint / or crackable weak key?
* DHCP may not be enabled, so you need an understanding of IP networking
* HTTP Management section default credentials
* A driver should usually be available to download from the HTTP management interface.
* The driver for Windows acts as an IP-based/IOT Printer
* Simply create you document in your favourite wordprocessing/multimedia program.
* Click ‘Print’ to send your document to the digital sign for display
* Other/advanced features
* You may have access to special features/configurations:
  - speed
  - transition
  - schedule
  - no of occurrences.

And possibly many more options?
