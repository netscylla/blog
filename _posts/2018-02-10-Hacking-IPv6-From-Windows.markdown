---
layout: post
title:  "Hacking IPv6 from Windows"
date:   2018-02-10 13:49:33 +0000
tags: [networking, pentest, redteam, blueteam]
---
![](/blog/assets/ipv6.jpeg)

The last article drew some constructive criticism that some penetration testers prefer (or are stuck with) using Windows, and that we had intentionally left the Windows Operating System out. Hopefully, this post will alleviate those concerns…

So here we go, IPv6 in Windows 10 should be supported straight out of the box. But if you want to double check, go to:
* Control Panel
* Network Connections
* Change adapter options
* Locate your interface, right-click and properties
* Look for the IPv6 module, ensure there is a tick in the box

![](/blog/assets/win_ipv6.jpeg)

# IPv6 Addressing
More on IPv6 addressing, and various formats can be found here: https://en.wikipedia.org/wiki/IPv6_address

## Get your IPv6 address
<pre>ipconfig
</pre>
Below is the basic usage of the ipconfig command to determine your IPv6 address:
<pre>
$ ipconfig
</pre>
Example:
```
$ ipconfig
Ethernet adapter vEthernet (External):
Connection-specific DNS Suffix  . : test.local
   Link-local IPv6 Address . . . . . : fe80::75f8:76d4:930e:19d5%26
   IPv4 Address. . . . . . . . . . . : 10.2.66.190
   Subnet Mask . . . . . . . . . . . : 255.255.224.0
   Default Gateway . . . . . . . . . : 10.2.95.254
```
# netsh
To get the most out of Windows and IPV6, you are going to have to learn how to use the netshell command or netsh.
```
Show your interfaces:

$ netsh int ipv6 show int
Idx     Met         MTU          State                Name
---  ----------  ----------  ------------  ---------------------------
 10           5        1500  disconnected  Ethernet
  1          75  4294967295  connected     Loopback Pseudo-Interface 
 26          25        1500  connected     vEthernet (External)
```
**Warning**: You may have many interfaces if you have Hypervisors and Docker installed. The list above has been trimmed to make it easier to read. But the task ahead of you to identify the correct interface and IP address may be harder?

# Show your IPv6 addresses:
<pre>netsh int ipv6 show address
</pre>
Example of a static configured host:
```
netsh int ipv6 show address
Interface 26: vEthernet (External)
Addr Type  DAD State   Valid Life Pref. Life Address
---------  ----------- ---------- ---------- ------------------------
Other      Preferred     infinite   infinite fe80::75f8:76d4:930e:19d5%26
```
# Using route
## Display IPv6 route
Display your Windows IPv6 routing table:
<pre>$ route print -6
</pre>
Example. Here you see different IPv6 routes for different interfaces on Windows.
```
$ route print -6
====================================================================
Interface List
 10...f0 de f1 1a 14 f9 ......Intel(R) 82577LM Gigabit Network    
 26...00 23 14 da 5e 90 ......Hyper-V Virtual Ethernet Adapter
  1...........................Software Loopback Interface 1
====================================================================
IPv6 Route Table
====================================================================
Active Routes:
 If Metric Network Destination      Gateway
  1    331 ::1/128                  On-link
 26    281 fe80::/64                On-link
 26    281 fe80::75f8:76d4:930e:19d5/128
                                    On-link
 26    281 ff00::/8                 On-link
====================================================================
Persistent Routes:
  None
```
Alternatively the same command with netsh
<pre>
$ netsh interface ipv6 show route
Publish  Type      Met  Prefix                    Idx  Gateway/Interface Name
-------  --------  ---  ------------------------  ---  ------------------------
No       System    256  ::1/128                     1  Loopback Pseudo-Interface 1
No       System    256  fe80::/64                  10  Ethernet
No       System    256  fe80::/64                  26  vEthernet (External)
</pre>

# IPv6 Tools

## IPv6 Ping
<pre>
ping -6 [ipv6address]
ping -6 [link-local-ipv6address]
</pre>
Example:
<pre>
$ ping -6 ::1 
Pinging ::1 with 32 bytes of data:
Reply from ::1: time<1ms
Reply from ::1: time<1ms
Reply from ::1: time<1ms
Reply from ::1: time<1ms
Ping statistics for ::1:
    Packets: Sent = 4, Received = 4, Lost = 0 (0% loss),
Approximate round trip times in milli-seconds:
    Minimum = 0ms, Maximum = 0ms, Average = 0ms
</pre>

