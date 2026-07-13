---
title : "Operations, Monitoring, and Troubleshooting"
date : 2024-01-01
weight : 7
chapter : false
pre : " <b> 5.7. </b> "
---

#### Deployment Scope

This section presents the operational workflow after the website is available through HTTPS: update the artifact, update the secret, verify the database, configure OAuth, create an SNS notification channel, create a CloudWatch alarm, and document common errors.

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

#### SNS Alerting

SNS provides the notification channel for the CloudWatch alarm. Create and confirm the topic and email subscription before creating the alarm so the action has a valid destination.

| Step | Verified information |
|---|---|
| 1 | The SNS topic is created in `us-east-1`. |
| 2 | The email subscription is `Confirmed`. |
| 3 | The CloudWatch alarm action uses the correct SNS topic. |
| 4 | An alert email is delivered when the alarm changes state. |

#### SNS Procedure

Configure the notification channel with an SNS topic, an email subscription, and confirmation of the recipient address.

**Procedure:**

1. **Start creating the SNS topic**

   Open **SNS > Topics**, then choose **Create topic**.

   ![Start creating the SNS topic](/images/5-Workshop/5.7-Operation/sns-01.png)

   <p class="image-caption"><em>Figure 1: Start creating the SNS topic</em></p>
   The SNS topic receives notifications from the CloudWatch alarm.


2. **Select the topic type and enter a name**

   Select **Standard** and enter `sportbooking-prod-alerts` as the topic name.

   ![Select the topic type and enter a name](/images/5-Workshop/5.7-Operation/sns-02.png)

   <p class="image-caption"><em>Figure 2: Select the topic type and enter a name</em></p>
   A Standard topic is suitable for simple email alerts that do not require FIFO ordering.


3. **Create the topic**

   Keep the delivery and tag settings at their defaults, then choose **Create topic**.

   ![Create the SNS topic](/images/5-Workshop/5.7-Operation/sns-03.png)

   <p class="image-caption"><em>Figure 3: Create the topic</em></p>
   The Standard topic is used for CloudWatch alarm emails; advanced delivery settings are not enabled.


4. **Open the new topic**

   Verify the topic ARN and details.

   ![Open the new topic](/images/5-Workshop/5.7-Operation/sns-04.png)

   <p class="image-caption"><em>Figure 4: Open the new topic</em></p>
   Select this topic ARN as the notification action in the CloudWatch alarm.


5. **Start creating a subscription**

   In the topic, choose **Create subscription**.

   ![Start creating a subscription](/images/5-Workshop/5.7-Operation/sns-05.png)

   <p class="image-caption"><em>Figure 5: Start creating a subscription</em></p>
   Without a subscription, the alarm publishes to the topic but no endpoint receives the message.


6. **Enter the protocol and endpoint**

   Select **Email** as the protocol and enter the alert recipient address.

   ![Enter the subscription protocol and endpoint](/images/5-Workshop/5.7-Operation/sns-06.png)

   <p class="image-caption"><em>Figure 6: Enter the protocol and endpoint</em></p>
   Email is the simplest notification method for this test deployment.


7. **Create the subscription**

   Verify the email address, then choose **Create subscription**.

   ![Create the SNS subscription](/images/5-Workshop/5.7-Operation/sns-07.png)

   <p class="image-caption"><em>Figure 7: Create the subscription</em></p>
   AWS sends a confirmation email; the subscription remains pending until its confirmation link is opened.


8. **Open the confirmation email**

   Open the mailbox and select the **AWS Notification - Subscription Confirmation** email.

   ![Open the subscription confirmation email](/images/5-Workshop/5.7-Operation/sns-08.png)

   <p class="image-caption"><em>Figure 8: Open the confirmation email</em></p>
   This security step confirms that the address agrees to receive alerts.


9. **Confirm the subscription**

   Select **Confirm subscription** in the email.

   ![Confirm the SNS subscription](/images/5-Workshop/5.7-Operation/sns-09.png)

   <p class="image-caption"><em>Figure 9: Confirm the subscription</em></p>
   The endpoint receives messages from the SNS topic only after confirmation.


10. **Verify the confirmed subscription**

   Return to the SNS subscription and confirm that its status is **Confirmed**.

   ![Verify the confirmed subscription](/images/5-Workshop/5.7-Operation/sns-10.png)

   <p class="image-caption"><em>Figure 10: Verify the confirmed subscription</em></p>
   Confirm the subscription before associating the topic with the CloudWatch alarm.



#### CloudWatch Monitoring

CloudWatch monitors operational metrics for EC2, the ALB and Target Group, and RDS. Create `SB-ALB-UnhealthyTargets` after SNS and configure it to monitor `UnHealthyHostCount` for `sportbooking-tg`.

| Resource | Verified metrics | Purpose |
|---|---|---|
| EC2/ASG | CPUUtilization, StatusCheckFailed | Detects unhealthy or overloaded instances. |
| ALB | HTTPCode_Target_5XX_Count, TargetResponseTime, HealthyHostCount | Monitors backend errors and target health. |
| RDS | CPUUtilization, FreeStorageSpace, DatabaseConnections | Monitors the database during operation. |

