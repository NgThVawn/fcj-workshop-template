---
title : "Domain and HTTPS"
date : 2024-01-01
weight : 6
chapter : false
pre : " <b> 5.6. </b> "
aliases:
  - /5-workshop/5.6-cleanup/
---

#### Deployment Scope

Configure the `sport.younglilliu.id.vn` domain, issue a certificate with ACM, attach an HTTPS listener to the ALB, and test the deployed website.

#### Contents

1. [Route 53](5.6.1-route53/)
2. [ACM and HTTPS](5.6.2-acm-https/)
3. [Access Testing](5.6.3-testing/)
4. [CloudFront Evaluation](5.6.4-cloudfront/)

{{% notice info %}}
The Hosted Zone and Alias record are completed before the certificate is requested. The HTTPS listener is created only after ACM validates the domain, and DNS and HTTPS testing is performed last.
{{% /notice %}}
