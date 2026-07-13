---
title : "Configure Networking and Security Groups"
date : 2024-01-01
weight : 3
chapter : false
pre : " <b> 5.3. </b> "
---

#### Deployment Scope

Build the SportBooking network layer with a VPC, subnets, an Internet Gateway, NAT Gateways, route tables, and Security Groups. Network resources are created before storage, database, and compute resources so later services can use an existing VPC, subnets, and security rules.

#### Contents

- [Create the VPC, Subnets, and Route Tables](5.3.1-create-gwe/)
- [Configure Security Groups](5.3.2-test-gwe/)

#### Network Design

| Component | Configuration |
|---|---|
| VPC | `sportbooking-vpc`, CIDR `10.20.0.0/16`. |
| Availability Zones | Two AZs distribute the ALB, EC2, and RDS resources. |
| Public subnets | Two public subnets for the ALB and NAT Gateways. |
| Private application subnets | Two private subnets for EC2 application instances in the Auto Scaling Group. |
| Private database subnets | Two private subnets for the RDS DB subnet group. |
| NAT Gateways | Two NAT Gateways, one in each Availability Zone. |

Recorded result: the ALB is the only public component that receives Internet requests; neither the EC2 application nor RDS is public.

