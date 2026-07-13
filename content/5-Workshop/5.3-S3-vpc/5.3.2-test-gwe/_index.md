---
title : "Configure Security Groups"
date : 2024-01-01
weight : 2
chapter : false
pre : " <b> 5.3.2 </b> "
---

#### Deployment Configuration

The three Security Groups `sportbooking-alb`, `sportbooking-app`, and `sportbooking-rds` follow the access chain: users reach the ALB; the ALB forwards traffic to the EC2 application; and the EC2 application connects to Amazon RDS for MySQL.

#### ALB Security Group

| Direction | Type | Port | Source/Destination | Role |
|---|---|---|---|---|
| Inbound | HTTP | 80 | `0.0.0.0/0` | Allows HTTP access and redirects it to HTTPS. |
| Inbound | HTTPS | 443 | `0.0.0.0/0` | Allows HTTPS access. |
| Outbound | Custom TCP | 8080 | EC2 App SG | Allows the ALB to forward traffic to the application. |

Recorded result: the ALB Security Group does not expose ports `8080`, `3306`, or `22` publicly.

#### EC2 App Security Group

| Direction | Type | Port | Source/Destination | Role |
|---|---|---|---|---|
| Inbound | Custom TCP | 8080 | ALB Security Group | Allows only the ALB to call the application. |
| Inbound | SSH | Not open | Not used | Administration uses Session Manager instead of SSH. |
| Outbound | HTTPS/All traffic | 443/All | Internet through NAT or AWS services | EC2 must reach S3, Secrets Manager, SSM, and RDS. |

Recorded result: no `0.0.0.0/0` rule allows inbound access to port `8080`.

#### RDS Security Group

| Direction/Setting | Type | Port | Source | Role |
|---|---|---|---|---|
| Inbound | MySQL/Aurora | 3306 | EC2 App Security Group | Allows only the EC2 application to connect to the database. |
| Public access | No | - | - | Prevents public Internet access to RDS. |

Recorded result: RDS accepts connections only from the EC2 App Security Group, not from a public IP address or `0.0.0.0/0`.

#### Post-configuration Verification

- Record the inbound rules of the ALB, EC2 App, and RDS Security Groups.
- Verify that the source of the EC2 rule is the ALB Security Group, not a public CIDR.
- Verify that the source of the RDS rule is the EC2 App Security Group.
- Do not open public SSH; administer EC2 through Session Manager after attaching the IAM Role.

#### Procedure

Create the Security Groups in ALB, EC2 application, and RDS order so every rule references a component that already exists.

**Procedure:**

1. **Open the Security Groups list**

   Open **EC2 > Security Groups**, then choose **Create security group**.

   ![Open the Security Groups list](/images/5-Workshop/5.3-S3-vpc/security-groups-01.png)

   <p class="image-caption"><em>Figure 1: Open the Security Groups list</em></p>
   A Security Group is a resource-level firewall. Separate Security Groups for the ALB, EC2 application, and RDS support least-privilege access.


2. **Configure the ALB Security Group**

   Enter `sportbooking-alb` as the name and describe the ALB role. Select `sportbooking-vpc`, then add inbound HTTP port `80` and HTTPS port `443` rules from `0.0.0.0/0`.

   ![Configure the ALB Security Group](/images/5-Workshop/5.3-S3-vpc/security-groups-02.png)

   <p class="image-caption"><em>Figure 2: Configure inbound rules for the ALB Security Group</em></p>
   Public rules are added only at the ALB layer. Application port `8080` and database port `3306` are not exposed to the Internet.


3. **Create the ALB Security Group**

   Review the outbound rules and Security Group identifiers. Add resource-management tags, then choose **Create security group**.

   ![Complete the ALB Security Group configuration](/images/5-Workshop/5.3-S3-vpc/security-groups-03.png)

   <p class="image-caption"><em>Figure 3: Review and create the ALB Security Group</em></p>
   Verify that the Security Group belongs to the correct VPC and contains only the required rules before creating it.


4. **Confirm that the Security Group was created**

   Open the new `sportbooking-alb` Security Group and inspect the **Inbound rules** tab and VPC information.

   ![Confirm that the Security Group was created](/images/5-Workshop/5.3-S3-vpc/security-groups-04.png)

   <p class="image-caption"><em>Figure 4: Confirm that the sportbooking-alb Security Group was created</em></p>
   The resource details confirm that `sportbooking-alb` belongs to the correct VPC and has the two HTTP and HTTPS inbound rules.


5. **Restrict outbound traffic from the ALB to the application**

   Open **Outbound rules**, set the protocol to **Custom TCP** and the port to `8080`. Select `sportbooking-app` as the destination Security Group, then choose **Save rules**.

   ![Restrict outbound traffic from the ALB to the application](/images/5-Workshop/5.3-S3-vpc/security-groups-05.png)

   <p class="image-caption"><em>Figure 5: Restrict ALB outbound traffic to port 8080 on sportbooking-app</em></p>
   The ALB can now forward application traffic only to EC2 instances associated with `sportbooking-app` on port `8080`.

Repeat the **Create security group** workflow for the remaining two tiers. `sportbooking-app` accepts inbound port `8080` only from `sportbooking-alb`; `sportbooking-rds` accepts inbound MySQL port `3306` only from `sportbooking-app`. These three Security Groups are then assigned to the ALB, Launch Template, and RDS respectively.

