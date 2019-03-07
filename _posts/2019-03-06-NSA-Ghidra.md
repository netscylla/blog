---
layout: post
title:  "NSA Release Ghidra"
description: "An old script that automated disassembly of shellcode to determine its functionality"
date:   2019-03-06 11:49:33 +0000
tags: [Malware, Incident Response, Pentesting, Red Team]
---
![ghidra logo](/blog/assets/ghidra.png)

## Intro

Today on 6th March 2019 Netscylla woke to a plethora of tweets and emails from collegues across the globe that Ghidra had finally been released!

But what is Ghidra?

A description (wiki page) of Ghidra was leaked as part of the March 7th 2017 Vault-7 via Wikileaks!  It gave us information that the NSA (Nation Security Agency)
had developed an in-house tool for the fast and effective analysis of malware and suspicious binaries
* [https://wikileaks.org/ciav7p1/cms/page_9536070.html](https://wikileaks.org/ciav7p1/cms/page_9536070.html)

During the RSA 2019 conference in San Francisco NSA cybersecurity adviser Rob Joyce called the tool a "contribution to the nation’s cybersecurity community", it will no doubt be used far beyond the United States.

You can't use Ghidra to hack devices; it's instead a reverse-engineering platform used to take "compiled," deployed software and "decompile" it. In other words, it transforms the ones and zeros that computers understand back into a human-readable structure, logic, and set of commands that reveal what the software you churn through it does. Reverse engineering is a crucial process for malware analysts and threat intelligence researchers, because it allows them to work backward from software they discover in the wild—like malware being used to carry out attacks—to understand how it works, what its capabilities are, and who wrote it or where it came from.

## What is Ghidra
Ghidra is a soon to be Open-Sourced project that specialises in the decompilation and analysis of binary executions. Because the platform is written in Java it is cross platform, meaning that it will successfully run on:
 * Windows
 * Linux
 * OSX
 * BSD

There are some small caveats:
 * Java JDK 11+

The power of the functionality behind Ghidra is quite phenominal as it can support multi binary formats including:
 * PE
 * ELF
 * Mach-O
 * Arm binaries
 * Apple IOS
 * Android  
 * & more ....
 
RSA 2019 Slides can be found here, that go into the release in more detail:
 * https://www.rsaconference.com/writable/presentations/file_upload/png-t09-come-get-your-free-nsa-reverse-engineering-tool_.pdf
 
## Where can I get Ghidra
So far only the binary is released (Version 9):
  * [https://ghidra-sre.org/](https://ghidra-sre.org/)

For help and bug fixing, we recommend bookmarking the github page:
  * [https://github.com/NationalSecurityAgency/ghidra](https://github.com/NationalSecurityAgency/ghidra) 
  
## Taking Ghidra for a Test-Drive...
Netscylla wanted to test Ghidra out on some crackme's we've never tried before, for an initial first impression, on how easy and intuitive it is to use the tool. 
MalwareTech (aka Marcus Hutchins) has some on his website [https://www.malwaretech.com/beginner-malware-reversing-challenges](https://www.malwaretech.com/beginner-malware-reversing-challenges) so we thought we would start there...

Ghidra made super fast work of the strings crackme's, it made it so easy that for the interest of others wanting to play we will skip over them here. 

We were very impressed that it enabled us to reverse the strings challenges in high speed - quite literally less than 2-minutes per challenge!!!

It was surprisinly super quick at:
* fingerprinting strings in the binary and memory
* extracting stack-strings; similar to this fireeye post [flare-ida-pro-script-series-automatic-recovery-of-constructed-strings-in-malware(https://www.fireeye.com/blog/threat-research/2014/08/flare-ida-pro-script-series-automatic-recovery-of-constructed-strings-in-malware.html)
* extracting and mining resources
After analysing what it was doing to the decompilation windows - although usually not the best approach or most accurate; but in this case it worked with Marcus' challenges
is that it appeared to interpret the code and deduce values and arguments - meaning we didn't have to break out the calculator to figure out sums and offsets. 

It really did make super light work or Marcus's challenges!

We thought we would semi-walkthrough the shellcode1 challenge in conjunction with GCHQ's tool (CyberChef)[https://gchq.github.io/CyberChef/].

### Shellcode1
We start a new project, import the binary Shellcode1.exe_

We are greeted with a binary details table on the imported binary:
![ghidra import](/blog/assets/ghidra_import.png)

As this is pure shellcode, and there are no obvious imports/exported functions, with some prior knowledge in reversing malware we head straight to **entry!**


![malwaretech shellcode disassembled](/blog/assets/malwaretech_sc1_0dis.png)

What is impressive here, is that the decompilation window on the right is pretty accurate, and enables us to quickly ascertain what this binary is trying to do.

We can see that its reserving two areas on the heap and copying code into each...
 * 0x404040
 * 0x404068

![malwaretech shellcode disassembled 2](/blog/assets/malwaretech_sc1_1dis.png)

Once we have copied these sections of code, we can leave Ghidra and have fun with CyberChef...

By copying the 2nd piece of raw code, and using CyberChef to manipulate the data, we can guess this is not a string but more code:

![malwaretech shellcode disassembled 3](/blog/assets/malwaretech_sc1_more_shellcode.png)

 * [Click here for Cyberchef Recipe](https://gchq.github.io/CyberChef/#recipe=Regular_expression('User%20defined','%5C%5Cw%7B8%7D%5C%5Cs%2B%5C%5Cw%2B%5C%5CS',true,true,true,false,false,false,'List%20matches')Fork('%5C%5Cn','%5C%5Cn',false)Drop_bytes(0,9,false)&input=MDA0MDQwNjggOGIgICAgICAgICAgICAgID8/ICAgICAgICAgOEJoCjAwNDA0MDY5IDNlICAgICAgICAgICAgICA/PyAgICAgICAgIDNFaCAKMDA0MDQwNmEgOGIgICAgICAgICAgICAgID8/ICAgICAgICAgOEJoCjAwNDA0MDZiIDRlICAgICAgICAgICAgICA/PyAgICAgICAgIDRFaCAKMDA0MDQwNmMgMDQgICAgICAgICAgICAgID8/ICAgICAgICAgMDRoCjAwNDA0MDZkIGMwICAgICAgICAgICAgICA/PyAgICAgICAgIEMwaAowMDQwNDA2ZSA0NCAgICAgICAgICAgICAgPz8gICAgICAgICA0NGggIAowMDQwNDA2ZiAwZiAgICAgICAgICAgICAgPz8gICAgICAgICAwRmgKMDA0MDQwNzAgZmYgICAgICAgICAgICAgID8/ICAgICAgICAgRkZoCjAwNDA0MDcxIDA1ICAgICAgICAgICAgICA/PyAgICAgICAgIDA1aAowMDQwNDA3MiBlMiAgICAgICAgICAgICAgPz8gICAgICAgICBFMmgKMDA0MDQwNzMgZjkgICAgICAgICAgICAgID8/ICAgICAgICAgRjloCjAwNDA0MDc0IGMzICAgICAgICAgICAgICA/PyAgICAgICAgIEMzaAo)

We can quickly disassemble the code to find out it is a Bitwise-Rotate-Left Operation:

![malwaretech shellcode disassembled 4](/blog/assets/malwaretech_sc1_ms_disass.png)

 * [Click here for Cyberchef 2nd Recipe](https://gchq.github.io/CyberChef/#recipe=Find_/_Replace(%7B'option':'Regex','string':'%5C%5Cn'%7D,'',true,false,true,false)Disassemble_x86('64','Full%20x86%20architecture',16,0,true,true)&input=OGIKM2UKOGIKNGUKMDQKYzAKNDQKMGYKZmYKMDUKZTIKZjkKYzM)

Analysing the code in Ghidra, we know that the bitwise rotate operation uses the 1st heap code as an argument, so lets use CyberChef again: 

![malwaretech shellcode disassembled 5](/blog/assets/malwaretech_sc1_result.png)

 * [Click here for Cyberchef 3rd Recipe](https://gchq.github.io/CyberChef/#recipe=From_Hex('Auto')Rotate_left(5,false)&input=xxxx)
 
## Conclusion
This is an awesome piece of kit that makes light work of easy trivial crackme's; we found Ghidira easy to use, and intuitive considering we did not bother reading the manual.
With further playing, and assessing its more advanced capabilities I've sure this is a wonderful gift from the NSA, and will quickly rival some of the more commercial tools.

It is probably worth following the tweets and blog posts of other professional reverse engineers, to gain insight to the more advanced features and handy shortcuts that are made
available to the engineer! 
