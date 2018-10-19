---
layout: post
title:  "AWS and VPC Endpoints (PrivateLinks)"
date:   2018-05-01 13:49:33 +0000
tags: [Cloud, AWS, pentest, redteam, blueteam]
---
Endpoints are a new feature of VPCs (Virtual Private Clouds), a VPC endpoint enables you to privately connect your VPC to supported AWS services and VPC endpoint services powered by PrivateLink without requiring an internet gateway, NAT device, VPN connection, or AWS Direct Connect connection.

![](/assets/aws_endpoints.jpeg)

Why is this useful? It means that your network communications no longer have to flow over the public internet to reach the public interfaces of AWS services such as S3, API Gateways; a full list of endpoints can be found below.

For those who face paranoid management types that dislike the use of AWS most powerful features like Cloudwatch and S3 due to their public endpoints (despite the use of SSL). We now have a way to architect a direct and secure data flow!

## Interface Endpoints (Powered by AWS PrivateLink)

An interface endpoint is an elastic network interface with a private IP address that serves as an entry point for traffic destined to a supported service. The following services are supported:
* Amazon CloudWatch Logs
* Amazon EC2 API
* Amazon Kinesis Data Streams
* Amazon SNS
* AWS KMS
* AWS Service Catalog
* AWS Systems Manager
* Elastic Load Balancing API
* Endpoint services hosted by other AWS accounts
* Supported AWS Marketplace partner services
* Gateway Endpoints

A gateway endpoint is a gateway that is a target for a specified route in your route table, used for traffic destined to a supported AWS service. The following AWS services are supported:
* Amazon S3
* DynamoDB

## Creating an Endpoint
To create an endpoint, open up the AWS management console and head over to your VPC panel, and choose Endpoints:

![](/assets/aws_endpoints_2.png)

## Create Endpoint Wizard
Click the big blue button, and you should see the following screen, simply select the AWS services you wish to join through this PrivateLink

![](/assets/aws_endpoints_3.png)

## Routing
Select which routing table and subnets you want routing to the endpoint. In our example we are using the default vpc:

![](/assets/aws_endpoints_4.png)

## Policies
Permitting a FullAccess policy is impractical to security, we advise utilising the PolicyGenerator to configure a secure policy to prevent any possible data leaks, or future misconfiguration or corruption of data/messages. https://awspolicygen.s3.amazonaws.com/policygen.html

An example policy that restricts access to a particular user in our demo account, and to only S3 API calls:
```
{
    "Version": "2012-10-17",
    "Id": "Policy1525026866984",
    "Statement": [
        {
            "Sid": "Stmt1525026861859",
            "Effect": "Allow",
            "Principal": {
                "AWS": "AFIBIMYXS24J9BCCWTTGA"
            },
            "Action": "s3:*",
            "Resource": "arn:aws:s3:::*"
        }
    ]
}
```
## Further Reading
To read up more on Endpoints and PrivateLinks please refer to the AWS documentation available here: 
[https://docs.aws.amazon.com/AmazonVPC/latest/UserGuide/vpc-endpoints.html](https://docs.aws.amazon.com/AmazonVPC/latest/UserGuide/vpc-endpoints.html)

For an Overview over advanced VPC design and Endpoints we found this AWS reInvent:2017 talk very useful: 
[https://www.youtube.com/watch?v=Pj11NFXDbLY](https://www.youtube.com/watch?v=Pj11NFXDbLY)
