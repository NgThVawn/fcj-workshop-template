---
title: "Workshop"
date: 2024-01-01
weight: 5
chapter: false
pre: " <b> 5. </b> "
---

# Deploying the SportBooking Football Pitch Booking Website on AWS

### [Website Feature Demo Video](<https://drive.google.com/file/d/1-0sFCHzJM-VBLlGwRa8S51IZn340tMVN/view?usp=drive_link>)

#### Report Scope

This report presents the deployment of **SportBookingSystem** on AWS using a basic production architecture. The website is published through an HTTPS domain; traffic passes through Route 53 and an Application Load Balancer; the Spring Boot application runs on EC2 instances in private subnets; business data is stored in Amazon RDS for MySQL; release artifacts and uploaded images are stored in Amazon S3; and sensitive configuration is managed in AWS Secrets Manager.

The content follows the order in which resources were provisioned: prepare the source code; build the network and security layers; create S3 buckets and upload the artifact; create RDS; store configuration in Secrets Manager; grant IAM permissions; deploy EC2, the ALB, and the Auto Scaling Group; configure DNS and HTTPS; establish monitoring; and finally remove the resources.

{{% notice warning %}}
The evidence images do not expose database passwords, OAuth client secrets, tokens, access keys, or secret values. The report retains only resource names, Region, status, ports, Security Group sources, and non-sensitive configuration.
{{% /notice %}}

#### Deployment Architecture

| Component | Role |
|---|---|
| Route 53 | Manages DNS for `sport.younglilliu.id.vn` and points an Alias record to the ALB. |
| AWS Certificate Manager | Issues the TLS certificate used to enable HTTPS on the ALB. |
| Application Load Balancer | Accepts public HTTP/HTTPS traffic, redirects HTTP to HTTPS, and forwards requests to the Target Group. |
| Auto Scaling Group + EC2 | Runs the Spring Boot JAR in private application subnets and replaces instances during rollouts. |
| Amazon RDS for MySQL | Stores application data in private database subnets. |
| Amazon S3 | Stores the release JAR and images uploaded through the website. |
| Secrets Manager | Stores the database URL, database credentials, domain, S3 configuration, and OAuth configuration. |
| IAM Role + SSM | Allows EC2 to read S3 and Secrets Manager and be administered through Session Manager without opening SSH. |
| CloudWatch + SNS | Monitors operational status and sends notifications when metrics or alarms cross configured thresholds. |
| CloudFront and AWS WAF | Components of the target architecture that were evaluated but were not included in the primary request path for this deployment. |

#### Contents

1. [Architecture](5.1-architecture/)
2. [Preparation](5.2-preparation/)
3. [Network and Security](5.3-network/)
4. [Data and Access](5.4-data-access/)
5. [EC2 and Load Balancing](5.5-compute/)
6. [Domain and HTTPS](5.6-domain-https/)
7. [Operations](5.7-operations/)
8. [Cleanup](5.8-resource-cleanup/)
