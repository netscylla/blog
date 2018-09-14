---
layout: post
title:  "Defend against Domain-Fronting with JA3"
date:   2018-08-09 13:49:33 +0000
tags: [SSL, pentest, redteam, blueteam]
---
Modern day cyber-criminals and legitimate Red-Teaming are increasingly using cyber squatted domains (including bitsquatting and homoglyphs) to register a domain that is very similar to the victim/targets’s corporate domain:
* www.companyname.com (legitimate domain)

may become any number of:
* www.companyname.online
* www.company-group.com
* www.company.co.uk
* www.c0mpany.com

These (or similar domains) are often intentionally chosen to avoid suspicion and human analysis, and can be difficult for the blue-team to determine, if some of these are new domains registered by legitimate developers, for new products and/or services? Or if these domains are part of a sophisticated attack platform, ready to exfiltrate corporate secrets?

As modern C2 implants are typically using encryption and SSL traffic intrusion detection becomes very difficult and near impossible without SSL termination and further analysis — this then has privacy and GDPR concerns! Therefore, domain fronting C2 infrastructures still have an advantage of remaining undetected by several existing products on the security stack.

Combining unsupervised machine learning with JA3 is incredibly powerful for the detection of domain fronting. Domain fronting is a popular technique to circumvent censorship and to hide C2 traffic. While some infrastructure providers take action to prevent domain fronting on their end, it is still prevalent and actively used by attackers.

Unsupervised machine learning makes the detection of domain fronting without having to break up encrypted traffic possible by combining unusual JA3 detection with other anomalies such as beaconing. A good start for a domain fronting threat hunt? A device beaconing to an anomalous CDN with an unusual JA3 hash.

Salesforce have started their JA3 project here:
* https://github.com/salesforce/ja3

Within their Readme on the front page they cite some examples that are of benefit to security researchers and fellow blue teamers:

JA3 fingerprint for the standard Tor client:
```
e7d705a3286e19ea42f587b344ee6865
```
JA3 fingerprint for the Dridex malware:
```
74927e242d6c3febf8cb9cab10a7f889
```
JA3 fingerprint for Metasploit’s Meterpreter (Linux):
```
5d65ea3fb1d4aa7d826733d2f2cbbb1d
```
Therefore, adding J3 support into your IDS/IPS could be advantageous in the early detection of possible attackers (where-there they by criminals or Red-Teamers).

## Conclusion
JA3 is not a magic wand or fabled silver bullet to end all remote-based malware compromises. It is a signature-based solution; which shares the exact limitations of other similar defensive products (such as Anti-Virus, Data Loss Prevention, Adaware, etc)that rely on predefined threats or blacklists! Leaving the blue team having to play a constant game of catch-up with innovative attackers and potential criminals.

However, JA3 provides a novel means of identifying TLS/SSL applications, JA3 hashing can be leveraged as a powerful network behavioural indicator, within machine learning infrastructures. Or at the very least an additional metric that can flag the use of unauthorised or potential dangerous software, in the initial stages of C2/malware communications.
