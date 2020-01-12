---
layout: post
title:  "Linux to Powershell Command Cheatsheet"
description: "Linux to Powershell command cheatsheet"
date:   2019-11-24 14:49:33 +0000
tags: [Blue Team, Red Team, Knowledge sharing, Powershell, cheatsheet]
---

![powershell logo](/blog/assets/PowerShell_5.0.png)

## General
Our comparison cheatsheet of frequently used bash commands and their ps equivalents

|Bash/Linux	|PowerShell|
|------|------|
|ls	|ls|
|mv	|mv|
|cp	|cp|
|pwd |pwd|
|rm	|rm|
|cat	|cat|
|grep	|select-string|
|echo	|echo|
|var=0	|$var=0|
|df	|gdr<br>Get-PSdrive|
|wc |measure-object|
|wc -l|type \[gc \[object\]\]\|measure-object -line\|select lines
|ps	|ps|
|find	|gci|
|diff	|diff|
|kill	|kill|
|time	|measure-command|
|if [condition] then something fi |if (condition) { something } |
|-e file	|Test-Path file|
|for ((i=0; i < 10; i++)) ; do echo $i ; done	|for ($i=0;$i -lt 10; $i++) { echo $i }|

## Networking
Our comparison cheatsheet of frequently used bash networking commands and their ps equivalents

|Bash/Linux	|PowerShell|
|------|------|
|ipconfig	|Sort InterfaceIndex \| FT InterfaceIndex, InterfaceAlias, AddressFamily, IPAddress, PrefixLength -Autosize
|ping	|Test-NetConnection|
|nslookup	|Resolve-DnsName<br>Resolve-Dnsname -Name <name> -Type <a/cname/txt/aaaa>|
|route	|Get-NetRoute <br> Get-NetRoute -Protocol Local -DestinationPrefix 192.168*\|Get-NetAdapter Wi-Fi 
|netstat -an	|Get-NetTCPConnection \| ? State -eq Listen <br> Get-NetTCPConnection \| ? State -eq Listen \| ? LocalAddress -notlike "::*"|
|tracert	|Test-NetConnection www.microsoft.com –TraceRoute<br>Test-NetConnection outlook.com -TraceRoute \| Select -ExpandProperty TraceRoute \| % { Resolve-DnsName $_ -type PTR -ErrorAction SilentlyContinue }
|netstat	|Get-NetTCPConnection \| Group State, RemotePort \| Sort Count \| FT Count, Name –Autosize<br>Get-NetTCPConnection \| ? State -eq Established \| FT –Autosize<br>Get-NetTCPConnection \| ? State -eq Established \| ? RemoteAddress -notlike 127* \| % { $_; Resolve-DnsName $_.RemoteAddress -type PTR -ErrorAction SilentlyContinue }|
|who<br>w|ForEach ($log in (get-eventlog system -source Microsoft-Windows-Winlogon -After (Get-Date).AddDays(-7))) {if($log.instanceid -eq 7001) {$type = "Logon"} Elseif ($log.instanceid -eq 7002){$type="Logoff"} Else {Continue} $res += New-Object PSObject -Property @{Time = $log.TimeWritten; "Event" = $type; User = (New-Object System.Security.Principal.SecurityIdentifier $Log.ReplacementStrings[1]).Translate([System.Security.Principal.NTAccount])}};$res;|

## Passwords
### Generate Random Password
Lastly some commands for generating random passwords:

|Bash/Linux	|PowerShell|
|------|------|
|/dev/urandom tr -dc _A-Z-a-z-0-9 | head -c${1:-32};echo;|
|tr -cd '[:alnum:]' < /dev/urandom | fold -w30 \| head -n1|
|openssl rand –base64 14|Add-Type -AssemblyName System.web<br>[System.Web.Security.Membership]::GeneratePassword(12,3)|
