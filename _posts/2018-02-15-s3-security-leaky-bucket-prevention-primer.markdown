---
layout: post
title:  "S3 Security: Leaky Bucket Prevention Primer"
date:   2018-02-15 13:49:33 +0000
tags: [Cloud, AWS, S3, pentest, redteam, blueteam]
---
![](/blog/assets/leaky_bucket.jpeg)

With yet another news report of another S3 bucket leak, we at Netscylla think its important to highlight some of the known features of how to secure your AWS S3 Buckets.

# Purpose of the Bucket
S3 buckets have multiple uses whether its hosting images and source files for websites (CDN), log files for attribution and non-repudiation. So depending on the use of the bucket, you might want full anonymous access, for website material, authenticated users for public but controlled content attached to your website, or a very restrictive bucket if storing company secrets, personal information or important logging data.

# Bucket Groups/ACLs
With AWS S3 there are 3 simple groups to remember:
* Everyone — which means any public internet user can access your data (This is commonly the default bucket ACL)
* Authenticated Users — which means any public internet user with an AWS account (including the free tier) can access your data.
* Granular IAM Approach — Your using the IAM permissions to strictly control who has access to the bucket. Now the security comes down to the controls and permissions you place on individual accounts. It is now the responsibility of the individual user or developer to ensure their account is not compromised.

# Securing those Bucket ACLs
Points to consider include:
* You should think and plan carefully your use cases before granting Read access to the Everyone group, because this allows anyone to access the bucket or object.
* Never allow Write access to the Everyone group, because this allows anyone to add objects to your bucket (which means you will be billed for the transfer and storage), and it allows anyone to delete objects in the bucket that you might want to keep.
* Be wary of any authenticated AWS user group permissions, which includes anyone with an active AWS account, not just IAM users in your account/organisation. Ideally your permissions and IAM policies should be audited on a regular basis.
* To control access for IAM users on your account, consider using an IAM policy instead. (Covered below)
# Monitoring your S3 Buckets
Amazon has two easily accessible features for monitoring your buckets:
* CloudTrail
* CloudWatch
CloudTrail logs. For more information on how to configure CloudTrail, see Getting Started with CloudTrail. Note: By default, CloudTrail only tracks bucket-level actions. To track object-level actions (such as GetObject), you would need to enable S3 Data Events for each individual bucket you want to log.

The CloudWatch metrics that S3 provides can be found here. There are two of them:
* The NumberOfObjects is the total number of objects in the bucket. The metrics is reported with the dimensions BucketName=X and StorageType=AllStorageTypes.
* The BucketSizeBytes contains the total size in bytes of all objects in the bucket of a certain storage class. The metrics is reported with the dimensions BucketName=X and StorageType as one of StandardStorage, StandardIAStorage or ReducedRedundancyStorage.
* There are also some known issues:
** The metrics are not real-time. Tt is collected and sent to CloudWatch internally by AWS only once a day. The values are reported with Timestamp as midnight UTC of each day.
** It is not clear when the metrics is computed and stored by S3. That is, if it is now 2018–02–15T09:30, the metrics for 2018–02–15 may not be available. It should be available sometime before 2018–02–15T23:59, and when available, will be reported with the timestamp of 2018–02–15T00:00.
** Glacier storage class metrics is not available.

For further examples on Cloudwatch and S3 we recommend reading the following blog post: https://www.opsdash.com/blog/aws-s3-cloudwatch-monitoring.html

# Using encryption to protect your data
AWS offers the following cryptographic services with S3:
* Transport Security
* Encryption at Rest
If your use case requires encryption during transmission, S3 supports the HTTPS protocol, which encrypts data in transit to and from S3.

If your use case requires encryption for data at rest, S3 offers server-side encryption (SSE). You can specify the S3 SSE parameters when you write objects to the bucket; S3 offers SSE-S3, SSE-KMS, or SSE-C.

**Note**: If you use third-party tools to interact with S3, contact the developers to confirm if their tools also support the HTTPS protocol. Also, when you list objects in your bucket, it is important to remember the list API will return a list of all objects, regardless of whether they are encrypted.

