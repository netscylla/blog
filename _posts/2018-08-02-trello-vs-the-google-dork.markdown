---
layout: post
title:  "Trello vs the Google Dork"
date:   2018-08-02 13:49:33 +0000
tags: [Web Application, pentest, redteam, blueteam]
---
![](/blog/assets/passwords.png)

## What is Trello?
Trello is a web-based project management application originally made by Fog Creek Software in 2011. Basically, its on online Kanban board.

## What is a Goole Dork?
A Google Dork query, sometimes just referred to as a dork, is a search string that uses advanced search operators to find information that is not readily available on a website. In other words, we can use Google Dorks to find vulnerabilities, hidden information and access pages on certain websites.

## Trello Dorks
We can use Google dorks to extract sensitive information such as credentials, cloud accesskeys, ssh private keys, and service passwords for various groups and organisations.

Trello Dork examples:
```
site:trello.com intext:@gmail.com
site:trello.com intext:accesskey
site:trello.com intext:sql intext:sa
site:trello.com intext:postgresql intext:root
site:trello.com intext:BEGIN RSA PRIVATE KEY
```

![](/blog/assets/trello_1.png)

![](/blog/assets/trello_2.png)

![](/blog/assets/trello_3.png)

If you are looking for a better way to share passwords among your employees the answer is, well, don’t. There are several good Secure Identity Management applications online offering a single sign-on (SSO) option instead. Simply put, you can give each of your employees a single, unique password granting them access to numerous applications. This as opposed to handing them dozens of master credentials to everything your company or organisation holds dear.

## References
This whole issue came to light thanks to:
* http://reddit.com/r/security/comments/93n6ln/stop_using_trello_as_a_password_manager_how_to/

