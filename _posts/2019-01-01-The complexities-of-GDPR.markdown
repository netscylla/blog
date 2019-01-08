---
layout: post
title:  "The complexities of understanding GDPR"
description: "Its now 2019 and people still don't understand GDPR. Are you aware of Article 25 and security by design!"
date:   2019-01-01 01:49:33 +0000
tags: [consultancy, GDPR, Data Privacy,]
---
# The complexities of understanding GDPR

![GDPR](/assets/GDPR.jpg)

## TL;DR
Last year (2018) we saw many mistakes by organisations and contractors that exposed a wealth of Personal Identifiable Information (PII)
through insecure bucket configurations. Cloud vendors (AWS, Google and Azure) have gone to great lengths to alert developers and
administrators about open and public buckets; But similar breaches keep on happening.

In this post we highlight the relatively unknown (or confused topic) **Article 25 - Security by design**!

## Quick Background on GDPR
GDPR is a EU enhancement to Data Privacy Directives (DPD) and trumps the British Data Protection Act 1998 (DPA). GDPR was driven by the European Union due to inadequacies in the way some member states manage data and data privacy.

Important changes between GDPR and DPA have been highlighted in mainstram media include:
* classifies personal data into new categories: personal, sensitive, biometric
* forces greater security controls and pseudomisation on sensitive data
* applies to all EU citizens, so affects third party countries that store personal data on EU citizens
* sanctions will be imposed for failed compliance. Between 10-20milllion euros or 2-4% of the previous years global turnover.
* breaches must be reported within the first 72 hours.

But we have already seen many incidents involving leaky S3 buckets where masses of personal data such as driving licenses, 
passports and other personal identifiable information have been easily publicly accessible.  Now cloud providers have made many 
changes to their platforms to highlight the security risk of public buckets, but why does this keep happening?

