---
title : "Architecture Overview"
date : 2024-01-01
weight : 1
chapter : false
pre : " <b> 5.1. </b> "
---

#### Project Architecture

The SportBooking target architecture follows a multi-tier AWS design that clearly separates the access, application, data, and monitoring layers. Two Availability Zones are used to improve availability for EC2 and RDS.

![Target architecture of the SportBooking system on AWS](/images/5-Workshop/5.1-Workshop-overview/sportbooking-architecture.png)

<p class="image-caption"><em>Figure 1: Target architecture of the SportBooking system on AWS</em></p>

#### Request Flow in the Diagram

1. Users send requests to the `sport.younglilliu.id.vn` domain.
2. The DNS and edge layer receives requests before traffic enters the Application Load Balancer. In the implemented scope, Route 53 points directly to the ALB; CloudFront and AWS WAF remain target-architecture extensions.
3. The static S3 branch in the diagram represents a future option to separate the frontend as a static website. The current deployment uses S3 for release artifacts and application uploads.
4. The ALB forwards requests to the Target Group on port `8080` and sends traffic only to targets that pass the health check.
5. The Auto Scaling Group maintains EC2 instances in private application subnets across two Availability Zones.
6. The application connects to Amazon RDS for MySQL Multi-AZ through a single endpoint; AWS manages the standby instance, which does not serve direct queries.
7. EC2 accesses S3 through an S3 Gateway VPC Endpoint to retrieve artifacts and read or write images without routing S3 traffic through the Internet.
8. CloudWatch collects operational metrics; alarms publish events to SNS, and SNS sends notifications to a confirmed email endpoint.

#### Recorded Deployment Results

- The website is accessible at `https://sport.younglilliu.id.vn`.
- EC2 runs the Spring Boot application in private subnets without public IP addresses.
- Amazon RDS for MySQL Multi-AZ resides in private database subnets and accepts connections only from the EC2 application layer.
- The release JAR and uploaded images are stored in S3 without making either bucket public.
- Sensitive configuration is read from Secrets Manager through an IAM Role.
- The ALB terminates HTTPS, redirects HTTP to HTTPS, and uses the `/actuator/health` health check.

{{% notice info %}}
The diagram shows the complete target architecture. CloudFront, AWS WAF, and the static S3 frontend were not part of the tested request path in this report; the actual path was Route 53 -> ALB -> Target Group -> EC2. IAM Role, Secrets Manager, and ACM were implemented but omitted from the diagram to keep the primary flow easy to follow.
{{% /notice %}}

#### Resource Information

| Resource | Deployment value |
|---|---|
| Region | `us-east-1` |
| Test deployment domain | `https://sport.younglilliu.id.vn` |
| ALB DNS | `sportbooking-alb-1198360694.us-east-1.elb.amazonaws.com` |
| Artifact bucket | `sportbooking-artifacts-3stars-us-east-1` |
| Artifact object | `releases/SportBooingSystem-0.0.1-SNAPSHOT.jar` |
| Upload bucket | `sportbooking-uploads-3stars-us-east-1` |
| Secret ID | `sportbooking/prod/app-env` |
| RDS endpoint | `sportbooking-mysql.cudgu8qk8z19.us-east-1.rds.amazonaws.com` |
| Database | `sport_booking` |
| Application port | `8080` |
| Health check path | `/actuator/health` |

#### AWS Services Used

| Service | Purpose |
|---|---|
| Amazon VPC | Separates public, private application, and private database subnets. |
| NAT Gateway | Provides outbound Internet access for private EC2 instances when packages or AWS services without an endpoint must be reached. |
| Security Group | Restricts traffic according to the principle of least privilege. |
| IAM Role | Allows EC2 to call AWS APIs without storing access keys on the instance. |
| Amazon S3 | Stores the release JAR and website uploads. |
| Secrets Manager | Keeps sensitive configuration out of the source code and JAR. |
| Amazon RDS for MySQL | Provides a managed database with backups and snapshots. |
| EC2 Launch Template | Defines the AMI, role, Security Group, and User Data bootstrap. |
| Auto Scaling Group | Maintains EC2 capacity and supports Instance Refresh when a new JAR is deployed. |
| Target Group | Health-checks EC2 application instances before they receive traffic. |
| Application Load Balancer | Accepts public requests, terminates TLS, and forwards traffic to the application. |
| Route 53 + ACM | Configures the domain and HTTPS. |
| Systems Manager Session Manager | Provides shell access to private EC2 instances without opening SSH. |
| Amazon CloudWatch | Observes metrics and logs and creates alarms for the system. |
| Amazon SNS | Sends notifications when a CloudWatch alarm changes state. |
| Amazon CloudFront + AWS WAF | Evaluated architecture extensions that were not enabled in the primary request path. |

