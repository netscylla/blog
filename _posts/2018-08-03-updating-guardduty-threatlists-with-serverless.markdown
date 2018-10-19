---
layout: post
title:  "Updating GuardDuty Threatlists with Serverless"
date:   2018-08-03 13:49:33 +0000
tags: [Cloud, AWS, pentest, redteam, blueteam]
---
![](/assets/aws_guardduty_1.png)

GuardDuty has the ability to read in custom intelligence feeds (or threat lists) from S3 buckets. But it can be quite a drag to manually download, prepare/sanitise and upload continuously to an S3 bucket; this is where we thought serverless would be a winner here — fire and forget, and always have an up-to-date reputation feed!

Below are our instructions to our first serverless example:

## Our Walkthrough
### The Lambda IAM Role
Below is an inline policy with minimal permissions needed to run our lambda script. You can name the role anything you want, but probably best to make it easier identifiable.
```
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "s3",
            "Effect": "Allow",
            "Action": [
                "s3:Put*"
            ],
            "Resource": [
                "arn:aws:s3:::<my_bucket>/reputation.snort",
            ]
        },
        {
            "Sid": "Logs",
            "Effect": "Allow",
            "Action": [
                "logs:CreateLogGroup",
                "logs:PutLogEvents"
            ],
            "Resource": "arn:aws:logs:*:*:*"
        }
    ]
}
```
**Note**: you will want to change the variable <my_bucket>

### The Lambda Code
Below is our code to downloads and carve the IPs from Alientvault’s reputational feed (updated every 24 hours):
```
import time
import os
print('Loading function')
""" Function to define Lambda Handler """
def lambda_handler(event, context):
url ="https://reputation.alienvault.com/reputation.snort"
      urllib.request.urlretrieve(url, '/tmp/reputation.snort')
      
      s3 = boto3.resource('s3')
      os.system("sed -i 1,8d /tmp/reputation.snort")
      os.system("cat /tmp/reputation.snort|cut -f 1 -d ' ' > /tmp/snort")
      test=open('/tmp/snort').read().encode('utf8')
      
      s3.Bucket("<my_bucket>").put_object(Key="reputation.snort", Body=test)
```
### Setting up a Scheduled Lambda Event
Hint: Use Cloudwatch

We used the following Cron expression to run every night at midnight:

![](/assets/lambda_event_1.png)

And linked it to our Lamba script:

![](/assets/lambda_event_2.png)

### Add the Threatlist to GuardDuty

![](/assets/guardduty_threat_1.png)

done

![](/assets/guardduty_threat_2.png)

### And Finished…
Now our OTX reputation feed will continually rotate every night.

This walkthrough also serves as an example of serverless architecture, and how AWS can handy those small fiddly scripts without the need for traditional server models.

## References
* https://hackernoon.com/setup-aws-lambda-with-scheduled-events-2840824ed1ad
