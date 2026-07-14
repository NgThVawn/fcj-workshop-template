---
title : "Application Updates"
date : 2024-01-01
weight : 1
chapter : false
pre : " <b> 5.7.1 </b> "
---

#### Artifact Update Procedure

| Step | Action | Command/Console path |
|---|---|---|
| 1 | Package the JAR locally | `.\mvnw.cmd clean package -DskipTests` |
| 2 | Overwrite the artifact in S3 | `aws s3 cp target\SportBooingSystem-0.0.1-SNAPSHOT.jar s3://sportbooking-artifacts-3stars-us-east-1/releases/SportBooingSystem-0.0.1-SNAPSHOT.jar --region us-east-1` |
| 3 | Start Instance Refresh | ASG > Instance refresh > Start > Use current Auto Scaling group configuration |
| 4 | Wait for targets to become Healthy | Target Group > Targets > Healthy |
| 5 | Retest the domain | `curl.exe -I https://sport.younglilliu.id.vn` and a browser |

#### Secret Update Procedure

| Step | Action | Details |
|---|---|---|
| 1 | Edit the secret | Secrets Manager > `sportbooking/prod/app-env` > Edit. |
| 2 | Protect sensitive data | Do not include `DB_PASSWORD` or OAuth secrets in images. |
| 3 | Redeploy EC2 | Instance Refresh creates new instances that read the current secret version. |
| 4 | Verify `app.env` | Use `grep` only for non-sensitive keys on a new EC2 instance. |
| 5 | Test functionality | Registration, login, OAuth, and image upload. |

{{% notice warning %}}
Changing the secret does not automatically update a running EC2 instance if the application reads it only during boot. After editing the secret, restart the service or run Instance Refresh so new instances load the updated configuration.
{{% /notice %}}

#### Verify the Database and Seed Data

After the EC2 application is running, connect to RDS from EC2 through Session Manager:

```bash
sudo -i
mariadb -h sportbooking-mysql.cudgu8qk8z19.us-east-1.rds.amazonaws.com -P 3306 -u sportbooking_ad -p
```

Verify the schema:

```sql
USE sport_booking;
SHOW TABLES;
```

If the `membership_levels` table exists but contains no data, run the following idempotent statement:

```sql
INSERT INTO membership_levels
  (name, min_bookings, discount_rate, description, badge_color)
VALUES
  ('NONE', 0, 0.0000, 'Thanh vien thuong', '#888888'),
  ('SILVER', 10, 0.0500, 'Bac - giam 5%', '#C0C0C0'),
  ('GOLD', 30, 0.0700, 'Vang - giam 7%', '#FFD700'),
  ('DIAMOND', 100, 0.1000, 'Kim cuong - giam 10%', '#B9F2FF')
ON DUPLICATE KEY UPDATE
  min_bookings = VALUES(min_bookings),
  discount_rate = VALUES(discount_rate),
  description = VALUES(description),
  badge_color = VALUES(badge_color);
```

{{% notice info %}}
`SPRING_JPA_HIBERNATE_DDL_AUTO=update` was used in this deployment. Flyway and Liquibase were outside the implementation scope and remain future work for schema version management.
{{% /notice %}}

#### OAuth Configuration After Enabling HTTPS

Google OAuth:

| Item | Configuration value |
|---|---|
| Authorized JavaScript origins | `https://sport.younglilliu.id.vn` |
| Authorized redirect URIs | `https://sport.younglilliu.id.vn/login/oauth2/code/google` |
| Common error | `redirect_uri_mismatch` when the redirect URI does not match exactly. |
| Resolution | Add the exact HTTPS callback, use the correct `younglilliu` spelling, and do not use the ALB DNS name. |

Facebook OAuth:

| Item | Configuration value |
|---|---|
| App domain | `sport.younglilliu.id.vn` |
| Valid OAuth Redirect URI | `https://sport.younglilliu.id.vn/login/oauth2/code/facebook` |
| Common error | Facebook requires a secure connection when HTTP or an ALB DNS name is used. |
| Resolution | Use the HTTPS domain through ACM and ALB listener 443. |