Example of secure S3 usage on the command line AWS client:
<pre>
$ awscli s3 ls --endpoint [https://address] --ca-bundle /path/to/certificate
</pre>
# AWS IAM
For those that are unfamiliar with IAM, it stands for Identity and Access Management. AWS IAM allows for very granular controls, where you can apply security policies to individual users, groups, and custom classes of users. This is a large topic, and we recommend you read up on Amazons IAM policies and examples. Below you can find some simplified examples that demonstrate IAM capabilities.

## IAM User Bucket Access
Granting a specific user access to an S3 bucket is relatively straight forward, below is a simple example:
<pre>
{
   "Version":"2012-10-17",
   "Statement":[
      {
         "Effect":"Allow",
         "Action":[
            "s3:ListAllMyBuckets"
         ],
         "Resource":"arn:aws:s3:::*"
      },
      {
         "Effect":"Allow",
         "Action":[
            "s3:ListBucket",
            "s3:GetBucketLocation"
         ],
         "Resource":"arn:aws:s3:::examplebucket"
      },
      {
         "Effect":"Allow",
         "Action":[
            "s3:PutObject",
            "s3:PutObjectAcl",
            "s3:GetObject",
            "s3:GetObjectAcl",
            "s3:DeleteObject"
         ],
         "Resource":"arn:aws:s3:::examplebucket/*"
      }
   ]
}
</pre>
## Restricting Bucket Access Via IP Address
This example is good for organisations, that want to restrict the bucket access to within their corporate range.

The Condition block uses the IpAddress and NotIpAddress conditions and the aws:SourceIp condition key, which is an AWS-wide condition key. For more information about these condition keys, see Specifying Conditions in a Policy. The aws:sourceIp IPv4 values use the standard CIDR notation. For more information, see IP Address Condition Operators in the IAM User Guide.
<pre>
{
  "Version": "2017-12-12",
  "Id": "S3PolicyId1",
  "Statement": [
    {
      "Sid": "IPAllow",
      "Effect": "Allow",
      "Principal": "*",
      "Action": "s3:*",
      "Resource": "arn:aws:s3:::secretsauce/*",
      "Condition": {
         "IpAddress": {"aws:SourceIp": "10.230.145.0/24"},
         "NotIpAddress": {"aws:SourceIp": "10.230.145.1/30"} 
      } 
    } 
  ]
}
</pre>
## Multi Factor Authentication (MFA)
In a bucket policy, you can add a condition to check whether the user has authenticated via their MFA token;aws:MultiFactorAuthAge , as shown in the following example bucket policy. The policy denies any Amazon S3 operation on the /secretreceipe folder in the secretsauce bucket if the request is not MFA authenticated. To learn more about MFA authentication, see Using Multi-Factor Authentication (MFA) in AWS
<pre>
{
    "Version": "2012-10-17",
    "Id": "123",
    "Statement": [
      {
        "Sid": "",
        "Effect": "Deny",
        "Principal": "*",
        "Action": "s3:*",
        "Resource": "arn:aws:s3:::secretsauce/secretreceipe",
        "Condition": { "Null": { "aws:MultiFactorAuthAge": true }}
      }
    ]
 }
</pre>
## More Examples
For more definitive examples on locking down your buckets, we recommend reading: https://docs.aws.amazon.com/AmazonS3/latest/dev/walkthrough1.html

For custom setups you may want to utilise the help of the AWS policy generator http://awspolicygen.s3.amazonaws.com/policygen.html and the Policy Simulation Sandbox https://policysim.aws.amazon.com/home/index.jsp to aid in perfecting your policies and permissions:

![](/blog/assets/iam_sandbox.jpeg)

## IAM Roles
You can use AWS IAM roles to grant permissions for AWS services to call other AWS services on your behalf, or create and manage AWS resources for you in your account. AWS services such as Amazon Lex also offer service-linked roles that are predefined and can be assumed only by that specific service.

IAM users or AWS services can assume a role to obtain temporary security credentials that can be used to make AWS API calls. Consequently, you don’t have to share long-term credentials or define permissions for each entity that requires access to a resource.

# Advantages
* Since role credentials are temporary and rotated automatically.
* Developers don’t have to manage the credentials
* We don’t have to worry about long-term security risks.
* Flexibility to assign single role to multiple EC2 instances where application requires access to S3 storage
* We can change the Role policy any time and the change is propagated automatically to all the instances.

# Caution
* IAM role cannot be assigned to an instance that is already running.
* If we need to add a role to the running instance, We have the only option to create an image of the instance and then launch a new instance from the image with the desired role assigned.
* Never give IAM Roles access to S3:PutObjectAcl, this basically permits the role to change permissions on objects to allow everyone public access
* If you have multiple AWS accounts, you could have a designated AWS account just for S3, where ALL access to S3 is performed via role assumption. This prevents cross-account issues and reduces the need for bucket policies.
* Roles are handy to apply to other EC2 instances, this means that an instance with a given role has temporary credentials, and no passwords or access keys are stored on the instance file-system. Worst case if there is an incident, the instance can be destroyed and re-instantiated with minimal effect. If there is some form of breach? the damage should by limited to the EC2 instance only.

Roles are not generally applied to other accounts, but this is often called cross-account management. An example of cross-account usage can be found here: https://docs.aws.amazon.com/AmazonS3/latest/dev/example-walkthroughs-managing-access-example2.html

# References
For more information please visit:
* https://aws.amazon.com/premiumsupport/knowledge-center/secure-s3-resources/
* https://www.opsdash.com/blog/aws-s3-cloudwatch-monitoring.html
* https://aws.amazon.com/iam/details/manage-roles/

# Media
Man with Bucket — https://upload.wikimedia.org/wikipedia/commons/e/e3/Mission_Accomplished_-_ALS_Ice_Bucket_Challenge_%2814848289439%29.jpg
