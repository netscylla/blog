---
layout: post
title:  "Zigbee & Killerbee"
date:   2018-02-26 13:49:33 +0000
tags: [hardware, pentest, redteam, blueteam, Zigbee, WiFi]
---
![](/blog/assets/xbee.jpeg)

Now it has been four years since I last performed any real Zigbee penetration testing, at the time I was trying to compromise a building management system through a fridge. But over the weekend I decided to pull out the old kit and have a scan for any hidden Zigbee devices in our office.

At the time Zigbee penetration testing was relatively new, and tooling was not as developed or blogged about as it is today. Killerbee was available and this helped with some initial enumeration of Zigbee enabled devices, but scapy was used to generate packets and communicate with the network.

![](/blog/assets/atmel_zigbee.jpeg)

# Flashing the RZ Raven
Now the RZ Raven is shipped with firmware that has no monitor mode! So you have to flash the modified firmware in the killerbee subdirectory. This device had the first firmware modification 001, but over the last four years this has been improved by the riverloopsec team and Scytmo. Firmware 003 is now the latest version. So we dusted off the AVR dragon to upgrade the device. The JTAG ports you see in the image above are a tiny 50mil 10 pin, so you have to be creative to get some wires into these through-holes without touching; I just pivoted off a breadboard.

![](/blog/assets/avr_dragon.png)

The AVR Dragon is a cheap board aimed at students for flashing Atmel chips and microcontrollers. Its easily available on eBay and electronic stores between £30–50 depending if you buying used or new.

The wires between the AVR dragon JTAG and the RZ Raven are in a slightly different order so take care when wiring them up together. Looking at other blogs online, they have not mentioned the wiring numbering so this will be depicted below:
```
JTAG Pin Out AVR Dragon
2 4 6 8 10
1 3 5 7 9 
JTAG Pin Out RZ Raven
9  7 5 3 1
10 8 6 4 2
```
# AVRdude
To flash the firmware I used the avrdude package that is easily available from the ubuntu apt-get repositories:
```
$ sudo apt install avrdude
$ cd killerbee/firmware
$ sudo avrdude -p usb1287 -c dragon_jtag -P usb -U flash:w:kb-rzusbstick-003.hex
```
**Note**: After a successful flash the RZ Raven LED should flash Orange!

# ZStumbler
I could then use zstumbler to scan my local airwaves, and discover two zigbee devices hidden within the office. They are probably related to the lighting and heating management systems?
```
$ sudo zbstumbler
zbstumbler: Transmitting and receiving on interface '004:007'
New Network: PANID 0x8304  Source 0x0001
        Ext PANID: 00:00:00:00:00:00:00:00
        Stack Profile: ZigBee Standard
        Stack Version: ZigBee 2006/2007
        Channel: 11
New Network: PANID 0x8304  Source 0x0000
        Ext PANID: 00:00:00:00:00:00:00:00
        Stack Profile: ZigBee Standard
        Stack Version: ZigBee 2006/2007
        Channel: 11
New Network: PANID 0x4EC5  Source 0x0000
        Ext PANID: 39:32:97:90:d2:38:df:B9
        Stack Profile: ZigBee Enterprise
        Stack Version: ZigBee 2006/2007
        Channel: 15
... abbreviated...
```
And the fun of packet capture in wireshark, protcol dissection, and packet fun with scapy begins again…

# Killerbee code
* [https://github.com/riverloopsec/killerbee](https://github.com/riverloopsec/killerbee)

# Other code available
* https://github.com/inguardians/zigbee_tools
* https://github.com/Cognosec/SecBee

# Other slides/presentations
* [Josh Wrights Toorcon 2011 Slides (original inspiration)](http://www.willhackforsushi.com/presentations/toorcon11-wright.pdf)
* [Defcon23 Team 360UnicornTeams ZigBee Presentation](https://media.defcon.org/DEF%20CON%2023/DEF%20CON%2023%20presentations/DEFCON-23-Li-Jun-Yang-Qing-I-AM-A-NEWBIE-YET-I-CAN-HACK-ZIGBEE.pdf)
