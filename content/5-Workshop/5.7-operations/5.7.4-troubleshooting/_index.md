---
title : "Troubleshooting"
date : 2024-01-01
weight : 4
chapter : false
pre : " <b> 5.7.4 </b> "
---

#### Recorded Errors and Resolutions

| Error | Identified cause | Resolution |
|---|---|---|
| JAR upload returns `NoSuchBucket` | The S3 bucket does not exist or the bucket name is incorrect. | Create the artifact bucket first and verify the S3 URI. |
| `Invalid bucket name <artifact-bucket>` | The release command still contains a placeholder. | Replace the placeholder with `sportbooking-artifacts-3stars-us-east-1`. |
| RDS missing table | The database schema does not contain a table required by Hibernate. | Check `ddl-auto` and application logs, then create the required seed data. |
| S3 Region displays literal `${AWS_REGION}` | The Region variable was not passed to the application. | Add `AWS_REGION`, `AWS_DEFAULT_REGION`, and `AWS_S3_REGION=us-east-1`. |
| `/uploads` images return 404 | The S3 controller is disabled, the bucket or prefix is incorrect, or IAM permissions are missing. | Verify `APP_STORAGE_TYPE=s3`, the bucket, `uploads` prefix, and IAM policy. |
| Registration fails on membership levels | The `membership_levels` table has no data. | Insert the `NONE`, `SILVER`, `GOLD`, and `DIAMOND` levels. |
| ACM remains Pending validation | The DNS validation CNAME is not public or the domain is misspelled. | Verify the hosted zone, registrar name servers, and validation CNAME. |
| Instance Refresh version is empty | Desired configuration has no Launch Template version. | Update the ASG to a valid version and use the current ASG configuration. |

#### Well-Architected Framework Review

| Pillar | Application in the architecture |
|---|---|
| Security | Private subnets for EC2 and RDS; least-privilege Security Groups; Secrets Manager; IAM Role instead of access keys; Session Manager without SSH; HTTPS through ACM. |
| Reliability | ALB, ASG, and two AZs; Target Group health checks; RDS backups and snapshots; Instance Refresh replaces instances. |
| Operational Excellence | User Data automates bootstrap; S3 stores deployment artifacts; `curl` and `systemctl` provide verification; CloudWatch and SNS support operations. |
| Cost Optimization | ASG desired capacity is reduced to `0` before deletion; RDS and NAT Gateways are removed after testing; resource sizes are recorded for cost control. |
| Performance Efficiency | The ALB distributes requests; S3 separates uploads from EC2; CloudFront remains a target-architecture extension because no distribution was created. |

#### Operational Verification Commands

```powershell
ipconfig /flushdns
nslookup -type=NS younglilliu.id.vn
nslookup sport.younglilliu.id.vn
curl.exe -I http://sport.younglilliu.id.vn
curl.exe -I https://sport.younglilliu.id.vn
```

```bash
sudo -i
systemctl status sportbooking --no-pager
journalctl -u sportbooking -n 100 --no-pager
curl -I http://127.0.0.1:8080/
curl http://127.0.0.1:8080/actuator/health
```
