---
title : "Data and Access"
date : 2024-01-01
weight : 4
chapter : false
pre : " <b> 5.4. </b> "
aliases:
  - /5-workshop/5.4-s3-onprem/
---

#### Deployment Scope

Provision the foundational services before deploying EC2: create the S3 buckets and upload the artifact, create the S3 Gateway Endpoint, deploy Amazon RDS for MySQL Multi-AZ, create a Secrets Manager secret for application configuration, and then grant the EC2 IAM Role access to the resources that now exist.

#### Implementation Order

1. [Create S3 Buckets, Upload the Artifact, and Configure the Gateway Endpoint](5.4.1-s3/)
2. [Create Private Amazon RDS for MySQL](5.4.2-rds/)
3. [Create the Application Secret in Secrets Manager](5.4.3-secrets/)
4. [Create the EC2 IAM Role](5.4.4-iam/)

{{% notice info %}}
Create the secret after RDS so `DB_URL` contains the correct database endpoint. Create the IAM Role after S3 and the secret so its policy can reference ARNs that already exist. Create the Launch Template and EC2 resources only after the artifact, RDS, secret, and role are all ready.
{{% /notice %}}

#### Recorded Results

- The S3 artifact bucket contains the JAR under the `releases/` prefix and is not public.
- The S3 upload bucket stores football-pitch images and has Block Public Access enabled.
- The S3 Gateway Endpoint is associated with the route tables of the private application subnets.
- The `sportbooking-ec2-role` IAM Role permits SSM access, S3 artifact reads, S3 upload reads and writes, and secret reads.
- Amazon RDS for MySQL is in private database subnets and is not publicly accessible.
- The `sportbooking/prod/app-env` secret contains the configuration required for EC2 to create `/opt/sportbooking/app.env`.
