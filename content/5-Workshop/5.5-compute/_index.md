---
title : "EC2 and Load Balancing"
date : 2024-01-01
weight : 5
chapter : false
pre : " <b> 5.5. </b> "
aliases:
  - /5-workshop/5.5-policy/
---

#### Deployment Scope

Deploy the compute layer only after S3 contains the artifact and the IAM Role, RDS, and Secrets Manager resources are ready. The sequence is: create a seed EC2 instance to verify the runtime and permissions; create an AMI and Launch Template; create the Target Group and Application Load Balancer; and finally create the Auto Scaling Group in private application subnets.

{{% notice info %}}
Create the Auto Scaling Group after the Target Group and ALB because the ASG is associated with the Target Group during configuration. This order automatically registers and health-checks new EC2 instances as they launch.
{{% /notice %}}

#### Contents

1. [Seed EC2 Instance](5.5.1-ec2-seed/)
2. [Launch Template](5.5.2-launch-template/)
3. [Target Group and ALB](5.5.3-alb/)
4. [Auto Scaling Group](5.5.4-auto-scaling/)

{{% notice info %}}
The pages follow resource dependencies. The seed EC2 instance validates the runtime and access permissions before the AMI is created; the Launch Template is completed before the Auto Scaling Group; and the Target Group and ALB must be ready before the ASG registers instances.
{{% /notice %}}
