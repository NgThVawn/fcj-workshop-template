---
title : "Launch Template"
date : 2024-01-01
weight : 2
chapter : false
pre : " <b> 5.5.2 </b> "
---

#### Launch Template

The Launch Template defines the standard EC2 application configuration: AMI, instance type, IAM instance profile, Security Group, and User Data. EC2 instances run in private application subnets, have no public IP addresses, and accept traffic only from the ALB.

| Item | Configuration |
|---|---|
| Launch Template | `sportbooking-lt`, version description `v1`. |
| AMI | The validated seed EC2 AMI or an Amazon Linux 2023 AMI with the standardized runtime. |
| Instance type | `t3.small`. |
| IAM instance profile | `sportbooking-ec2-role`. |
| Security Group | `sportbooking-app`, accepting port `8080` only from `sportbooking-alb`. |
| ASG subnets | Private application subnets. |
| Public IP | Disabled. |
| User Data | Installs the runtime, retrieves the secret, downloads the JAR from S3, and creates a systemd service. |

User Data used in the Launch Template:

```bash
#!/bin/bash
set -euxo pipefail

APP_DIR="/opt/sportbooking"
APP_ARTIFACT_S3_URI="s3://sportbooking-artifacts-3stars-us-east-1/releases/SportBooingSystem-0.0.1-SNAPSHOT.jar"
APP_ENV_SECRET_ID="sportbooking/prod/app-env"
AWS_DEFAULT_REGION="us-east-1"

mkdir -p "$APP_DIR"
dnf install -y java-21-amazon-corretto-headless jq awscli || true

aws secretsmanager get-secret-value \
  --secret-id "$APP_ENV_SECRET_ID" \
  --region "$AWS_DEFAULT_REGION" \
  --query SecretString \
  --output text | jq -r 'to_entries[] | "\(.key)=\(.value)"' > "$APP_DIR/app.env"

chmod 600 "$APP_DIR/app.env"
aws s3 cp "$APP_ARTIFACT_S3_URI" "$APP_DIR/app.jar" --region "$AWS_DEFAULT_REGION"

cat > /etc/systemd/system/sportbooking.service <<'EOF'
[Unit]
Description=SportBooking Spring Boot Application
After=network-online.target
Wants=network-online.target

[Service]
WorkingDirectory=/opt/sportbooking
EnvironmentFile=/opt/sportbooking/app.env
Environment=AWS_REGION=us-east-1
Environment=AWS_DEFAULT_REGION=us-east-1
Environment=AWS_S3_REGION=us-east-1
ExecStart=/usr/bin/java -Daws.region=us-east-1 -Daws.s3.region=us-east-1 -Dcloud.aws.region.static=us-east-1 -jar /opt/sportbooking/app.jar
Restart=always
RestartSec=10
User=root

[Install]
WantedBy=multi-user.target
EOF

systemctl daemon-reload
systemctl enable sportbooking
systemctl restart sportbooking
```

{{% notice warning %}}
Verify `APP_ARTIFACT_S3_URI`, `APP_ENV_SECRET_ID`, and `AWS_DEFAULT_REGION` carefully in User Data. An incorrect value in any of these variables allows EC2 to boot but prevents the application service from starting.
{{% /notice %}}

#### Launch Template Procedure

Create an AMI from the validated seed instance, then build a Launch Template with the IAM Role, Security Group, and User Data shared by all instances in the Auto Scaling Group.

**Procedure:**

1. **Create an AMI from the seed instance**

   Open the prepared seed EC2 instance, then choose **Actions > Image and templates > Create image**.

   ![Create an AMI from the seed instance](/images/5-Workshop/5.5-compute/launch-template-01.png)

   <p class="image-caption"><em>Figure 1: Create an AMI from the seed instance</em></p>
   The AMI captures the seed instance state so the Launch Template can create consistent new instances.


2. **Name the AMI**

   Enter a project-specific image name and add a description identifying the SportBooking AMI.

   ![Name the AMI](/images/5-Workshop/5.5-compute/launch-template-02.png)

   <p class="image-caption"><em>Figure 2: Name the AMI</em></p>
   A descriptive AMI name identifies the image used by SportBooking and its deployment version.


