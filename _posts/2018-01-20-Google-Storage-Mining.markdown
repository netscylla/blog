---
layout: post
title:  "Google Storage Mining"
date:   2018-01-20 13:49:33 +0000
tags: [malware, powershell, pentest, redteam, blueteam]
---

![](/assets/google_store.png)

In the recent months we have seen much media coverage over S3 bucket leaks, where developers and organisations have forgotten to check the Access-Control-List’s (ACLs) that protect the authorisation of whom can access the data stored within these particular cloud containers. This has lead to some embarrassing moments for both Governments and Private companies.

Most of the tools out there are currently configured to only scan Amazon S3! and we at Netscylla thought, why not Google?

Google’s buckets are typically secure by default! Developers / administrators need to manually add the ‘allUsers’ group to ‘Storage View’ (or more dangerously read&write) permissions. But from experience we know that during testing or development, security settings are often relaxed to help speed up progress; this can lead to mistakes, where weak permissions or overly open ACLs are left in place on Storage Buckets, when they should have been locked-down or removed!

Today, Netscylla release GBucketDump, its a small fork of the original code AWSBucketDump from Jordan Potti (https://twitter.com/ok_bye_now).

# How To Run GBucketDump
The code runs in exactly the same way you would run Jordan’s AWSBucketDump. We also added in a flag (-o) to specify an organisation name, should you wish to target and enumerate possible Google Buckets for your company or educational institution:
<pre>
$ python ./GBucketDump.py -h
usage: GBucketDump.py [-h] [-o ORGANISATION] [-D] [-d SAVEDIR] -l HOSTLIST
[-g GREPWORDS] [-m MAXSIZE] [-t THREADS]
optional arguments:
-h, — help show this help message and exit
-o ORGANISATION target an organisation
-D Download files. This requires significant diskspace
-d SAVEDIR if -D, then -d 1 to create save directories for each bucket
with results.
-l HOSTLIST
-g GREPWORDS Provide a wordlist to grep for
-m MAXSIZE Maximum file size to download.
-t THREADS thread count.
</pre>
# Example Output

Below is a brief example, of enumerating a public bucket (now destroyed) that Netscylla created for the purpose of testing the code:
<pre>
$ python ./GBucketDump.py -l netscylla.txt -g interesting_Keywords.txt -m 500000 -d out -D -t 5
Downloads enabled (-D), and save directories (-d) for each host will be created/used
queuing http://storage.googleapis.com/netscylla/
queuing http://storage.googleapis.com/netscylla-dev/
queuing http://storage.googleapis.com/netscylla-test/
queuing http://storage.googleapis.com/netscylla-prod/
fetching http://storage.googleapis.com/netscylla-dev/
fetching http://storage.googleapis.com/netscylla/
fetching http://storage.googleapis.com/netscylla-test/
fetching http://storage.googleapis.com/netscylla-prod/
Pilfering http://storage.googleapis.com/netscylla-test/
Collectable: http://storage.googleapis.com/netscylla-test/logo.jpg
Downloading http://storage.googleapis.com/netscylla-test/logo.jpg
Collectable: http://storage.googleapis.com/netscylla-test/user.txt
local out/storage.googleapis.com/netscylla-test/logo.jpg
Downloading http://storage.googleapis.com/netscylla-test/user.txt
local out/storage.googleapis.com/netscylla-test/user.txt
http://storage.googleapis.com/netscylla-dev/ is not accessible
http://storage.googleapis.com/netscylla-prod/ is not accessible
http://storage.googleapis.com/netscylla/ is not accessible
Finshed enumeration. Starting Download...
Cleaning Up Files
</pre>
# Code
The code has been released as open-source, for educational purposes only:

* [netscylla/GBucketDump](https://github.com/netscylla/GBucketDump)

# Check Your Storage Permissions
Check your permissions by visiting the following URL:

https://console.cloud.google.com/storage/browse

And Click ‘Edit Bucket Permissions’ to review the Users, Groups and ACL’s applied to your buckets.

Just like AWS it is important to remember the what the following two accounts actually mean:

* **allUsers** = everyone with anonymous access
* **allAuthenticatedUsers** = everyone with a Google Account

# References
* [nagwww/s3-leaks](https://github.com/nagwww/s3-leaks)
* [Australian Broadcasting Corporation leaks passwords, video from AWS S3 bucket](https://www.theregister.co.uk/2017/11/16/australian_broadcasting_corporation_leaks_data_from_s3_bucket/)
* [www.scmagazine.com](https://www.scmagazine.com/national-credit-federation-unsecured-aws-s3-bucket-leaks-credit-personal-data/article/710743/)
