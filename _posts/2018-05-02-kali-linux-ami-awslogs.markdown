---
layout: post
title:  "Kali-Linux AMI & AWSLogs"
date:   2018-05-02 13:49:33 +0000
tags: [Cloud, AWS, pentest, redteam, blueteam]
---
Today we had to build a Kali Linux AMI for some pentesters. Kali itself is easy enough to install from the market place and configure.

But on applying our hardening and logging scripts we encountered an error with awslogs — the script didnt know about the flavour of Kali linux (secretly ubuntu based).

So we patched awslogs-agent-setup.py to make things easy in future with our automation processes. Our patched script can be found here: [https://gist.github.com/netscylla/a94f49095c5ea8774b2b691a51360d7e](https://gist.github.com/netscylla/a94f49095c5ea8774b2b691a51360d7e)

{% gist a94f49095c5ea8774b2b691a51360d7e %}
