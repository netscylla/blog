---
layout: post
title:  "Installing Sysmon"
description: "a brief post on the installation and setup of Sysinternals Sysmon & associated Tooling"
date:   2020-01-25 13:01:01 +0000
tags: [Blue-Team, SOC, Monitoring, Logging]
---

![Sysmon](/blog/assets/search.png)

## Why use Sysmon
System Monitor (Sysmon) is a Windows system service and device driver that, once installed on a system, 
remains resident across system reboots to monitor and log system activity to the Windows event log. 
It provides detailed information about process creations, network connections, and changes to file creation 
time. By collecting the events it generates using Windows Event Collection or SIEM agents and subsequently 
analysing them, you can identify malicious or anomalous activity and understand how intruders and malware 
operate on your network.

With the increase of phishing attacks, malware and ransomware last year (2019). Netscylla thinks as a proactive approach any business running
Microsoft Windows should implment Sysmon with its advanced logging functionality. Especially with regards to the new DNS event logging. 
DNS logging can be tricky on Windows domains, but Sysmon makes life so much easier with less technical complexities.

## Installation

The download is available from the main Microsoft.com website under the Sysinternal's section:
 * [https://docs.microsoft.com/en-us/sysinternals/downloads/sysmon](Download Sysmon)

Installation is as simple as running the executable with the -i flag with a configuration file:
```
sysmon -i <config.xml>
```
For a headstart on the configuration you can use the one from that is regularly updated from SwiftonSecurity:
 * [https://github.com/SwiftOnSecurity/sysmon-config](https://github.com/SwiftOnSecurity/sysmon-config)

So the install might look something like:
```
sysmon -i sysmonconfig-export.xml
``` 

## Advanced Installation
Many attackers have recently developed automation platforms or tooling that can detect and with the right level of privileges possibly disable Sysmon logging.

Examples of such tooling could be
 * Empire
 * Metasploit
 * Cobalt Strike
 
The Sysinternal's team have implemented some evasion tricks in an attempt to masquerade the Sysmon process and driver from a curious user, or less deep-inspective attack software, 
so we thought we would highlight these available options here.

### Changing the Sysmon Service Name
Changing the service name is as easy as first changing the name of the installation executable
```
rename sysmon.exe custom_.exe
```

### Changing the Sysmon Driver Name
You have two options to change the driver name.

**Warning** you are limited to 8 characters for a filter name.

1. Use the -d flag on install

```
sysmon.exe -c config.xml -d custom_
```

2. Edit the xml configuration, use the Drivername element

```
<xml>
...abbrev
<DriverName>NotReal</DriverName>
...abbrev
```

### Downside of renaming
The downside is that if you use additional Sysmon tooling (from 3rd-parties) you may have to alter the source-code to reflect the name changes of both the service and the driver.  This is why many organisation tend to use the default Sysmon names.

## Attacking Sysmon
A known attack is unloading the Sysmon Driver.  The Sysmon executable still remains open the successfully running. However, without the driver functionality, no further logs would be recorded.

Driver unloading (with the right level of privileges) is as simple as
```
fltmc.exe unload SysmonDrv
```
Therefore, as a small defensive measure, you may want to consider renaming Sysmon and the driver.

## Sysmon in Operation
The following are examples of each event type that Sysmon generates.

### Event ID 1: Process creation
The process creation event provides extended information about a newly created process. The full command line provides context on the process execution. The ProcessGUID field is a unique value for this process across a domain to make event correlation easier. The hash is a full hash of the file with the algorithms in the HashType field.

### Event ID 2: A process changed a file creation time
The change file creation time event is registered when a file creation time is explicitly modified by a process. This event helps tracking the real creation time of a file. Attackers may change the file creation time of a backdoor to make it look like it was installed with the operating system. Note that many processes legitimately change the creation time of a file; it does not necessarily indicate malicious activity.

### Event ID 3: Network connection
The network connection event logs TCP/UDP connections on the machine. It is disabled by default. Each connection is linked to a process through the ProcessId and ProcessGUID fields. The event also contains the source and destination host names IP addresses, port numbers and IPv6 status.

### Event ID 4: Sysmon service state changed
The service state change event reports the state of the Sysmon service (started or stopped).

### Event ID 5: Process terminated
The process terminate event reports when a process terminates. It provides the UtcTime, ProcessGuid and ProcessId of the process.

### Event ID 6: Driver loaded
The driver loaded events provides information about a driver being loaded on the system. The configured hashes are provided as well as signature information. The signature is created asynchronously for performance reasons and indicates if the file was removed after loading.

### Event ID 7: Image loaded
The image loaded event logs when a module is loaded in a specific process. This event is disabled by default and needs to be configured with the –l option. It indicates the process in which the module is loaded, hashes and signature information. The signature is created asynchronously for performance reasons and indicates if the file was removed after loading. This event should be configured carefully, as monitoring all image load events will generate a large number of events.

### Event ID 8: CreateRemoteThread
The CreateRemoteThread event detects when a process creates a thread in another process. This technique is used by malware to inject code and hide in other processes. The event indicates the source and target process. It gives information on the code that will be run in the new thread: StartAddress, StartModule and StartFunction. Note that StartModule and StartFunction fields are inferred, they might be empty if the starting address is outside loaded modules or known exported functions.

### Event ID 9: RawAccessRead
The RawAccessRead event detects when a process conducts reading operations from the drive using the \\.\ denotation. This technique is often used by malware for data exfiltration of files that are locked for reading, as well as to avoid file access auditing tools. The event indicates the source process and target device.

### Event ID 10: ProcessAccess
The process accessed event reports when a process opens another process, an operation that’s often followed by information queries or reading and writing the address space of the target process. This enables detection of hacking tools that read the memory contents of processes like Local Security Authority (Lsass.exe) in order to steal credentials for use in Pass-the-Hash attacks. Enabling it can generate significant amounts of logging if there are diagnostic utilities active that repeatedly open processes to query their state, so it generally should only be done so with filters that remove expected accesses.

### Event ID 11: FileCreate
File create operations are logged when a file is created or overwritten. This event is useful for monitoring autostart locations, like the Startup folder, as well as temporary and download directories, which are common places malware drops during initial infection.

### Event ID 12: RegistryEvent (Object create and delete)
Registry key and value create and delete operations map to this event type, which can be useful for monitoring for changes to Registry autostart locations, or specific malware registry modifications.
Sysmon uses abbreviated versions of Registry root key names, with the following mappings:

|Key name	|Abbreviation|
|-----------|------------|
|HKEY_LOCAL_MACHINE	|HKLM|
|HKEY_USERS	|HKU|
|KEY_LOCAL_MACHINE\System\ControlSet00x	|HKLM\System\CurrentControlSet|
|KEY_LOCAL_MACHINE\Classes	|HKCR|

### Event ID 13: RegistryEvent (Value Set)
This Registry event type identifies Registry value modifications. The event records the value written for Registry values of type DWORD and QWORD.

### Event ID 14: RegistryEvent (Key and Value Rename)
Registry key and value rename operations map to this event type, recording the new name of the key or value that was renamed.

### Event ID 15: FileCreateStreamHash
This event logs when a named file stream is created, and it generates events that log the hash of the contents of the file to which the stream is assigned (the unnamed stream), as well as the contents of the named stream. There are malware variants that drop their executables or configuration settings via browser downloads, and this event is aimed at capturing that based on the browser attaching a Zone.Identifier “mark of the web” stream.

### Event ID 17: PipeEvent (Pipe Created)
This event generates when a named pipe is created. Malware often uses named pipes for interprocess communication.

### Event ID 18: PipeEvent (Pipe Connected)
This event logs when a named pipe connection is made between a client and a server.

### Event ID 19: WmiEvent (WmiEventFilter activity detected)
When a WMI event filter is registered, which is a method used by malware to execute, this event logs the WMI namespace, filter name and filter expression.

### Event ID 20: WmiEvent (WmiEventConsumer activity detected)
This event logs the registration of WMI consumers, recording the consumer name, log, and destination.

### Event ID 21: WmiEvent (WmiEventConsumerToFilter activity detected)
When a consumer binds to a filter, this event logs the consumer name and filter path.

### Event ID 22: DNSEvent (DNS query)
This event generates when a process executes a DNS query, whether the result is successful or fails, cached or not. The telemetry for this event was added for Windows 8.1 so it is not available on Windows 7 and earlier.

Example:

![Sysmon dns alert](/blog/assets/sysmon_dns.png)

## References
 * [https://docs.microsoft.com/en-us/sysinternals/downloads/sysmon](https://docs.microsoft.com/en-us/sysinternals/downloads/sysmon)
 * [https://github.com/SwiftOnSecurity/sysmon-config](https://github.com/SwiftOnSecurity/sysmon-config)
 * [https://cqureacademy.com/blog/server-monitoring/sysmon](https://cqureacademy.com/blog/server-monitoring/sysmon)
 * [https://github.com/mattifestation/PSSysmonTools](https://github.com/mattifestation/PSSysmonTools)
 * [https://github.com/MHaggis/sysmon-dfir](https://github.com/MHaggis/sysmon-dfir)
 * [https://www.sans.org/cyber-security-summit/archives/file/summit-archive-1554993664.pdf](https://www.sans.org/cyber-security-summit/archives/file/summit-archive-1554993664.pdf)
