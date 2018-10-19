---
layout: post
title:  "Adding Security Headers to S3 Websites"
date:   2018-05-18 13:49:33 +0000
tags: [Cloud, AWS, pentest, redteam, blueteam]
---
![](/assets/s3.png)

Having recently converted to Amazon Web Services, one of the frustrations was adding missing security headers. If you want to learn more about security headers we recommend visiting Scott Helme’s website: http://securityheaders.com

Adding security headers was a frustrating process and took us a few attempts; Many iterations resulted in 502/503 Errors from Cloudfront. In the end this blog post (https://iangilham.com/2017/08/22/add-headers-with-lambda-edge.html) helped us to identify some of the mistakes and in the end we were successful!

These are the steps we followed to achieve our goal:
* Login into the AWS Console
* Switch Region to N.Virginia (as this has the capability to create Cloudfront Edge Triggers)
* Open Lambda
Create New Lambda Node 6.10 from Scratch, and give it a name (we used s3_edge_headers)

![](/assets/lambda_1.png)

Add the following Javascript code to [$LATEST]
```
‘use strict’;
// Code sourced from https://iangilham.com/2017/08/22/add-headers-with-lambda-edge.html
 exports.handler = (event, context, callback) => {
 function add(h, k, v) {
     h[k.toLowerCase()] = [
         {
             key: k,
             value: v
         }
     ];
 }
    
const response = event.Records[0].cf.response;
const headers = response.headers;
// hsts header at 2 yrs 63072000;
//                1 yr 31536000;
// Strict-Transport-Security: max-age=63072000; includeSubDomains;
add(headers, “Strict-Transport-Security”, “max-age=31536000; includeSubDomains; preload”);
// Reduce XSS risks
add(headers, “X-Content-Type-Options”, “nosniff”);
add(headers, “X-XSS-Protection”, “1; mode=block”);
add(headers, “X-Frame-Options”, “DENY”);
add(headers, “Referrer-Policy”, “no-referrer-when-downgrade”);
// TODO: fill in value of the sha256 hash
const csp = “default-src ‘none’” +
“; frame-ancestors ‘none’” +
“; base-uri ‘none’” +
“; style-src ‘self’ ‘unsafe-inline’” +
“; img-src ‘self’ https:” +
“; script-src ‘strict-dynamic’ ‘sha256-my_script_hash’ ‘unsafe-inline’ https:”
//add(headers, “Content-Security-Policy”, csp);
console.log(‘Response headers added’);
callback(null, response);
};
```
6. Create a New IAM Role ‘Lamba_Basic_Execute’ and attach the following role:
<pre>
AWSLambdaBasicExecutionRole
</pre>
7. Next, modify the ‘Trust Relationship”, use the following inline policy:
```
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "Service": [
          "edgelambda.amazonaws.com",
          "lambda.amazonaws.com"
        ]
      },
      "Action": "sts:AssumeRole"
    }
  ]
}
```
8. Go back to the Lambda Function. Publish the New Function (as Aliases and $LATEST arent used by Cloudfront)

9. Select Cloudfront, and ensure Origin = ORIGIN-RESPONSE

10. Tick the box Enable trigger

![](/assets/lambda_2.png)

11. Click Add, then Save!

12. Check replication in the Cloudfront console

![](/assets/cloudfront_1.png)

13. Finally, check your website’s headers, and navigate your pages to ensure there are no errors!

![](/assets/website.png)

![](/assets/secheaders.png)

## Final word
We hope this walkthrough is useful to other AWS developers, we have tried to document each step-by-step process so that you to can easily add security headers to your static websites hosted within S3 buckets.
