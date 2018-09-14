---
layout: post
title:  "Hacking IPv6 a Primer"
date:   2018-02-05 13:49:33 +0000
tags: [networking, pentest, redteam, blueteam]
---
![](/assets/ipv6.jpeg)

Today, I had a junior network administrator enquire about IPv6 connectivity; They had heard of IPv6 and had seen IPv6 addresses on some Operating System interfaces, but had never really interacted with IPv6. This was due to the business network (at this point of time) not supporting it. So I gave him some notes to try at home for him to practise on his home-lab.

This got me thinking… I should share some of my simple IPv6 notes with the world! Below is a list of commands and examples to get anyone started with IPv6 on Linux or OSX.

Linux — Load the IPv6 module
Some Linux distributions will autoload and pre-configure IPv6, if you dont see an inet6 address under ifconfig or ip you may have to load the ipv6 kernel module (as root or sudo):
<pre># modprobe ipv6
</pre>
# Localhost
check your /etc/hosts file, localhost in IPv6 is denoted as [::1]
<pre>::1          localhost</pre>
more on IPv6 addressing, and various formats can be found here: [https://en.wikipedia.org/wiki/IPv6_address](https://en.wikipedia.org/wiki/IPv6_address)
<pre>/sbin/ip</pre>
Below is the basic usage of the ip command to determine your IPv6 address:

<pre>
$ /sbin/ip -6 addr show dev [interface]
</pre>

Example of a static configured host:
```
$ /sbin/ip -6 addr show dev eth0
2: eth0: <BROADCAST,MULTICAST,UP> mtu 1500 qdisc pfifo_ fast qlen 100
inet6 fe80::210:a4ff:fee3:9566/10 scope link
inet6 2001:0db8:0:f101::1/64 scope global
inet6 fec0:0:0:f101::1/64 scope site
```
Example for a host which is auto-configured:

Here you see some auto-magically configured IPv6 addresses and their lifetime.
<pre>
$ /sbin/ip -6 addr show dev eth0 
3: eth0: <BROADCAST,MULTICAST,PROMISC,UP> mtu 1500 qdisc pfifo_fast qlen
 100 
inet6 2002:d950:f5f8:f101:2e0:18ff:fe90:9205/64 scope global dynamic 
valid_lft 16sec preferred_lft 6sec 
inet6 3ffe:400:100:f101:2e0:18ff:fe90:9205/64 scope global dynamic 
valid_lft 2591997sec preferred_lft 604797sec inet6 fe80::2e0:18ff:fe90:9205/10 scope link
</pre>
# Manually Adding IPv6 Addresses
Using /sbin/ip to manually configure an IPv6 address:

<pre>$ /sbin/ip -6 addr add [ipv6address]/[prefixlength] dev [interface]</pre>
Example:

<pre>$ /sbin/ip -6 addr add 2001:0db8:0:f101::1/64 dev eth0</pre>

# Manually Removing IPv6 Addresses
Using /sbin/ip to manually remove a IPv6 address:

<pre>$ /sbin/ip -6 addr del [ipv6address]/[prefixlength] dev [interface]</pre>
Example:
<pre>
$ /sbin/ip -6 addr del 2001:0db8:0:f101::1/64 dev eth0
</pre>

# IPv6 Ping
<pre>
ping6 [ipv6address]
ping6 [-I [device]] [link-local-ipv6address]
$ ping6 -c 1 ::1 
PING ::1(::1) from ::1 : 56 data bytes 
64 bytes from ::1: icmp_seq=0 hops=64 time=292 usec
--- ::1 ping statistics --- 
1 packets transmitted, 1 packets received, 0% packet loss 
round-trip min/avg/max/mdev = 0.292/0.292/0.292/0.000 ms
</pre>
# Ping6 Multicast
<pre>
# ping6 -I eth0 ff02::1
PING ff02::1(ff02::1) from fe80:::2ab:cdff:feef:0123 eth0: 56 data bytes
64 bytes from ::1: icmp_seq=1 ttl=64 time=0.104 ms
64 bytes from fe80::212:34ff:fe12:3450: icmp_seq=1 ttl=64 time=0.549 ms (DUP!)
</pre>
# Traceroute6
**Note**: unlike some modern versions of IPv4 traceroute, which can use ICMPv4 echo-request packets as well as UDP packets (default), current IPv6-traceroute is only able to send UDP packets. As you perhaps already know, ICMP echo-request packets are more accepted by firewalls or ACLs on routers inbetween than UDP packets.
<pre>
$ traceroute6 www.6bone.net 
traceroute to 6bone.net (3ffe:b00:c18:1::10) from 2001:0db8:0000:f101::2, 30
 hops max, 16 byte packets 
 1 localipv6gateway (2001:0db8:0000:f101::1) 1.354 ms 1.566 ms 0.407 ms 
 2 swi6T1-T0.ipv6.switch.ch (3ffe:2000:0:400::1) 90.431 ms 91.956 ms 92.377 ms 
 3 3ffe:2000:0:1::132 (3ffe:2000:0:1::132) 118.945 ms 107.982 ms 114.557 ms 
 4 3ffe:c00:8023:2b::2 (3ffe:c00:8023:2b::2) 968.468 ms 993.392 ms 973.441 ms 
 5 3ffe:2e00:e:c::3 (3ffe:2e00:e:c::3) 507.784 ms 505.549 ms 508.928 ms 
 6 www.6bone.net (3ffe:b00:c18:1::10) 1265.85 ms * 1304.74 ms
</pre>
# Tracepath6
It’s a program like traceroute6 and traces the path to a given destination discovering the MTU along this path
<pre>
$ tracepath6 www.6bone.net 
 1?: [LOCALHOST] pmtu 1480 
 1: 3ffe:401::2c0:33ff:fe02:14 150.705ms 
 2: 3ffe:b00:c18::5 267.864ms 
 3: 3ffe:b00:c18::5 asymm 2 266.145ms pmtu 1280 
 3: 3ffe:3900:5::2 asymm 4 346.632ms 
 4: 3ffe:28ff:ffff:4::3 asymm 5 365.965ms 
 5: 3ffe:1cff:0:ee::2 asymm 4 534.704ms 
 6: 3ffe:3800::1:1 asymm 4 578.126ms !N 
Resume: pmtu 1280
</pre>
# IPv6 Tcpdump
tcpdump uses expressions for filtering packets to minimize the noise:
* icmp6: filters native ICMPv6 traffic
* ip6: filters native IPv6 traffic (including ICMPv6)
* proto ipv6: filters tunneled IPv6-in-IPv4 traffic
* not port ssh: to suppress displaying SSH packets for running tcpdump in a remote SSH session
Also some command line options are very useful to catch and print more information in a packet, mostly interesting for digging into ICMPv6 packets:
* “-s 512”: increase the snap length during capturing of a packet to 512 bytes
* “-vv”: really verbose output
* “-n”: don’t resolve addresses to names, useful if reverse DNS resolving isn’t working proper

IPv6 ping to 2001:0db8:100:f101::1 native over a local link
<pre>
$ tcpdump -t -n -i eth0 -s 512 -vv ip6 or proto ipv6 
tcpdump: listening on eth0 
2001:0db8:100:f101:2e0:18ff:fe90:9205 > 2001:0db8:100:f101::1: icmp6: echo
 request (len 64, hlim 64) 
2001:0db8:100:f101::1 > 2001:0db8:100:f101:2e0:18ff:fe90:9205: icmp6: echo
 reply (len 64, hlim 64)
IPv6 ping to 2001:0db8:100::1 routed through an IPv6-in-IPv4-tunnel
1.2.3.4 and 5.6.7.8 are tunnel endpoints (all addresses are examples)
</pre>
<pre>
$ tcpdump -t -n -i ppp0 -s 512 -vv ip6 or proto ipv6 
tcpdump: listening on ppp0 
1.2.3.4 > 5.6.7.8: 2002:ffff:f5f8::1 > 2001:0db8:100::1: icmp6: echo request (len 64, hlim 64) (DF) (ttl 64, id 0, len 124) 
5.6.7.8 > 1.2.3.4: 2001:0db8:100::1 > 2002:ffff:f5f8::1: icmp6: echo reply (len 64, hlim 61) (ttl 23, id 29887, len 124) 
1.2.3.4 > 5.6.7.8: 2002:ffff:f5f8::1 > 2001:0db8:100::1: icmp6: echo request (len 64, hlim 64) (DF) (ttl 64, id 0, len 124) 
5.6.7.8 > 1.2.3.4: 2001:0db8:100::1 > 2002:ffff:f5f8::1: icmp6: echo reply (len 64, hlim 61) (ttl 23, id 29919, len 124)
</pre>
# Checking DNS for resolving IPv6 addresses
Because of security updates in the last years every Domain Name System (DNS) server should run newer software which already understands the (intermediate) IPv6 address-type AAAA (the newer one named A6 isn’t still common at the moment because only supported using BIND9 and newer and also the non-existent support of root domain IP6.ARPA). A simple test whether the used system can resolve IPv6 addresses is
<pre>
$ host -t AAAA www.join.uni-muenster.de
</pre>
and should show something like following:
<pre>
www.join.uni-muenster.de. is an alias for tolot.join.uni-muenster.de. 
tolot.join.uni-muenster.de. has AAAA address 2001:638:500:101:2e0:81ff:fe24:37c6
</pre>
# Some IPv6 Ready Tools
## IPv6-ready Telnet clients
IPv6-ready telnet clients are available. A simple test can be done with
<pre>
$ telnet 3ffe:400:100::1 80
Trying 3ffe:400:100::1... 
Connected to 3ffe:400:100::1. 
Escape character is '^]'. 
HEAD / HTTP/1.0
HTTP/1.1 200 OK 
Date: Sun, 16 Dec 2001 16:07:21 
GMT Server: Apache/2.0.28 (Unix) 
Last-Modified: Wed, 01 Aug 2001 21:34:42 GMT 
ETag: "3f02-a4d-b1b3e080" 
Accept-Ranges: bytes 
Content-Length: 2637 
Connection: close 
Content-Type: text/html; charset=ISO-8859-1
Connection closed by foreign host.
</pre>
## IPv6-ready SSH
<pre>
$ ssh -6 ::1 
user@::1's password: ****** 
[user@ipv6host user]$
</pre>
# IPv6-ready Curl
A set of simple example of grabbing the contents of a webpage:
<pre>
$ curl -g [fe80::ba27:ebff:fe60:89af]
$ curl -g [::1]:8080
$ curl -g http://[::1]:8080/
</pre>
# Why -g? Well from the manpage (man curl)
<pre>
-g, — globoff
This option switches off the “URL globbing parser”. When you set
this option, you can specify URLs that contain the letters {}[]
without having them being interpreted by curl itself. Note that
these letters are not normal legal URL contents but they should
be encoded according to the URI standard.
</pre>
In english this permits us to use ‘[‘,’]’ and ‘:’ in the URL without evaluating or confusing part of the address as a custom port value.

# Using route
## Display IPv6 route
### Display your Linux IPv6 routing table:
<pre>
$ /sbin/route -A inet6
</pre>
Example (output is filtered for interface eth0). Here you see different IPv6 routes for different addresses on a single interface.
<pre>
$ /sbin/route -A inet6 |grep -w "eth0"
2001:0db8:0:f101 ::/64 :: UA  256 0 0 eth0 <- Interface route for global address
fe80::/10        ::       UA  256 0 0 eth0 <- Interface route for link-local address
ff00::/8         ::       UA  256 0 0 eth0 <- Interface route for all multicast addresses
::/0             ::       UDA 256 0 0 eth0 <- Automatic default route
</pre>
# Add IPv6 route through gateway
On Linux how to manually add an IPv6 gateway/router
<pre>
$ /sbin/route -A inet6 add [ipv6network]/[prefixlength] gw [ipv6address] [dev [device]]
</pre>
A device can be needed, too, if the IPv6 address of the gateway is a link local one.

Following shown example adds a route for all currently global addresses (2000::/3) through gateway 2001:0db8:0:f101::1
<pre>
$ /sbin/route -A inet6 add 2000::/3 gw 2001:0db8:0:f101::1
</pre>
# Neighbour Discovery
With the following command you can display the learnt or configured IPv6 neighbours:
<pre>
$ ip -6 neigh show [dev [device]]
</pre>
The following example shows one neighbor, which is a reachable router
<pre>
$ ip -6 neigh show
fe80::201:23ff:fe45:6789 dev eth0 lladdr 00:01:23:45:67:89 router nud reachable
</pre>
# OSX
## Network Discovery Protocol
Use ndp on OSX to list your neighbours/routing table:
<pre>
$ sudo ndp -an
Neighbor                        Linklayer Address  Netif Expire    St Flgs Prbs
::1                             (incomplete)         lo0 permanent R      
fdc7:1ed2:1edc:xxxx:::4687:52fc (incomplete) utun0 permanent R      
fe80::1%lo0                     (incomplete)         lo0 permanent R      
fe80::1240:xxxx::5bf6%en1   10:40:f3:9b:5b:f6    en1 permanent R      
fe80::a221:xxxx::3f1b%en1   a0:21:b7:40:3f:1b    en1 9h4m53s   S  R
</pre>
## Nmap
You need the -6 flag and do not forget to add the interface after the ipv6 address eg %en1 Example:
<pre>
$ sudo nmap -6 fe80::ba27:ebff:fe60:89af%en1  -p 4444
Starting Nmap 6.25 ( http://nmap.org ) at 2013-03-12 22:29 GMT
Nmap scan report for fe80::ba27:ebff:fe60:89af
Host is up (0.0048s latency).
PORT     STATE SERVICE
4444/tcp open  krb524
MAC Address: B8:27:EB:60:89:AF (Raspberry Pi Foundation)
</pre>
# Conclusion
You should now have a basic understanding of enumerating, mapping and investigating IPv6 protocols on the network. **Remember, Netscylla does not advocate hacking other networks that do not belong to you!** Instead practise on your own home networks or Raspberry Pi’s, and one day you to can be a network Ninja!


