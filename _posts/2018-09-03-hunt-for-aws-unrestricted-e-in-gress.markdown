---
layout: post
title:  "Hunt for AWS Unrestricted E/In-gress"
date:   2018-09-03 13:49:33 +0000
tags: [Cloud, AWS, pentest, redteam, blueteam]
---
Centre of Information Security (CIS) for Amazon Web Services has a networking section and two points related to security management of AWS firewalls or security groups. More specifically section 4 Networking has two checks for unrestricted ingress on TCP ports 22 (SSH) and 3389 (RDP). At Netscylla we decided to create two scripts that would highlight security groups and associated instances that run exposed services to/from the internet.

## Trusted Advisor
For those with minimalistic accounts, or even support contracts ‘ Trusted Advisor’ is an excellent source for potential weaknesses. But what if we wanted a more generic audit, to flag any instances that expose any service to the public internet (0.0.0.0/0 or ::0)

![](/blog/assets/trusted_advisor_1.png)

That is why we created two simple python scripts to help fish out mis-configured security groups and ec2 instances:
* [https://github.com/netscylla/AWS_Scripts/blob/master/hunt_egress.py](https://github.com/netscylla/AWS_Scripts/blob/master/hunt_egress.py)
* [https://github.com/netscylla/AWS_Scripts/blob/master/hunt_ingress.py](https://github.com/netscylla/AWS_Scripts/blob/master/hunt_ingress.py)

## hunt_egress.py
Makes us aware of any servers that are allowed unrestricted access to the internet:
```
$ python3 ./hunt_egress.py
Groups containing unrestricted Egress:
{'sg-080b3563', 'sg-66e88b0d', 'sg-da12d9b1', 'sg-891259e1'}
Found i-023abc: sg-66e88b0d : Sample Webserver
Webserver
Ubuntu
Found i-021ae25cff10b4a6a: sg-891259e1 : Test web server
Webserver
Redhat
Found i-0bf032: sg-080b3563 : test database
Webserver
Ubuntu
```
## hunt_ingress.py
Makes us aware of any servers that are allowed unrestricted access from the internet:
```
$ python3 ./hunt_ingress.py
Groups containing unrestricted Ingress:
{'sg-891259e1'}
Found i-021ae25cff10b4a6a: sg-891259e1 : Test web server
Webserver
Redhat
```
## Conclusion
These scripts are still very much in beta, but they can be very helpful in threat hunting and incident response for knowing which ec2 instances sit on the perimeter, and quickly denoting offending or suspicious security groups.

## Future Improvements
We can improve on them by adding arguments to control accounts and region access. We also have a bit more work of parsing the security groups for offending rules, rather than leaving it to the user to manually check the results.
