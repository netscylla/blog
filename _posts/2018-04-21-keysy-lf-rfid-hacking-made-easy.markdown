---
layout: post
title:  "Keysy LF RFID Hacking Made Easy"
date:   2018-04-21 13:49:33 +0000
tags: [RFID, hardware, pentest, redteam, blueteam]
---
![](/blog/assets/keysy.jpeg)

Those of you that familiar with Netscylla from past escapades and Red-Teaming know that we love RFID technology, and that our own director was once a Proxmark3 developer. Today, in GB we finally received our Keysy devices from TinyLabs from a Kickstarter campaign that started over 2 years ago.

The Keysy is a small battery operated Low Frequency (LF) card cloner that fits neatly on your keychain, and it is also nice, small and convert in the palm of your hand.

## A simple Red-Team device
### Initial Testing
This device is just awesome for Red-Teams (or Black-Teams that conduct physical security testing in GB). Simply click and hold a button to put it in read mode, and you have a maximum of 15 seconds to clone a tag. In our testing, we found most tags are cloned in under 6 seconds. The tag can then be replayed by pushing the same button, or cloned to the provided tag (by pushing the button 5 times).

### Small problem
Further testing revealed that when the Keysy is used to replay the cloned signal, you have to touch the device to the reader, and hold it at a particular angle. Its worth noting here that a number of our RFID readers failed to read a signal from the Keysy on more than one occasion!

### The Solution
Simply clone the captured signal (stored on the Keysy) to the provided tag! Then touch the provided tag to the reader and it works perfectly every time! I admit its not perfect to constantly clone to tag, then tag to the reader. But it means you can quickly tag into/out of places without looking too awkward.

## Tags
The tags we have tested so far:
* AWID
* Indala
* HID26
* Pyramid
* EM4100

Watch this space as we assess more tags in the near future.

