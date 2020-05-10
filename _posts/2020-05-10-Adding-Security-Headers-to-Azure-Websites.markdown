---
layout: post
title:  "Adding Security Headers to Azure Websites"
date:   2020-05-18 08:49:33 +0000
tags: [Cloud, AWS,Azure, pentest, redteam, blueteam]
---
![](/blog/assets/azure-storage.png)

After the success of adding [Security Headers to our AWS S3 websites](https://www.netscylla.com/blog/2018/05/18/adding-security-headers-to-s3-websites.html) (nearly 2 years ago), Netscylla have since had customers migritate to the Azure cloud stack.  Our customers asked if we could do the same thing, on Azure Storage  blobs.  Quickly we came up with a solution to the problem.  For our readers that are not familar with AWS or Azure, below is how AWS and Azure have similar services:

 * AWS S3 <-> Azure Storage Blob
 * AWS Lambda <-> Azure Functions
 * AWS Cloudfront <-> Azure CDN 

We've come up with this basic JSON template to get you started in Azure Functions:
```
{
    "$schema": "http://json.schemastore.org/proxies",
    "proxies": {
        "index": {
            "matchCondition": {
                "route": "/",
                "methods": [
                    "GET",
                    "HEAD"
                ]
            },
            "backendUri": "https://%STORAGEACCOUNTADDRESS%/Index.html?%STORAGEACCOUNTSAS%",
            "responseOverrides": {
                "response.headers.strict-transport-security": "max-age=31536000; includeSubDomains",
                "response.headers.X-Powered-By": "redacted",
                "response.headers.X-Content-Type-Options": "nosniff",
                "response.headers.X-XSS-Protection": "1; mode=block",
                "response.headers.x-ms-blob-type": "redacted",
                "response.headers.x-ms-lease-state": "redacted",
                "response.headers.x-ms-lease-status": "redacted",
                "response.headers.Server": "redacted",
                "response.headers.x-frame-options": "SAMEORIGIN",
                "response.headers.Content-Security-Policy": "script-src 'self'",
                "response.headers.Upgrade-Insecure-Requests": "1",
                "response.headers.Referrer-Policy": "same-origin",
                "response.headers.Feature-Policy": "payment 'self'; geolocation 'self'"
            }
        },
        "otherpages": {
            "matchCondition": {
                "route": "/{page}",
                "methods": [
                    "GET",
                    "HEAD"                    
                ]
            },
            "backendUri": "https://%STORAGEACCOUNTADDRESS%/{page}.html?%STORAGEACCOUNTSAS%",
            "responseOverrides": {
                "response.headers.strict-transport-security": "max-age=31536000; includeSubDomains",
                "response.headers.X-Powered-By": "redacted",
                "response.headers.X-Content-Type-Options": "nosniff",
                "response.headers.X-XSS-Protection": "1; mode=block",
                "response.headers.x-ms-blob-type": "redacted",
                "response.headers.x-ms-lease-state": "redacted",
                "response.headers.x-ms-lease-status": "redacted",
                "response.headers.Server": "redacted",
                "response.headers.x-frame-options": "SAMEORIGIN",
                "response.headers.Content-Security-Policy": "script-src 'self'",
                "response.headers.Upgrade-Insecure-Requests": "1",
                "response.headers.Referrer-Policy": "same-origin",
                "response.headers.Feature-Policy": "payment 'self'; geolocation 'self'"
            }
        }
    }
}
```
## Protecting the Origin
Azure has some differences to AWS because we want to ensure website traffic passes through the Functions Proxy (so the security headers can be added) we do not want to expose the entire storage container publicly! Therefore, we cannot use the Azure static hosting feature. Instead **we create a normal blob storage container with private access** which we then access from the functions proxy. **To allow the functions proxy to access the storage account you need to create a Shared Access Signature (SAS)** which you will use to allow the functions proxy to access the storage.

## Azure Functions Proxy
Azure Functions Proxy allows you to define a route pattern in an Azure Function application which can act as a proxy to another resource, the proxy allows you to override request and response parameters including headers. In this example the proxy allows us to securely access our storage account pages and and add response headers to each response. This is what my proxy looks like once I’ve setup the routing and headers

![Azure Functions Proxy](/blog/assets/azureproxy.png)

### Downsides
Unlike AWS Lambda the Functions Proxy feature does not permit you to remove or alter existing headers!

## Final word
We hope this walkthrough is useful to other Azure developers, we have tried to document each step-by-step process so that you to can easily add security headers to your static websites hosted within Storage Blob buckets.
