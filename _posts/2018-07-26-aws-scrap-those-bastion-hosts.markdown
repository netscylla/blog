---
layout: post
title:  "AWS — Scrap those bastion hosts"
date:   2018-07-26 13:49:33 +0000
tags: [Cloud, AWS, pentest, redteam, blueteam]
---
Recently, we attended an AWS workshop, where there appeared to be a change of opinion on the use of bastion hosts. Instead AWS Systems Manager (SSM) is viewed as a more secure alternative to manage your EC2 instances, with the additional benefit of lower administration costs.

![](/blog/assets/aws_bastion.png)

## Why Use Bastion Hosts?
Many cloud training outfits and AWS themselves have boasted several reasons, and good-architecture papers as to utiltise bastion hosts. Some of these reasons include:
* Security
* Compliance
* Logging
* Access & Scalability

But are these actual true benefits?
* Security — what is the actual benefit of security? You have an extra EC2 instance to manage. You need to update and patch on a regular basis. Maybe a host based firewall to manage and additional Security Groups to configure and maintain. — This is a lot of work
* Compliance — we touched on this above; yet another EC2 instance to harden, maintain and update.
* Logging — You need to configure Cloudwatch logging, and maybe additional onhost logging (auditd, ossec, user-monitoring).
* Access & Scalability — You need to configure auto-scaling groups to ensure there is always a bastion instance available to manage your back end/ private instances.

In reality, this is just extra work for the Devsecops team and not really necessary!

## Scrapping the bastion host
By scrapping the bastion host we can utilise the SSM capability to manage our EC2 instances. Access is controlled through IAM, and we can create fine grained access-control (FGA) through the use of inline-policies.

Logging is still maintained by Cloudwatch and Cloudtrail, and SSM records commands issued and output.

We do not need double firewalls, we can easily maintain NACLs and SecurityGroups.

Compliance can alternatively be managed by AWS Config.

Accessibility and scalability are no longer an issue as SSM is highly available.

Straight away we have reduced the complexity of security, access & scalabililty, and due to the need for less EC2 instances — we have reduced our monthly AWS bill.

You can additionally utilise AWS services like S3 storage, SNS, and cloudwatch!

## But what if the bastion was acting like a proxy?
If the bastion was used as a proxy to forward all relevant traffic to your private subnet, we need to make one more change. Simply use an AWS NAT Gateway (to connect the EC2 instance)connected Internet Gateway (IGW)!

With AWS IGW you can also utilise Application Load Balancer (ALB) and WAF to further secure your web applications served over AWS. These are high availability services with build in DoS protection. Which should not only increase your level of security, but increase the availability and speed of your applications.

## New Architecture

![](/blog/assets/aws_bastion_2.png)

## References
* https://aws.amazon.com/blogs/mt/replacing-a-bastion-host-with-amazon-ec2-systems-manager/
* https://aws.amazon.com/premiumsupport/knowledge-center/attach-igw-elb/
