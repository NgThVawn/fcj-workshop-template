---
title : "Secrets Manager"
date : 2024-01-01
weight : 3
chapter : false
pre : " <b> 5.4.3 </b> "
aliases:
  - /5-workshop/5.4-s3-onprem/5.4.3-secrets-manager/
---

#### Deployment Scope

Create the `sportbooking/prod/app-env` secret after RDS reaches the `Available` state. Store the RDS endpoint and credentials together with the application's environment variables; during the bootstrap process in section 5.5, EC2 reads this secret to create `/opt/sportbooking/app.env`.

#### Configuration Variables

| Variable | Value used | Purpose |
|---|---|---|
| `DB_URL` | `jdbc:mysql://sportbooking-mysql.cudgu8qk8z19.us-east-1.rds.amazonaws.com:3306/sport_booking?useSSL=true&serverTimezone=Asia/Ho_Chi_Minh` | Connects to Amazon RDS for MySQL. |
| `DB_USERNAME` | `sportbooking_ad` | Database account. |
| `DB_PASSWORD` | Value concealed in the image | Database password. |
| `SPRING_JPA_HIBERNATE_DDL_AUTO` | `update` | Allows schema updates during the test deployment. |
| `APP_BASE_URL` | `https://sport.younglilliu.id.vn` | Primary application domain. |
| `APP_UPLOADS_BASE_URL` | `https://sport.younglilliu.id.vn/uploads` | Base URL for uploaded images. |
| `SERVER_FORWARD_HEADERS_STRATEGY` | `framework` | Allows Spring to interpret `X-Forwarded-*` headers correctly behind ALB HTTPS. |
| `APP_STORAGE_TYPE` | `s3` | Enables S3 image storage. |
| `APP_STORAGE_S3_BUCKET` | `sportbooking-uploads-3stars-us-east-1` | Upload image bucket. |
| `APP_STORAGE_S3_KEY_PREFIX` | `uploads` | Image object prefix. |
| `AWS_REGION`, `AWS_DEFAULT_REGION`, `AWS_S3_REGION` | `us-east-1` | Keeps the AWS SDK and CLI Region consistent. |

{{% notice warning %}}
Do not expose `DB_PASSWORD`, OAuth client secrets, tokens, or secret values in report images. The evidence only needs to show that the required configuration keys and correct secret name exist.
{{% /notice %}}

#### Verify Secret Read Access After EC2 Deployment

After the EC2 application is created in section 5.5, use Session Manager to verify secret read access:

```bash
sudo -i
aws secretsmanager get-secret-value \
  --secret-id sportbooking/prod/app-env \
  --region us-east-1 \
  --query SecretString \
  --output text | jq -r 'to_entries[] | "\(.key)=\(.value)"' > /tmp/app.env

grep -n -E "APP_BASE_URL|APP_STORAGE_TYPE|APP_STORAGE_S3_BUCKET|AWS_REGION" /tmp/app.env
```

{{% notice tip %}}
Display only non-sensitive keys with `grep` in the evidence. Do not print the complete environment file or any secret values to the screen.
{{% /notice %}}

#### Procedure

Store the configuration as key/value pairs, name the secret by environment, and confirm that it exists before granting EC2 access.

**Procedure:**

1. **Start creating the secret**

   Open **Secrets Manager > Secrets**, then choose **Store a new secret**.

   ![Start creating the secret](/images/5-Workshop/5.4-data-access/secrets-manager-01.png)

   <p class="image-caption"><em>Figure 1: Start creating the secret</em></p>
   The secret separates sensitive configuration from the source code and JAR, especially the database password, OAuth secret, and storage configuration.


2. **Select the secret type and enter key/value pairs**

   Select **Other type of secret**, then enter key/value pairs such as `DB_URL`, `DB_USERNAME`, `APP_BASE_URL`, and `APP_STORAGE_TYPE`.

   ![Select the secret type and enter key/value pairs](/images/5-Workshop/5.4-data-access/secrets-manager-02.png)

   <p class="image-caption"><em>Figure 2: Select the secret type and enter key/value pairs</em></p>
   The Spring Boot application uses multiple environment variables, so key/value storage can be converted directly into `/opt/sportbooking/app.env` when EC2 starts.


3. **Complete the environment variables**

   Add the remaining keys for the S3 bucket, upload prefix, AWS Region, and OAuth configuration, then choose **Next**.

   ![Complete the environment variables](/images/5-Workshop/5.4-data-access/secrets-manager-03.png)

   <p class="image-caption"><em>Figure 3: Complete the environment variables</em></p>
   A missing variable such as `AWS_S3_REGION` or `APP_STORAGE_S3_BUCKET` can break image uploads even when EC2 and RDS are running.


4. **Name the secret**

   Enter `sportbooking/prod/app-env` as the secret name, add a description identifying the SportBooking deployment environment, and choose **Next**.

   ![Name the secret](/images/5-Workshop/5.4-data-access/secrets-manager-04.png)

   <p class="image-caption"><em>Figure 4: Name the secret</em></p>
   Keep the secret name stable so it matches references in the IAM policy and User Data.


5. **Configure rotation**

   Automatic rotation is not enabled within this deployment scope. Choose **Next**.

   ![Configure secret rotation](/images/5-Workshop/5.4-data-access/secrets-manager-05.png)

   <p class="image-caption"><em>Figure 5: Configure rotation</em></p>
   Rotation can be enabled in a later operational phase when the application and database account support synchronized password changes.


6. **Review and store the secret**

   Review all non-sensitive keys and values. Conceal passwords and tokens in report images, then choose **Store**.

   ![Review and store the secret](/images/5-Workshop/5.4-data-access/secrets-manager-06.png)

   <p class="image-caption"><em>Figure 6: Review and store the secret</em></p>
   Verify key names and the Region before saving, and conceal every sensitive value in the report.


7. **Confirm that the secret exists**

   Return to the Secrets list and confirm that `sportbooking/prod/app-env` appears.

   ![Confirm that the secret exists](/images/5-Workshop/5.4-data-access/secrets-manager-07.png)

   <p class="image-caption"><em>Figure 7: Confirm that the secret exists</em></p>
   This evidence supports the next IAM Role step: the EC2 Role requires `secretsmanager:GetSecretValue` permission on this secret.
