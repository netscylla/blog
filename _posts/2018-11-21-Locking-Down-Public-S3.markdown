---
layout: post
title:  "Locking Down S3 Public Access"
description: "A walkthrough on utilising new AWS features/policies, to prevent accidentally making S3 buckets public."
date:   2018-11-21 13:49:33 +0000
tags: [blue-team, devops, cloud, AWS, S3, buckets]
---
![S3](/assets/s3.png)

# Introduction

Amazon AWS has yet again, pushed changes into the way S3 buckets are managed. This is to prevent the never
ending disclosures of companies accidentally publicising their data in public.  With EU's GDPR and USA's Privacy 
Shield 2018 laws, there are now severe fines for organisations that do not adequately manage their data security.

This is not our first post about leaky S3 buckets, our previous post can be found here:
 * [https://www.netscylla.com/blog/2018/02/15/s3-security-leaky-bucket-prevention-primer.html](https://www.netscylla.com/blog/2018/02/15/s3-security-leaky-bucket-prevention-primer.html)

# Whats New?

## Logging into S3
When you log into S3 you should see warning text next to your bucket alerting you that some objects could be public:

![Public Access Possible](/assets/s3_pub_1.png)

## S3 Permissions

Now under **Permissions** we can see four new options for fine grained control over the public accessibility
of our data.  Now within a few clicks, we can remove all public access, and prevent future public access (unless 
you have full control over the bucket). 
 
![Screenshot](/assets/s3_pub_2.png)
 
 
![Screenshot2](/assets/s3_pub_3.png)

# Conclusion
This is another attempt by Amazon, to prevent leaky S3 buckets.  Hopefully, developers and system administrators (DevOps)
can use these policies to prevent abuse or lazyiness, and stop all those companies dumping their
data in public buckets for the world to see, access and copy.