#### CloudWatch Alarm Procedure

Use a Target Group metric, require the threshold for two consecutive periods, and send the alarm notification through SNS.

**Procedure:**

1. **Start creating the CloudWatch alarm**

   Open **CloudWatch > Alarms**, then choose **Create alarm**.

   ![Start creating the CloudWatch alarm](/images/5-Workshop/5.7-Operation/cloudwatch-01.png)

   <p class="image-caption"><em>Figure 1: Start creating the CloudWatch alarm</em></p>
   An alarm provides early detection when EC2, the ALB, or RDS fails or a metric crosses its threshold.


2. **Select a metric**

   Choose **Select metric**, then select the **ApplicationELB** namespace.

   ![Select a CloudWatch metric](/images/5-Workshop/5.7-Operation/cloudwatch-02.png)

   <p class="image-caption"><em>Figure 2: Select a metric</em></p>
   This alarm focuses on targets that fail health checks behind the ALB.


3. **Open the ApplicationELB namespace**

   Open the Application Load Balancer metric category and select the metric type associated with a Target Group or Load Balancer.

   ![Open the ApplicationELB namespace](/images/5-Workshop/5.7-Operation/cloudwatch-03.png)

   <p class="image-caption"><em>Figure 3: Select the ApplicationELB namespace</em></p>
   ALB metrics indicate whether backends are healthy and whether users encounter 5xx errors.


4. **Select the specific metric**

   Select `UnHealthyHostCount` for `sportbooking-alb` and `sportbooking-tg`, then choose **Select metric**.

   ![Select the UnHealthyHostCount metric](/images/5-Workshop/5.7-Operation/cloudwatch-04.png)

   <p class="image-caption"><em>Figure 4: Select the specific metric</em></p>
   This metric increases when one or more EC2 application instances behind the ALB fail their health checks.


5. **Set the alarm condition**

   Select **Maximum** as the statistic and set the period to **1 minute**.

   ![Set the alarm condition](/images/5-Workshop/5.7-Operation/cloudwatch-05.png)

   <p class="image-caption"><em>Figure 5: Set the alarm condition</em></p>
   This statistic and period capture the highest unhealthy-target count in each one-minute interval.


6. **Configure the threshold and datapoints**

   Set the condition to `UnHealthyHostCount >= 1` and **Datapoints to alarm** to `2 out of 2`. Keep **Treat missing data as missing**, then choose **Next**.

   ![Configure the threshold and datapoints](/images/5-Workshop/5.7-Operation/cloudwatch-06.png)

   <p class="image-caption"><em>Figure 6: Configure the threshold and datapoints</em></p>
   Requiring two datapoints reduces false alerts caused by one brief anomalous data point.


7. **Select the notification action**

   Select the alarm state that sends a notification, then select the SNS topic created earlier.

   ![Select the notification action](/images/5-Workshop/5.7-Operation/cloudwatch-07.png)

   <p class="image-caption"><em>Figure 7: Select the notification action</em></p>
   An alarm requires a notification channel to be operationally useful. SNS sends email when the system has a problem.


8. **Skip unused additional actions**

   Do not configure EC2, Auto Scaling, or Systems Manager actions for this alarm. Choose **Next**.

   ![Skip unused additional actions](/images/5-Workshop/5.7-Operation/cloudwatch-08.png)

   <p class="image-caption"><em>Figure 8: Skip unused additional actions</em></p>
   This alarm sends a notification through SNS and does not modify resources automatically.


9. **Name the alarm**

   Enter `SB-ALB-UnhealthyTargets` as the alarm name. Leave the description and tags empty for this creation, then choose **Next**.

   ![Name the CloudWatch alarm](/images/5-Workshop/5.7-Operation/cloudwatch-09.png)

   <p class="image-caption"><em>Figure 9: Name the alarm</em></p>
   The alarm name should identify the resource and condition so the issue is immediately clear in an email.


10. **Review and create the alarm**

   Review the metric, condition, action, and name, then choose **Create alarm**.

   ![Review and create the CloudWatch alarm](/images/5-Workshop/5.7-Operation/cloudwatch-10.png)

   <p class="image-caption"><em>Figure 10: Review and create the alarm</em></p>
   Validate the metric, condition, SNS action, and alarm name before creation.


11. **Verify the alarm in the list**

   Return to **Alarms** and confirm that `SB-ALB-UnhealthyTargets` appears with actions enabled and an initial `Insufficient data` state.

   ![Verify the alarm in the list](/images/5-Workshop/5.7-Operation/cloudwatch-11.png)

   <p class="image-caption"><em>Figure 11: Verify the alarm in the list</em></p>
   The OK, Alarm, and Insufficient data states show whether CloudWatch has received metrics and evaluated the condition.



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