## IPv6 Traceroute
<pre>
tracert -6 [ipv6address]
tracert -6 [link-local-ipv6address]
</pre>
Example:
<pre>
tracert -6 ::1
Tracing route to Andys-Laptop [::1]
over a maximum of 30 hops:
1    <1 ms    <1 ms    <1 ms  test-laptop [::1]
Trace complete.
</pre>
# Neighbour Discovery
With the following command you can display the learnt or configured IPv6 neighbours:
<pre>
$ netsh int ipv6 show neigh
</pre>
The following example shows neighbors on interface 26, which is a reachable router
```
$ netsh int ipv6 show neigh 26
Internet Address                              Physical Address   Type
--------------------------------------------  -----------------  -----------
fe80::f841:cd78:7c8a:e20d                     00-00-00-00-00-00  Unreachable
ff02::1                                       33-33-00-00-00-01  Permanent
ff02::2                                       33-33-00-00-00-02  Permanent
ff02::16                                      33-33-00-00-00-16  Permanent
ff02::fb                                      33-33-00-00-00-fb  Permanent
ff02::1:2                                     33-33-00-01-00-02  Permanent
ff02::1:3                                     33-33-00-01-00-03  Permanent
ff02::1:ff01:228                              33-33-ff-01-02-28  Permanent
ff02::1:ff0e:19d5                             33-33-ff-0e-19-d5  Permanent
ff02::1:ff14:322c                             33-33-ff-14-32-2c  Permanent
ff02::1:ff7e:ff50                             33-33-ff-7e-ff-50  Permanent
ff02::1:ff8a:e20d                             33-33-ff-8a-e2-0d  Permanent
ff02::1:ffb1:992d                             33-33-ff-b1-99-2d  Permanent
ff05::c                                       33-33-00-00-00-0c  Permanent
```
## Adding Manual Routes/Neighbors
### Windows Auto-Magic
Windows is an odd beast? when it comes to routing as this appears to be handled automatically. As soon as you discover a link-local host, simply try to ping it, it takes a few packets but windows should automatically add the IPv6 address to the list of known neighbours
```
>ping -6  fe80::20c:29ff:fee2:fdb1
Pinging fe80::20c:29ff:fee2:fdb1 with 32 bytes of data:
Destination host unreachable.
Destination host unreachable.
Destination host unreachable.
Destination host unreachable.
Ping statistics for fe80::20c:29ff:fee2:fdb1:
    Packets: Sent = 4, Received = 0, Lost = 4 (100% loss),
>ping -6  fe80::20c:29ff:fee2:fdb1
Pinging fe80::20c:29ff:fee2:fdb1 with 32 bytes of data:
Destination host unreachable.
Reply from fe80::20c:29ff:fee2:fdb1: time=37ms
Reply from fe80::20c:29ff:fee2:fdb1: time=3ms
Reply from fe80::20c:29ff:fee2:fdb1: time=4ms
Ping statistics for fe80::20c:29ff:fee2:fdb1:
    Packets: Sent = 4, Received = 3, Lost = 1 (25% loss),
Approximate round trip times in milli-seconds:
    Minimum = 3ms, Maximum = 37ms, Average = 14ms
```
### Windows Manually Adding Routes
<pre>
>netsh int ipv6 del route [prefix] [interface_string] [next_hop]
</pre>
Example
<pre>
>netsh int ipv6 add route fe80::/16 “vEthernet (External)” fe80::20c:29ff:fe25:2403
Ok.
</pre>
### Windows Manually Deleting Routes
<pre>
>netsh int ipv6 del route [prefix] [interface_string] [next_hop] 
</pre>
Example:
<pre>
>netsh int ipv6 del route fe80::/16 “vEthernet (External)” fe80::20c:29ff:fe25:2403
Ok.
</pre>
# Other IPv6 Tools
Since Windows is more GUI driven on the User experience level, we do not have the luxury of many and different default command line tools . For viewing web-pages we prefer to use either Chrome/Firefox/Edge.

Just remember to encapsulate the address in square brackets ‘[ ]’

For example Google: https://[2a00:1450:4009:808::200e]/

# Other tools
Using 3rd party tools (or Chocolately package manager) you can install familiar tools for accessing the usual services over IPv6.

## Nmap and putty
Nmap:

![](/blog/assets/nmap_ipv6.jpeg)

Putty:

![](/blog/assets/putty_ipv6.jpeg)

# Conclusion
Hopefully, this was enough of an introduction for the more Windows orientated penetration testers to progress with and have a play? Not only to prepare for the future, but also for self development.
