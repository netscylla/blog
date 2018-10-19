---
layout: post
title:  "Secret Holes Behind the Common Load-Balancer"
date:   2018-07-19 13:49:33 +0000
tags: [networking, pentest, redteam, blueteam]
---
![](/assets/loadbalancer.jpeg)

Those new to Information Security may not know what a load-balancer is? Those that have even been in the industry a few years, may not fully understand them? But due to a new flux of High-Availability (HA) application architectures due to Cloud Provisioners, being able to understand these mythical beasts, and assess individually the servers behind them may pay off considerably in the red-team?

## What is a load balancer?
A load balancer acts as the “traffic cop” sitting in front of your servers and routing client requests across a number of tiered-servers in-order to maximise speed and capacity utilisation and to ensure that no single server is overworked - which could degrade performance. If a single server goes down, the load balancer redirects traffic to the remaining online servers. When a new server is added to the server group, the load balancer automatically starts to send requests to it.

In this manner, a load balancer performs the following functions:
* Distributes client requests or network load efficiently across multiple servers
* Ensures high availability and reliability by sending requests only to servers that are online
* Provides the flexibility to add or subtract servers as demand dictates

## Load Balancing Algorithms
Different load balancing algorithms provide different benefits; the choice of load balancing method depends on your needs:
* Round Robin — Requests are distributed across the group of servers sequentially.
* Least Connections — A new request is sent to the server with the fewest current connections to clients. The relative computing capacity of each server is factored into determining which one has the least connections.
* IP Hash — The IP address of the client is used to determine which server receives the request.

## Session Persistance
The best load balancers can handle session persistence as needed. Another use case for session persistence is when an upstream server stores information requested by a user in its cache to boost performance. Switching servers would cause that information to be fetched for the second time, creating performance inefficiencies.

**In AWS we call these Sticky Sessions**.

## Dynamic Configuration of Server Groups
Many fast-changing applications require new servers to be added or taken down on a constant basis. This is common in environments such as the Amazon Elastic Compute Cloud (EC2), which enables users to pay only for the computing capacity they actually use, while at the same time ensuring that capacity scales up in response traffic spikes. In such environments it greatly helps if the load balancer can dynamically add or remove servers from the group without interrupting existing connections.

## How to identify load balancing
There are a few ways in which to tell if the request is hitting such a load balancing system. One way earlier in the discussion of DNS load balancing. A simple series of ICMP messages sent interspersed with flushing out DNS cache would easily reveal if this was the case. If, DNS load balancing is present, you could attempt to do your testing on the IP addresses rather than the DNS name in order to lower the chance of running into any of the issues discussed.

Example of detecting DNS load balancing via ping (notice the TTL and IP address values are different):
```
PING www.google.co.uk (216.58.212.99): 56 data bytes
64 bytes from 216.58.212.99: icmp_seq=0 ttl=54 time=30.412 ms
64 bytes from 216.58.212.99: icmp_seq=1 ttl=54 time=20.043 ms
64 bytes from 216.58.212.99: icmp_seq=2 ttl=54 time=22.393 ms
$ ping www.google.co.uk
PING www.google.co.uk (216.58.213.67): 56 data bytes
64 bytes from 216.58.213.67: icmp_seq=0 ttl=55 time=30.559 ms
64 bytes from 216.58.213.67: icmp_seq=1 ttl=55 time=23.026 ms
64 bytes from 216.58.213.67: icmp_seq=2 ttl=55 time=22.366 ms
```
Another way is due to a mis-configuration of the load balancing devices. Some devices put values into cookies that identify them clearly.
```
BIGIPCOOKIE — F5 Loadbalancer
BigIPcookie = 673059850.20480.0000
```
## Reverse a BigIP Cookie
Use this linux one-liner to reveal the IP address of the webserver behind the BigIP:
```
echo 673059850.20480.0000 | perl -ne'print join ".", map {hex} reverse ((sprintf "%08x", split /\./, $_) =~ /../g);';echo -e ""
```
## Using HPing3
An example using Hping3, notice the ipid (bold) is different on each request:
```
hping3 www.microsoft.com -S -p 80 HPING www.microsoft.com (eth0 207.46.193.254): S set, 40 headers + 0 data bytes len=46 ip=207.46.193.254 ttl=242 id=64800 sport=80 flags=SA seq=0 win=8190 rtt=106.0 ms len=46 
ip=207.46.193.254 ttl=242 id=6989 sport=80 flags=SA seq=1 win=8190 rtt=106.3 ms len=46 
ip=207.46.193.254 ttl=242 id=45431 sport=80 flags=SA seq=2 win=8190 rtt=106.9 ms
```
## The Common Problem (and often missed…)
Why do we care about load-balancing? Why? Because as a pentester or red-teamer you might not be doing a thorough job, you might have missed a vulnerability that permits an attacker to take control of an underlying webserver?

### How?
As we have highlighted above, you may have a number of webservers or application servers behind the load-balancer, due on inconsistent builds, or auto-scaling launching mis-configured AMI (Amazon EC2 instances). You may encounter mis-configured and vulnerable web-app servers.

Depending on the flavour and access of the web/application server opens up a number of possibilities. Currently on the OWASP Top Ten we have these vulnerabilities:
* A3:2017 — Sensitive Data Exposure
* A5:2017 — Broken Access Control
* A6:2017 — Security Misconfiguration
* A9:2017 — Using Components with Known Vulnerabilities

So far in 2018 Netscylla have discovered the following types of vulnerabilities across a number of clients (which will remain anonymous):
* **Apaches /server-status/**: which leaks IP addresses, visited pages (for forced browsing), and general information gathering
* **Unsecured admin interfaces**: default/blank credentials to admin interfaces where we are free to upload webshells and gain access to the network
* **Other known application exploits**: direct exploitation of an unpatched server to obtain a direct webshell, and foothold on the the network.

## So What Now, Moving Forward?
Don’t exclude tests for load-balancers during the reconnaissance or discovery phase of your application / infrastructure pentest or even during your red-team reconnaissance phase!

It may be impractical to fully assess each individual web/app server behind a webserver? But at least ask the client during a pentest! To ensure you have direct access through the load-balancer / firewall to assess for any possible weaknesses in the server, application or admin interface components that could be missed? And as a result get attacked and compromised by any lucky attacker!

## References
* https://www.sans.org/reading-room/whitepapers/testing/identifying-load-balancers-penetration-testing-33313
* https://www.nginx.com/resources/glossary/load-balancing/
* https://www.owasp.org/images/7/72/OWASP_Top_10-2017_%28en%29.pdf.pdf
