---
title : "Cleanup"
date : 2024-01-01
weight : 8
chapter : false
pre : " <b> 5.8. </b> "
aliases:
  - /5-workshop/5.8-cleanup/
---

#### Cleanup Scope

After deployment, testing, and evidence collection were completed, the SportBooking resources were decommissioned in dependency order. The artifact bucket and the compute and network layers were removed. The upload bucket, final RDS snapshot, and retained automated backup were intentionally preserved to protect data required for recovery.

{{% notice warning %}}
Deleting the Auto Scaling Group, RDS database, S3 bucket, VPC, and IAM role can cause data loss or make the website completely unavailable. Perform these actions only after the required data has been backed up, the report has been completed, and the environment is confirmed to be no longer in use.
{{% /notice %}}

#### Cleanup Order

| Phase | Resources | Dependency |
|---|---|---|
| 1 | Route 53, CloudWatch, and SNS | Stop directing users to the application; delete the alarm before the SNS topic. |
| 2 | ASG, ALB, Target Group, and Launch Template | Delete the ASG before the Launch Template; delete the ALB before the Target Group. |
| 3 | IAM, RDS, S3, Secrets Manager, and ACM | EC2 has stopped; ACM is no longer used by the ALB; no CloudFront distribution was created. |
| 4 | NAT Gateway, Elastic IP, and VPC Endpoint | Wait until each NAT Gateway reaches `Deleted` before releasing its Elastic IP. |
| 5 | Security Group, DB Subnet Group, and VPC | The ALB, EC2 instances, RDS database, endpoint, and Security Group references have been removed. |
| 6 | Remaining-resource review | No snapshot, backup, log group, hosted zone, or policy remains outside the retention plan. |

{{% notice info %}}
Verify the correct AWS account and Region `us-east-1` before every action. Select only resources named for the SportBooking project; do not delete default or shared resources.
{{% /notice %}}

#### Preparation Before Deletion

1. Identify the final RDS snapshot, retained automated backup, and data in the upload bucket that must be preserved.
2. Stop the Alias record `sport.younglilliu.id.vn` from routing to the ALB before deleting the load-balancing layer.
3. Confirm that no CloudFront distribution was created, so no CloudFront dependency needs to be removed.
4. Retain the public hosted zone because the domain remains managed; remove only records that are no longer used.
5. Record every retained storage resource so that it is not mistaken for an overlooked resource.

#### Contents

1. [Monitoring](5.8.1-monitoring/)
2. [Application and Load Balancing](5.8.2-application/)
3. [Data and Configuration](5.8.3-data/)
4. [NAT Gateways and VPC Endpoint](5.8.4-nat-endpoint/)
5. [Security Groups and VPC](5.8.5-security-vpc/)
6. [Final Verification](5.8.6-verification/)