3. **Create the image**

   Review the volume and tags, then choose **Create image**.

   ![Create the image](/images/5-Workshop/5.5-compute/launch-template-03.png)

   <p class="image-caption"><em>Figure 3: Create the image</em></p>
   This image is the base for ASG instances. An AMI that does not exist or remains pending cannot be used reliably by the Launch Template.


4. **Confirm that the AMI was created**

   Return to EC2 AMIs or the instance page and wait for the successful image-creation message.

   ![Confirm that the AMI was created](/images/5-Workshop/5.5-compute/launch-template-04.png)

   <p class="image-caption"><em>Figure 4: Confirm that the AMI was created</em></p>
   Use the AMI only after its state becomes `Available` to prevent ASG launch failures.


5. **Start creating the Launch Template**

   Open **EC2 > Launch Templates**, then choose **Create launch template**.

   ![Start creating the Launch Template](/images/5-Workshop/5.5-compute/launch-template-05.png)

   <p class="image-caption"><em>Figure 5: Start creating the Launch Template</em></p>
   The Launch Template standardizes the AMI, instance type, IAM Role, Security Group, and User Data for every EC2 instance in the ASG.


6. **Enter the Launch Template name**

   Enter `sportbooking-lt` as the Launch Template name and `v1` as the version description. Enable the guidance option for EC2 Auto Scaling.

   ![Enter the Launch Template name](/images/5-Workshop/5.5-compute/launch-template-06.png)

   <p class="image-caption"><em>Figure 6: Enter the Launch Template name</em></p>
   The ASG references `sportbooking-lt` and default version `1`.


7. **Select the AMI**

   Open the AMI section and select the AMI created from the seed instance.

   ![Select the AMI for the Launch Template](/images/5-Workshop/5.5-compute/launch-template-07.png)

   <p class="image-caption"><em>Figure 7: Select the AMI for the Launch Template</em></p>
   The AMI ensures that new instances use the same base runtime as the validated environment.


8. **Select the instance type**

   Select `t3.small` with 2 vCPUs and 2 GiB of memory.

   ![Select the instance type](/images/5-Workshop/5.5-compute/launch-template-08.png)

   <p class="image-caption"><em>Figure 8: Select the instance type</em></p>
   Compute runs continuously in the ASG. A small instance limits cost while providing sufficient capacity for Spring Boot.


9. **Configure the key pair and network security**

   Select **Don't include in launch template** for the key pair, then select the `sportbooking-app` Security Group.

   ![Configure the key pair and network security](/images/5-Workshop/5.5-compute/launch-template-09.png)

   <p class="image-caption"><em>Figure 9: Configure the key pair and network security</em></p>
   `sportbooking-app` permits port `8080` only from `sportbooking-alb`; public SSH remains closed.


10. **Attach the IAM instance profile**

   Open **Advanced details**, then select `sportbooking-ec2-role` as the IAM instance profile.

   ![Attach the IAM instance profile](/images/5-Workshop/5.5-compute/launch-template-10.png)

   <p class="image-caption"><em>Figure 10: Attach the IAM instance profile</em></p>
   The role allows EC2 to read the S3 artifact, read Secrets Manager, and register with Session Manager at startup.


11. **Enter the User Data bootstrap**

   Paste the User Data script that installs the runtime, retrieves the secret, downloads the JAR from S3, and creates the systemd service. Verify the S3 URI, secret ID, and Region.

   ![Enter the User Data bootstrap](/images/5-Workshop/5.5-compute/launch-template-11.png)

   <p class="image-caption"><em>Figure 11: Enter the User Data bootstrap</em></p>
   User Data turns the Launch Template into an automated deployment process. Each new instance retrieves its configuration and artifact without manual intervention.


12. **Create the Launch Template**

   Review the configuration, then choose **Create launch template**.

   ![Create the Launch Template](/images/5-Workshop/5.5-compute/launch-template-12.png)

   <p class="image-caption"><em>Figure 12: Create the Launch Template</em></p>
   The template is an input to the ASG and Instance Refresh. Create a new version whenever the AMI or User Data changes.
