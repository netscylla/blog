---
layout: post
title:  "Google S3 Secure Storage"
date:   2018-02-18 13:49:33 +0000
tags: [Cloud, Google, S3, pentest, redteam, blueteam]
---
Google S3 Bucket configuration is relatively secure by default. This is what should happen (by default) if you try to access any files from your bucket as an anonymous user:

![](/assets/google_s3.png)

This is because by default only the ‘object owners’ are permitted access.

# Testing Google Buckets
Below we will assess the functionality of Googles S3 buckets and IAM policies, the controls are much simpler than AWS. Therefore, Google S3 should be faster to get familiar with the controls and policies, and harder to make mistakes?

# Bucket Contents
What is actually stored in Netscylla’s test bucket at the time of testing (may not be available at the time of reading):

![](/assets/gcp.png)

**Do not click ‘Share Publicly’ unless you want your data accessible anonymously!** This changes the permissions of the data in an uncontrollable way apart from that tickbox! Also the data will become cached in Google’s CDN!

**Note**: unticking the box has no effect for 1 hour! Due to the Google Caching Timeout of 3600 secs.

This is where developers can sometimes get things wrong. And a lucky well-timed attack could mean an anonymous internet user can still access your data!

As a test-case image 256222_2.jpg was publicly shared, and immediately we unticked the ‘share publicly’ box. This has no noticeable effect as I can still access the stored object in another browser. This was the case for further 60mins.

![](/assets/google_s3_2.png)

# Google Storage & Anonymous Access
S3 Buckets should be secure by default, to give anonymous users access you must add the ‘allUsers’ to the ‘Object Viewer’ role:

![](/assets/google_s3_3.png)

This configuration enables anonymous internet users to query your bucket, and access all of your data stored within:

![](/assets/google_s3_4.png)

If we wanted to share this with all authenticated Google Users (those with gmail/googlemail/g-suite accounts) we could alternatively have opened the permissions to ‘allAuthenticatedUsers’.

# Removing Anonymous Access
Simply deleting the IAM user policies for the following accounts should stop anonymous internet users accessing your files:
* allUsers
* allAuthenticatedUsers
Simply clicking the trash can next to ‘allUsers’ (in the example above) should return the bucket to its default state.:

![](/assets/google_s3_5.png)

Then confirm the permissions are removed by revisiting the bucket as an anonymous user in another Internet browser:

![](/assets/google_s3_6.png)

**Note**: Any private files will remain private, but any previously shared files will still remain accessible while the Google server still caches them!

# Google IAM Roles
Managing IAM roles in Google is much simpler than AWS, but then you lack the granularity of AWS where you can permit/block IP ranges, and advanced features such as MFA.

For more information on Google Storage IAM policies please refer to Googles documentation:https://cloud.google.com/storage/docs/access-control/iam?

# Google Storage Logging
Google does log some information about your buckets, accessible here: https://console.cloud.google.com/logs/viewer

Just remember to change the drop-down from ‘GCE Project’ to ‘GCS Bucket’

![](/assets/gcp_2.png)

The logging data has enough details to track IAM policies being removed and applied, which UserAccount, and IP Address and Browser Information were involved in the changes. Should a breach occur? we can easily track the root-cause, of what happened? and when it happened?

![](/assets/google_s3_7.png)

More on Storage logging can be found here: https://cloud.google.com/storage/docs/gsutil/commands/logging

# Conclusion
After reading this mini blog post on Google Storage, you should have a better understanding on how Google’s Storage works, how to re-configure the permissions to your S3 buckets, and in case of an incident? Search the logs to figure out how the incident occurred.
