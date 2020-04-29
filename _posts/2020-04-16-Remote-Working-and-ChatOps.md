---
layout: post
title:  "Remote Working & ChatOps"
description: "A brief post on how to setup on premise chat-ops"
date:   2020-04-16 13:00:01 +0000
tags: [Blue-Team, Red-Team, Pentest, DevSecOps]
---

![ChatOps](/blog/assets/rocketchat.png)

## Introduction
With the Covid-19 pandemic continuing, and a number of security breaches occuring on misconfigued and insecure cloud platforms. Attackers have been targeting Corporation's Operational environments in an attempt to leverage access into the corporate domain for Intellectual Property Theft (IP Theft), installing bitcoin miners, or other malware/ransomware.

A number of our clients have asked us to help them develop an in-house/on-premise solution, so that their ChatOps/DevOps environment does not have to really on 3rd parties, or multi-hosted cloud environments.  It can be managed in-house by their own teams, and there is a reduced risk of IP being inadvertenly released into the public internet. Now there are many different vendors in this space. But as Netscylla has used the community version of [Rocket.Chat](https://rocket.chat) as on on-premise solution in the past, we felt that this was the right possible solution for our clients.

## Our Proposed Solution - RocketChat

### Installation
You can either choose to install RocketChat manually using:
 * [https://rocket.chat/install](https://rocket.chat/install)

Or there are many 1-click solutions from numerous cloud-hosting-providers

We choose to use the version that installs through a fresh version of Ubuntu Server 18.04. You can install RocketChat from the installation medium (which uses snaps to install and manage the application).  It is quick and easy.  All that is left is to configure the SSL Certificates, add users, or LDAP/AD support, install some useful plugins and integrations and your Chat server is complete.

### Ubuntu, RocketChat install and configure SSL
Out of the box the Rocket.Chat is installed using default mongo and rocketchat settings which can be deemed insecure.  In this example we will use Lets-Encrypt to demonstrate how easy it is to configure SSL encrypted chat for Rocket.Chat.  We prefer to use Nginx as a reverse proxy, and manage the SSL configuration there, and bind rocketchat services to the localhost.

Install additional packages
```
apt-get install nginx certbot
```
**Don't forget to configure your DNS settings, and wait at least 10mins before requesting certificates from Lets-Encrypt**

```
certbot certonly -d <your FQDN for rocketchat here>
```
This should install certificates at:
 * /etc/letsencrypt/live/your_fqdn/cert.pem
 * /etc/letsencrypt/live/your_fqdn/privkey.pem

Edit */etc/nginx/sites-enabled/default* and be sure to use your actual hostname in lieu of the sample hostname “your_fqdn.com” below.
```
# Upstreams
upstream backend {
    server 127.0.0.1:3000;
}

# HTTPS Server
server {
    listen 443;
    server_name your_fqdn.com;

    # You can increase the limit if your need to.
    client_max_body_size 200M;

    error_log /var/log/nginx/rocketchat.access.log;

    ssl on;
    ssl_certificate /etc/letsencrypt/live/<your fqdn>/privkey.pem;
    ssl_certificate_key /etc/letsencrypt/live/<your fqdn>/privkey.pem;
    ssl_protocols TLSv1.2, TLSv1.3; # don’t use SSLv3 ref: POODLE

    location / {
        proxy_pass http://backend/;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
        proxy_set_header Host $http_host;

        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forward-Proto http;
        proxy_set_header X-Nginx-Proxy true;

        proxy_redirect off;
    }
}
Restart Nginx: service nginx restart
```

### Disable Caddy and Change the Bind IP
Since we're using Nginx as the reverse proxy and for the SSL we can disable Caddy.  Caddy is the default web-server/reverse proxy for Rocket.Chat. Using snap this is easy as:
```
snap set rocketchat-server caddy=disable
```
To change the default bind_ip from 0.0.0.0 to 127.0.0.1, we use snaps again
```
snap set rocketchat-server port="127.0.0.1:3000"
```
You may get an error on the last command that the port is invalid? But it does actually work.

Next restart Rocketchat and check the listening service
```
systemctl restart rocketchat-server
netstat -antp |grep 3000
tcp 0 127.0.0.1:3000 127.0.0.1:39928 TIME_WAIT -
```

Hopefully, you should be up and running!

![rocket chat example](/blog/assets/rocketchat2.png)

### Integrations
We won't cover the details here, but integrations are exactly the same as you would configure webhooks in the most popular ChatOps platforms.  For an example of integrating Github & Gitlab alerts, Rocket.Chat have already provided working examples, that are easy to follow here:
 * [https://rocket.chat/docs/administrator-guides/integrations/github/](https://rocket.chat/docs/administrator-guides/integrations/github/)
 * [https://rocket.chat/docs/administrator-guides/integrations/gitlab/](https://rocket.chat/docs/administrator-guides/integrations/gitlab/)

## Final Thoughts
Do not forget to harden the base Operating System, configure a suitable firewall and possibly include a VPN such as [Wireguard](https://www.netscylla.com/blog/2020/03/24/Wireguard-VPN.html), to ensure secure access to your new ChatOps server.

You probably want to discuss with the Developers on what integrations they need, so that appropriate bots alert your staff of networking/security/development events.
