---
layout: post
title:  "What are CAA Records and their use with HTTPS Websites"
date:   2018-03-05 13:49:33 +0000
tags: [SSL, pentest, redteam, blueteam]
---
All certificate authorities should now utilise a Domain Name System (DNS) record that allows HTTPS website owners to restrict who can issue SSL certificates for their domain names. It’s a long-needed defence against the issue of fraudulent certificates, a security risk that domain owners had few protections for until now.

![](/assets/caa.png)

# Certification Authority Authorization (CAA)
The Certification Authority Authorization (CAA) is a DNS-based mechanism that allows domain administrators to specify which specific certificate authorities are allowed to issue SSL certificates for their domain names.

The CA/Browser Forum, an organisation that combines major browser vendors and certificate authorities, voted last year to make CAA checking mandatory as part of the certificate issuance process. The CA/Browser Forum maintains the “Baseline Requirements for the Issuance and Management of Publicly-Trusted Certificates,” an industry-accepted set of guidelines that all CAs have to follow.

# The good…
CAA creates a DNS mechanism that enables domain name owners to whitelist CAs that are allowed to issue certificates for their hostnames. It operates via a new DNS resource record (RR) called CAA (type 257). Owners can restrict certificate issuance by specifying zero or more CAs; if a CA is allowed to issue a certificate, their own hostname will be in the DNS record. For example, this is what someone’s CAA configuration could be (in the zone file):
<pre>
netscylla.com. CAA 128 issue "letsencrypt.org"
</pre>
The CAA DNS record supports three properties:
* issue — allow specifying one or more certificate authorities
* issuewild — allow specifying one or more wildcard certificates
* iodef — reporting/alerting purposes, CAs will send reports to the email address specified in this record. e.g. a fraudulent certificate request
And that’s all there is to it. Before issuing a certificate, CAs are expected to check the DNS record and refuse issuance unless they find themselves on the whitelist. (Note: the number “128” is a flags byte with its highest bit set, meaning the directive use is considered critical and must be followed.)

# The bad and the ugly…
Of course, if hackers manage to obtain a certificate from a CA authorised by the domain owner, there will be no report! Also, if hackers break into a CA they might be able, depending on what kind of access the gain, to bypass CAA checks entirely. So, CAA with its iodef reporting is not a silver bullet, but it’s better than nothing.

CAA policies can also be set per subdomain. For example, for subdomain.example.com, a CA will first check the CAA policy for the subdomain and then for example.com. If a CAA record exists for subdomain.example.com, it will take precedence over that for example.com.

Also, if you’re using a content delivery network or hosting provider that provides its own certificates through a different CA, this has to be taken into account.

# Attackers moving forward
Many CAs publish all the certificates they issue to a public log as part of a framework called Certificate Transparency (CT). It’s a good idea to check these CT logs — for example through services like crt.sh — in order to create an inventory of all SSL certificates used on your domains and subdomains and build a list of certificate providers for those websites.

# Conclusion
CAA records are not a great protection mechanism, but its better than nothing, and can help defend against some attacks. However, with a large adoption of free SSL providers like LetsEncrypt, the protection becomes non-existant with shared CAs.

There is HTTP Public Key Pinning (HPKP), but this can be potentially very dangerous if you mess things up! But with CAA it’s not possible to abuse and bring down a web property. Whereas HPKP can kill your business if you really misconfigure or make a mistake in the setup.

In the end, one way to look at CAA is “public key pinning for normal web properties”, who would find HPKP to complicated and scary to use?

# References
* https://blog.qualys.com/ssllabs/2017/03/13/caa-mandated-by-cabrowser-forum
* https://www.internetsociety.org/blog/2017/06/caa-mandated-by-cabrowser-forum/
