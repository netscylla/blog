---
layout: post
title:  "HTTP Public Key Pinning (HPKP)"
date:   2018-03-11 13:49:33 +0000
tags: [SSL, pentest, redteam, blueteam]
---
With SSL interception and Man-in-the-Middle attacks becoming more frequent. In an attempt to reduce fraudulent SSL certificate interception attempts browsers implement Public Key Pinning. But configure this wrong, and it could be a big and expensive mistake.

HPKP has the potential to lock out users for a long time if used incorrectly! The use of backup certificates and/or pinning the CA certificate is recommended.

![](/blog/assets/https.jpeg)

> HPKP is an Internet security mechanism delivered via an HTTP header which allows HTTPS websites to resist impersonation by attackers using mis-issued or otherwise fraudulent certificates. In order to do so, it delivers a set of public keys to the client (browser), which should be the only ones trusted for connections to this domain. — Wikipedia

## Enabling HPKP
To enable this feature for your site, you need to return the Public-Key-Pins HTTP header when your site is accessed over HTTPS:
<pre>
Public-Key-Pins: pin-sha256="base64=="; max-age=expireTime [; includeSubDomains][; report-uri="report"]
</pre>
## Extracting the Base64 encoded public key information
The following commands will help you extract the Base64 encoded information from a key file, a certificate signing request, or a certificate.
```
openssl rsa -in my-rsa-key-file.key -outform der -pubout | openssl dgst -sha256 -binary | openssl enc -base64
openssl ec -in my-ecc-key-file.key -outform der -pubout | openssl dgst -sha256 -binary | openssl enc -base64
openssl req -in my-signing-request.csr -pubkey -noout | openssl pkey -pubin -outform der | openssl dgst -sha256 -binary | openssl enc -base64
openssl x509 -in my-certificate.crt -pubkey -noout | openssl pkey -pubin -outform der | openssl dgst -sha256 -binary | openssl enc -base64
```
The following command will extract the Base64 encoded information for a website.
<pre>
openssl s_client -servername www.example.com -connect www.example.com:443 | openssl x509 -pubkey -noout | openssl pkey -pubin -outform der | openssl dgst -sha256 -binary | openssl enc -base64
</pre>
# Example HPKP Header
The following header is an example of pinning the server certificate to a domain for 2 months:
```
Public-Key-Pins:
  pin-sha256="cUPcTAZWKaASuYWhhneDttWpY3oBAkE3h2+soZS7sWs="; 
  pin-sha256="M8HztCzM3elUxkcjR2S5P4hhyBNf6lHkmjAHKhpGPWE="; 
  max-age=5184000; includeSubDomains; 
  report-uri="https://www.example.org/hpkp-report"
```
What about the danger of making a mistake?
The danger of making a mistake with HPKP is that you can possibly lock a certificate to a domain for the period of time you specified. If your website/service is critical for your business this could end up being a very expensive mistake. But luckily there is an alternative header to practise on first.

We can use Public-Key-Pins-Report-Only header instead of Public-Key-Pins

## Report-Only header
This header only sends reports to the report-uri specified in the header and does still allow browsers to connect to the webserver even if the pinning is violated.

## Pin the Intermediate/Signing CA
Alternatively, you could place the pin on an intermediate certificate of the CA that issued the server certificate. This eases certificates renewals and rotations. But then this is similar to CAA, and CAA is much easier to implement.

## Setting HPKP Headers on Modern Web Engines
### Apache
Adding a line similar to the following to your webserver’s config will enable HPKP on your Apache. This requires mod_headers enabled.
<pre>
Header always set Public-Key-Pins "pin-sha256=\"base64+primary==\"; pin-sha256=\"base64+backup==\"; max-age=5184000; includeSubDomains"
</pre>
### Nginx
Adding the following line and inserting the appropriate pin-sha256="..." values will enable HPKP on your nginx. This requires the ngx_http_headers_module.
<pre>
add_header Public-Key-Pins 'pin-sha256="base64+primary=="; pin-sha256="base64+backup=="; max-age=5184000; includeSubDomains' always;
</pre>
## References
* https://developer.mozilla.org/en-US/docs/Web/HTTP/Public_Key_Pinning
