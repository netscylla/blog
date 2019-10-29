---
layout: post
title:  "Jupyter Notebooks for Incident Response"
description: "Demonstrating the advantages of Jupyter notebooks for reporting, collaboration and code-sharing in Incident Response"
date:   2019-10-28 14:49:33 +0000
tags: [Blue Team, Red Team, Knowledge sharing, Reporting, Python, Incident Response]
---

![jupyter logo](/assets/Jupyter.png)

## Background

Following on from our previous post, of utilising Jupyter notebooks for reporting and collaboration in penetration testing.
In this blog post, we have decided to show how powerful Jupyter can be for the Blue Team (Incident Responders and security analysts).

Our previous post can be found here:
 * [Jupyter Notebooks for Reporting & Collaboration](https://www.netscylla.com/blog/2019/10/15/Jupyter-Notebooks-for-Reporting.html)

## Using Jupyter in Incident Response

### John Lambert saves us some time and effort

When we started to build the notebooks for this blog post, we realised that the hard-work hard already been done.

With thanks to John Lambert (Distinguished Engineer, Microsoft Threat Intelligence Center) [@johnlatwc](https://twitter.com/JohnLaTwC)

John's Jupyter notebooks can be found publicly available on his personal Github account: 
 * [https://github.com/JohnLaTwC/Shared/notebooks](https://github.com/JohnLaTwC/Shared/tree/master/notebooks)

In this post we will look at two of his sample notebooks:
 * Malware Decode Demo
 * Decode Powershell & Shellcode Demo
 
### Malware Decode Demo

John's first Jupyter demo involves the decoding of a malicious macro which contains a base64 encoded payload.

John demonstrates that you can easily use python and the ***requests*** module to fetch a payload stored within 
a gist on the internet.
 * https://gist[.]githubusercontent[.]com/JohnLaTwC/230485a13c49501279e023ad4b9510e2/raw/6dccac89d73030cc4c69c2edbfb9a5557482c76c/  

![Malware Decode Demo](/assets/jupyter-malware-demo-1.png)

John's notebook then extracts the relevant base64 encoded strings, concatenates them together and then finally base64 decodes them
to reveal a decryption dropper:

```
...
static byte[] DrainPipe(Stream stream){
    byte[] buffer = new byte[2048];
    byte[] iv = new byte[16] { 0x6b,0x18,0xb7,0x15,0x43,0xab,0xc3,0x30,0xe9,0x6d,0xa5,0xea,0x57,0x35,0xc6,0xf1 };
    byte[] key = new byte[16] { 0x7a,0x79,0xa7,0xe2,0x3a,0x5c,0x9c,0xe4,0x2a,0x13,0x8,0xe8,0xba,0xc7,0x65,0xae };

    Aes encryptor = Aes.Create();
    encryptor.Mode = CipherMode.CBC;
              
    encryptor.Padding = PaddingMode.PKCS7;
        encryptor.Key = key;
        encryptor.IV = iv;
        ICryptoTransform decryptor = encryptor.CreateDecryptor();
...
``` 
So with a little bit of python, we've decoded the payload, stored the notebook for future reference so that other analysts can
use it in the near future with similar suspicious samples.  

Now all thats required is to write up the formal report.  

### Malware PowerShell & Shellcode Analysis  

The next example goes a little deeper...

![Malware Decode Demo](/assets/jupyter-malware-demo-2.png)  

The first cell uses the ***PIL*** library to convert our shellcode into a safe easy to copy picture.

The second cell, should be familir from the previous example, that the suspect code is fetched from an online gist using
the python ***requests*** library. 

The result is a picture that can be copied and pasted into any formal reporting documentation:

![base64 as pic](/assets/jupyter-malware-demo-3.png)   

John then uses the base64 decoding routine and above code to render another safe image of the actual shellcode content in hexadecimal form:
           
![shellcode as pic](/assets/jupyter-malware-demo-4.png)    

From experience, Netscylla can straight away tell this is a Metasploit preloader shell, likely a staged loader for meterpreter over HTTP(S). 
Netscylla in the past has released our shellcode decoding bash script. But John has kindly included a python program and python scripts within
the notebook that will now do all the clever work.

#### Technique #1: dump strings from it

The python script in cell-82 dumps all the strings from the hexadecimal like so, which reveals a URL, and a random path:
```
hwiniThLw
SSSSSh
/gSM74TQSA0uQzpHPyzb8pA3p-2Ym3
SSSWSVh
SSSSVh-
epelix-63870.portmap.io
```      
Therefore we can deduce that the payload will talk to a service at: ***epelix-63870.portmap.io/gSM74TQSA0uQzpHPyzb8pA3p-2Ym3*** the path resembles
a typical Metasploit random string.

#### Technique #2: disassemble the shellcode

This section is more complicated and John has utilise freely available code from [Vivisect](https://github.com/vivisect/vivisect)

A full disassembly dump is present in cell-84, with the inclusion of some handy comments. These comments have been automatically inserted by the
python script thanks to John's inclusion of ***apihashes.db***.  Its essentially the same technique Netscylla uses, matching the hash signatures 
to common Windows API calls.  Knowing these hash calls in plain english, permits analysts to quickly deduce the nature of the suspicious shellcode.

In cell-85 John has created two lamda functions to extract all these comments, so we can easily see them in cell-86, without analysing the full shelldump.
```
['0x00000099 ffd5             call ebp --> APICALL kernel32.dll!LoadLibraryA',
 '0x000000a7 ffd5             call ebp --> APICALL wininet.dll!InternetOpenA',
 '0x000000de ffd5             call ebp --> APICALL wininet.dll!InternetConnectA',
 '0x000000f3 ffd5             call ebp --> APICALL wininet.dll!HttpOpenRequestA',
 '0x0000010b ffd5             call ebp --> APICALL wininet.dll!InternetSetOptionA',
 '0x00000117 ffd5             call ebp --> APICALL wininet.dll!HttpSendRequestA',
 '0x00000127 ffd5             call ebp --> APICALL kernel32.dll!Sleep',
 '0x00000131 ffd5             call ebp --> APICALL kernel32.dll!ExitProcess',
 '0x00000145 ffd5             call ebp --> APICALL kernel32.dll!VirtualAlloc',
 '0x00000159 ffd5             call ebp --> APICALL wininet.dll!InternetReadFile']
 ```
 From the API calls above, we can clearly see the code, opens an Internet URL, sends some data, waits and then expects to download a file.
 
 John then proceeds to explain how apihashes work until we reach the next section.
 
#### Extracting Network indicators from code
 
This example (cell-86) shows how we an automatically extract IOC's such as network addresses(IP) and port numbers:
```
['0x00000000 68c0a88382       push 0x8283a8c0',
'0x00000005 680200115c       push 0x5c110002--> NETWORK IP 192.168.131.130:4444',
...
```
E.g. NETWORK IP 192.168.131.130:4444

### Further Examples

John has continued to provide additional useful examples and walkthroughs (various encoder detections, communication methods and shellcode formats).
The fact that John has freely shared this notebook, means incident responders around the globe and lift-and-shift (or copy n paste) his clever examples.

### Give John a Thank-you & Follow him on Twitter

 * [@johnlatwc](https://twitter.com/JohnLaTwC)

John is a great contributor to the community, he always has interesting tweets and insights on malware, and reversing techniques. He also has additional examples of Jupyter notebooks that can extract data from powerpoint files, to quickly construct graphs and charts.

John if your reading this - you get a big thank you from the staff at Netscylla, and you've also taken all the hard work out of this blog post!

## References
* [@johnlatwc](https://twitter.com/JohnLaTwC)
* https://techcommunity.microsoft.com/t5/Azure-Sentinel/Security-Investigation-with-Azure-Sentinel-and-Jupyter-Notebooks/ba-p/432921