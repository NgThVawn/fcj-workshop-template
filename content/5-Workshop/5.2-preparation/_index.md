---
title : "Preparation"
date : 2024-01-01
weight : 2
chapter : false
pre : " <b> 5.2. </b> "
aliases:
  - /5-workshop/5.2-prerequiste/
---

#### Prepared Components

Before provisioning AWS resources, verify the source code, deployment tools, access permissions, and parameters used throughout the project. At this stage, only confirm that the application builds successfully; do not upload the artifact because the S3 bucket has not yet been created.

#### Prerequisites

| Item | Requirement |
|---|---|
| AWS Region | Use `us-east-1` consistently. |
| AWS CLI | Configure credentials with permissions for VPC, EC2, S3, IAM, Secrets Manager, RDS, ALB, ACM, Route 53, CloudWatch, and SNS. |
| Source code | The `SportBookingSystem` project builds successfully with Maven Wrapper. |
| Domain | Administrative access is available for `younglilliu.id.vn` to change name servers and create DNS records. |
| Test tools | PowerShell, a browser, `curl.exe`, and `nslookup`; Session Manager is used after EC2 has been provisioned. |

{{% notice info %}}
All resources in this report are deployed in `us-east-1`. Using one Region consistently reduces endpoint, ARN, and certificate errors when HTTPS is configured on the ALB.
{{% /notice %}}

#### Verify That the Application Builds

Open Windows PowerShell in the source-code directory and run:

```powershell
cd D:\22DTHD5\J2EE\SportBookingSystem
.\mvnw.cmd clean package -DskipTests
dir target\*.jar
```

Recorded results:

- The Maven command completed successfully.
- The `target` directory contains `SportBooingSystem-0.0.1-SNAPSHOT.jar`.
- No compilation or missing-dependency errors were reported.

{{% notice tip %}}
This build validates the source code before infrastructure is created. The release JAR is built again and uploaded immediately after the artifact bucket is created in section `5.4.1`; the upload operation does not depend on the EC2 IAM Role.
{{% /notice %}}

#### Deployment Parameters

| Parameter | Value |
|---|---|
| Application domain | `https://sport.younglilliu.id.vn` |
| Artifact bucket | `sportbooking-artifacts-3stars-us-east-1` |
| Artifact object | `releases/SportBooingSystem-0.0.1-SNAPSHOT.jar` |
| Upload bucket | `sportbooking-uploads-3stars-us-east-1` |
| Secret ID | `sportbooking/prod/app-env` |
| Database name | `sport_booking` |
| Application port | `8080` |
| Health check path | `/actuator/health` |

{{% notice warning %}}
Do not store access keys, database passwords, OAuth client secrets, or tokens directly in the source code, JAR, or report images. Sensitive values are stored in AWS Secrets Manager in section `5.4.3`.
{{% /notice %}}
