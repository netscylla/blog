---
layout: post
title:  "Putty vs TeraTerm"
description: "Demonstrating that sometimes putty goes wrong, and why to use another terminal program"
date:   2019-10-01 14:49:33 +0000
tags: [Blue Team, Red Team, Cisco, Hardware, TTY]
---

![](/assets/tele.jpg)

## Putty
Putty is usually the most common go-to program on Windows OS, for SSH or Serial communication 
 * [https://www.chiark.greenend.org.uk/~sgtatham/putty/](https://www.chiark.greenend.org.uk/~sgtatham/putty/)

When you first open putty you typically see the following screenshot:

![Putty](/assets/putty-screen-00.jpg)

You fill in the connection details (Serial, COM, Connection speed). Hopefully, if successful you should 
see a screen similar to the following:

![Putty Success](/assets/putty-screen-01.jpg)

The screenshot is an example of connecting to a Raspberry Pi over the UART interface.

### Putty and the unrecognisable character problem

But sometimes when interacting with other devices or networking equipment you may receive the following:

![Putty Failure](/assets/putty-screen-02.jpg)

This has happened on many hardware hacking assignments, when we've tried to get a UART or JTAG shell.
We have even had this happen once (or twice) when using a serial connection to perform configuration audits
on network routing equipment.

So we keep receiving strange garbage? So what went wrong?

There are many different theories of what has possibly gone wrong?
 * putty not validating line-end characters
 * putty not correctly validating the signal strength on the wires.
 * unfamiliar fonts
 * incorrect UTF-8 formatting

We've never gotten to the bottom of this issue. But our work-around is to use another terminal program. 
We previously used Hyper-Terminal (Old Windows Built-in tool), but this is not available in the latest OS versions; therefore
we have to use an alternative. 

Currently our favourite recommendation is TeraTerm.

## TeraTerm
Our favorite Windows tool for communicating over TTY/COM is TeraTerm
 * [https://ttssh2.osdn.jp/](https://ttssh2.osdn.jp/)

When you start TeraTerm you should see the following window:

![TeraTerm Configuration](/assets/tt-00.PNG)

Fill in the connection details: Serial and select the correct COM X port. Hopefully you should receive a proper shell: 

![TeraTerm Success](/assets/tt-01.PNG)