### 2018 Leaks
* [Fedex S3 Leak](https://www.theregister.co.uk/2018/02/15/fedex_aws_s3_leak/)
* [Medcall S3 Leak](https://www.healthcareitnews.com/news/update-misconfigured-database-breaches-thousands-medcall-advisors-patient-files)
* [GoDaddy S3 Leak](https://threatpost.com/godaddy-leaks-map-of-the-internet-via-amazon-s3-cloud-bucket-misconfig/135009/)
* [History of S3 Leaks](https://github.com/nagwww/s3-leaks)

## What has happened in the last few months
GDPR came into place on 25th May 2018. Companies and organisations were given fair warning and at least two years to prepare for GDPR compliance.
Like with any other deadline, it might have been misunderstood, ill-planned, or consultants took the opportunity to prey on the confused and mis-sell GDPR consultancy.
Certainly between December 2017 and May 2018 there were a number of GDPR ambulance chasers, and we were severely unimpressed with some of the quality of work
and minimal knowledge these 'so-called experts' were peddling.

Netscylla on several occasions has been employed by a handful of our clients to re-work a number of IT projects for small and medium businesses due mistakes made by Data Protection Cowboys.

## Article 25
Many Data Protection Officers/Analysts are forgetting about Article 25, so what is Article 25 and why is it important?

Article 25(1) specifies the requirements for data protection by design:

```
Taking into account the state of the art, the cost of implementation and the nature, 
scope, context and purposes of processing as well as the risks of varying likelihood 
and severity for rights and freedoms of natural persons posed by the processing, the 
controller shall, both at the time of the  determination of the means for processing 
and at the time of the processing itself, implement appropriate technical and 
organisational measures, such as pseudonymisation, which are designed to implement 
data-protection principles, such as data minimisation, in an effective manner and to 
integrate the necessary safeguards into the processing in order to meet the 
requirements of this Regulation and protect the rights of data subjects.’
```
Article 25(2) specifies the requirements for data protection by default:

```
The controller shall implement appropriate technical and organisational measures for 
ensuring that, by default, only personal data which are necessary for each specific 
purpose of the processing are processed. That obligation applies to the amount of 
personal data collected, the extent of their processing, the period of their storage 
and their accessibility. In particular, such measures shall ensure that by default 
personal data are not made accessible without the individual's intervention to an 
indefinite number of natural persons.’
```
Reference
* [ICO guide to data protection](https://ico.org.uk/for-organisations/guide-to-data-protection/guide-to-the-general-data-protection-regulation-gdpr/accountability-and-governance/data-protection-by-design-and-default/)

### What is data protection by design?
Data protection by design is ultimately an approach that ensures you consider privacy and data protection issues at the design phase of any system, service, product or process and then throughout the lifecycle.

As expressed by the GDPR, it requires you to:

 * put in place appropriate technical and organisational measures designed to implement the data protection principles; and
 * integrate safeguards into your processing so that you meet the GDPR's requirements and protect the individual rights.
In essence this means you have to integrate or ‘bake in’ data protection into your processing activities and business practices.

Data protection by design has broad application. Examples include:
 * developing new IT systems, services, products and processes that involve processing personal data;
 * developing organisational policies, processes, business practices and/or strategies that have privacy implications;
 * physical design;
 * embarking on data sharing initiatives; or
 * using personal data for new purposes.
The underlying concepts of data protection by design are not new. Under the name ‘privacy by design’ they have existed for many years. Data protection by design essentially inserts the privacy by design approach into data protection law.

Under the 1998 Act, the ICO supported this approach as it helped you to comply with your data protection obligations. It is now a legal requirement.

### What is data protection by default?
Data protection by default requires you to ensure that you only process the data that is necessary to achieve your specific purpose. It links to the fundamental data protection principles of data minimisation and purpose limitation.

You have to process some personal data to achieve your purpose(s). Data protection by default means you need to specify this data before the processing starts, appropriately inform individuals and only process the data you need for your purpose. 
What you need to do depends on the circumstances of your processing and the risks posed to individuals.

Nevertheless, you must consider things like (We've bolded the line, which centers around the argument on the LinkedIn post that many DPO missed):
 * adopting a ‘privacy-first’ approach with any default settings of systems and applications;
 * ensuring you do not provide an illusory choice to individuals relating to the data you will process;
 * not processing additional data unless the individual decides you can;
 * **ensuring that personal data is not automatically made publicly available to others unless the individual decides to make it so;** and
 * providing individuals with sufficient controls and options to exercise their rights.

### Who is responsible for complying with data protection by design and by default?
**Article 25 specifies that, as the controller, you have responsibility for complying with data protection by design and by default**. 
Depending on your circumstances, you may have different requirements for different areas within your organisation. For example:
 * your senior management. eg developing a culture of ‘privacy awareness’ and ensuring you develop policies and procedures with data protection in mind;
 * your software engineers, system architects and application developers. eg those who design systems, products and services should take account of data protection requirements and assist you in complying with your obligations; and
 * your business practices. eg you should ensure that you embed data protection by design in all your internal processes and procedures.
 
## What could have the financial institute done differently.
 
 * Disable open/public access to the offending S3 bucket
 * Encrypted the objects inside the S3 bucket
 * Periodic audits, if they periodically audited their security posture and cloud accounts surely they would have been aware of this security risk sooner.
 * * In AWS **Trusted Advisor** is brilliant at auditing this vulnerability! 
 
## Other GDPR Articles that may effect an organisation
This all comes down whether the organisation correctly notifies data subjects about their lapse in security in the next few weeks?

**Article34 - Do not have to report breaches to data subjects if the data was encrypted.** - We all know S3 buckets support encryption, why did they not utilise atleast the offered builtin encryption? 
 
## References
 I lot of statements and descriptions of the articles from GDPR were extracted from
 * [ICO guide to data protection](https://ico.org.uk/for-organisations/guide-to-data-protection/guide-to-the-general-data-protection-regulation-gdpr/accountability-and-governance/data-protection-by-design-and-default/)

 