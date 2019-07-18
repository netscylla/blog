---
layout: post
title:  "Whois API for phishing and brand-abuse detection"
description: "A script / flask-web frontend for querying recent changes in whois records given a specific keyword/domain-name, for early detection on phishing and brand-abuse."
date:   2019-07-17 14:49:33 +0000
tags: [Threat Intel, Threat Hunting, Blue Team]
---

![suspicious word doc](/blog/assets/whois1.png)

## The common problem
One of the biggest security concerns from our customers is web-based phishing campaigns and brand abuse.

Attackers to create fake domains and attempt to legitamise them through signing freely available SSL certificates. A cheaper alternative is usually to attack the brand through subdomains, available cheaper top-level-domains (eg. *.dk, *.vip, *.online), or typo-squatted domains.

Instead of going after gmail.com, its easier for attackers to include other names or keywords and register those domains. E.g.
 * wwwexample.com
 * examplebanking.com
 * secure-example.com

We cannot use Certificate Transparency Alerts (ct-alerts) unless the attackers have registered for SSL certificates. Some attackers may choose not to implement SSL inorder to not trigger ct-alerts. Our trusted customers even asked us (Netscylla) if we could improve on our detection and response times, by triggering on the registration of actual domain names, or rather specific keyword's (E.g. their brands, their corporation names, etc).

## Our Solution
Netscylla's solution to the problem is to *'mine'*, the Whois registrars for the organisation's name(s) update /record change information.
Now we said it was tricky... But the cost of scraping and maintaining these records quickly accelerated, we then found that some private organisations based out of USA
were also solving the same problem, and were slightly ahead in terms of scalability and backing. Therefore we abandoned our costly functions for a more economical use of
using their fully featured apis.

## Whois-search

### Usage

#### Docker

Build the Docker image
```
docker build -t whois .
```
Run the docker image, and portforward TCP/5000
```
docker run -p 5000:5000 whois
```
#### Python

For python, use pip to install the required modules, save the api key as an enviroment variable, and use flask to serve the application
```
 python3 -m venv .venv
 source .venv/bin/activate
 pip install -r requirements.txt
 flask run --host 0.0.0.0
```

### Running

In any browser
 * http://127.0.0.1:5000

### Example 1 - example keyword
We will enter **github** and **14** vaules into their respective fields

![screenshot 1 - web page](/blog/assets/ns_whois_screen.png)

and low and behold the results:

![screenshot 2 - results](/blog/assets/ns_whois_screen2.png)


## The code
We have hosted our demo code on github:
 * [https://github.com/netscylla/whois_search](https://github.com/netscylla/whois_search)
The demo code makes use of the freely available api, the advantages are its free, the disadvantages are that the first four characters of any domain-name are masked. 

## Disclaimer
Netscylla or its staff cannot be held responsible for any abuse relating from this blog post. 
This post is to raise awareness in threat hunting and brand-abuse alerting to help aid specific organisations in defending their perimeter or brand based on keyword patterns. REMEMBER: It is illegal to attempt unauthorised access on any system you do not personally own, unless you have explicit permission in writing from the system owner!
