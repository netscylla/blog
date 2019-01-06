---
layout: post
title:  "AWS: Linking EC2 logs to Cloudwatch"
date:   2018-03-13 13:49:33 +0000
tags: [Cloud, AWS, pentest, redteam, blueteam]
---
Quick and simple post on playing around with Cloudwatch on AWS EC2.

![](/blog/assets/cloudwatch.png)

## Install an IAM role to govern the transfer of the logs
Then go to the EC2 dashboard https://console.aws.amazon.com/iam

Choose ‘Create new IAM role’, then

### Create Role
EC2 Instance, ignore all other settings, click next
### Create policy
Paste in the following JSON policy:
<pre>
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "logs:CreateLogGroup",
        "logs:CreateLogStream",
        "logs:PutLogEvents",
        "logs:DescribeLogStreams"
    ],
      "Resource": [
        "arn:aws:logs:*:*:*"
    ]
  }
 ]
}
</pre>
Finally, name the role for ease of identification later.
## Install AWSLogs
SSH into your EC2 instance and issue the following commands:
<pre>
sudo yum update -y
sudo yum install -y awslogs
</pre>
The latest version of Redhat uses systemctl as opposed to legacy init.d services:
<pre>
systemctl start awslogsd
sudo systemctl enable awslogsd.service
</pre>
on older versions of the ami the traditional commands are:
<pre>
/etc/init.d/awslogs start
chkconfig -add awslogs
chkconfig awslogs on
</pre>
## Configuring Logging
By default only /var/log/messages is logged, but adding extra logs is dead easy, simply edit the following file /etc/awslogs/awslogs.conf:
<pre>
[/var/log/messages]
datetime_format = %b %d %H:%M:%S
file = /var/log/messages
buffer_duration = 5000
log_stream_name = {instance_id}
initial_position = start_of_file
log_group_name = /var/log/messages
[/var/log/secure]
datetime_format = %b %d %H:%M:%S
file = /var/log/secure
buffer_duration = 5000
log_stream_name = {instance_id}
initial_position = start_of_file
log_group_name = ssh
</pre>
Also the default location for awslogs is us-east-1, you may want to change this? Edit the file /etc/awslogs/awscli.conf:
<pre>
[plugins]
cwlogs = cwlogs
[default]
region = us-east-1
Cloudwatch
</pre>
Hopefully, you should then be collecting logs from the EC2 instance(s) that have the role applied.

![](/blog/assets/cloudwatch_log.png)

Then the fun begins, playing with filter, metrics, alarms, and Lambda rules to shift the data into an elastic search platform.

