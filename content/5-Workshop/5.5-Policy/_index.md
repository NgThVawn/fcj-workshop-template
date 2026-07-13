---
title : "EC2, ALB, and Auto Scaling Group"
date : 2024-01-01
weight : 5
chapter : false
pre : " <b> 5.5. </b> "
---

#### Deployment Scope

Deploy the compute layer only after S3 contains the artifact and the IAM Role, RDS, and Secrets Manager resources are ready. The sequence is: create a seed EC2 instance to verify the runtime and permissions; create an AMI and Launch Template; create the Target Group and Application Load Balancer; and finally create the Auto Scaling Group in private application subnets.

{{% notice info %}}
Create the Auto Scaling Group after the Target Group and ALB because the ASG is associated with the Target Group during configuration. This order automatically registers and health-checks new EC2 instances as they launch.
{{% /notice %}}

#### Seed EC2 Configuration

Create the seed EC2 instance after `releases/SportBooingSystem-0.0.1-SNAPSHOT.jar` exists in the artifact bucket. Use this instance to verify Amazon Linux, the Java runtime, Session Manager, the IAM Role, and access to foundational services before standardizing the configuration in a Launch Template.

#### Seed EC2 Deployment Procedure

Provision the seed instance only after S3, IAM, RDS, and Secrets Manager are ready so the validation has the required artifact, permissions, and application configuration.

**Procedure:**

1. **Start creating the seed EC2 instance**

   Open **EC2 > Instances**, then choose **Launch instances**.

   ![Start creating the seed EC2 instance](/images/5-Workshop/5.5-Policy/ec2-seed-01.png)

   <p class="image-caption"><em>Figure 1: Start creating the seed EC2 instance</em></p>
   The seed instance validates the runtime, IAM, networking, and application behavior before an AMI is created for the Launch Template.


2. **Name the instance and select an AMI**

   Enter a project-specific instance name, then select Amazon Linux 2023.

   ![Name the instance and select an AMI](/images/5-Workshop/5.5-Policy/ec2-seed-02.png)

   <p class="image-caption"><em>Figure 2: Name the instance and select an AMI</em></p>
   The name distinguishes the seed instance from EC2 instances launched by the ASG. The AMI must support the application's Java runtime.


3. **Confirm the AMI and select the instance type**

   Confirm the selected AMI, then select `t3.small` as the instance type.

   ![Confirm the AMI and select the instance type](/images/5-Workshop/5.5-Policy/ec2-seed-03.png)

   <p class="image-caption"><em>Figure 3: Confirm the AMI and select the instance type</em></p>
   The seed instance uses `t3.small`, which is sufficient for validating the project runtime and application.


4. **Configure the key pair and network**

   Select **Proceed without a key pair** because Session Manager administers the instance. Select `sportbooking-vpc` and the `sportbooking-app-a` private subnet, then disable **Auto-assign public IP**.

   ![Configure the key pair and network](/images/5-Workshop/5.5-Policy/ec2-seed-04.png)

   <p class="image-caption"><em>Figure 4: Configure the key pair and network</em></p>
   The instance has no public IP address and uses no key pair; all administrative sessions use Session Manager.


5. **Attach the Security Group and configure storage**

   Select the `sportbooking-app` Security Group, then configure a `20 GiB` `gp3` root volume.

   ![Attach the Security Group and configure storage](/images/5-Workshop/5.5-Policy/ec2-seed-05.png)

   <p class="image-caption"><em>Figure 5: Attach the Security Group and configure storage</em></p>
   The Security Group accepts application traffic only from the ALB. The root volume stores the runtime, configuration file, and artifact retrieved from S3.


6. **Attach the IAM instance profile**

   Open **Advanced details**, select `sportbooking-ec2-role` as the IAM Role, and choose **Launch instance**.

   ![Attach the IAM instance profile](/images/5-Workshop/5.5-Policy/ec2-seed-06.png)

   <p class="image-caption"><em>Figure 6: Attach the IAM instance profile</em></p>
   The role allows the instance to read the S3 artifact, read the secret, access the upload bucket, and use Session Manager without storing access keys.


7. **Verify the instance after launch**

   Open the new instance and wait for the `Running` state and successful status checks.

   ![Verify the instance after launch](/images/5-Workshop/5.5-Policy/ec2-seed-07.png)

   <p class="image-caption"><em>Figure 7: Verify the instance after launch</em></p>
   Open a connection only after the instance reaches `Running` and passes its status checks.


8. **Choose Connect**

   Select the instance, then choose **Connect**.

   ![Choose Connect for the seed instance](/images/5-Workshop/5.5-Policy/ec2-seed-08.png)

   <p class="image-caption"><em>Figure 8: Choose Connect</em></p>
   The session is used to verify IAM permissions, load the secret, start the application, and test local HTTP.


9. **Connect through Session Manager**

   Select the **Session Manager** tab, then choose **Connect**.

   ![Connect through Session Manager](/images/5-Workshop/5.5-Policy/ec2-seed-09.png)

   <p class="image-caption"><em>Figure 9: Connect through Session Manager</em></p>
   Session Manager is appropriate for private EC2 instances because it requires neither inbound SSH nor a bastion host.


10. **Verify the EC2 terminal**

   Read the `sportbooking/prod/app-env` secret and create `/opt/sportbooking/app.env`. Restart the `sportbooking` service and run `curl -I http://127.0.0.1:8080/`. Record the `HTTP/1.1 200` response after the service starts successfully.

   ![Verify the EC2 terminal](/images/5-Workshop/5.5-Policy/ec2-seed-10.png)

   <p class="image-caption"><em>Figure 10: Verify the EC2 terminal</em></p>
   The terminal confirms that EC2 has the correct IAM permissions, network access, and runtime before Launch Template automation is introduced.



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

   ![Create an AMI from the seed instance](/images/5-Workshop/5.5-Policy/launch-template-01.png)

   <p class="image-caption"><em>Figure 1: Create an AMI from the seed instance</em></p>
   The AMI captures the seed instance state so the Launch Template can create consistent new instances.


2. **Name the AMI**

   Enter a project-specific image name and add a description identifying the SportBooking AMI.

   ![Name the AMI](/images/5-Workshop/5.5-Policy/launch-template-02.png)

   <p class="image-caption"><em>Figure 2: Name the AMI</em></p>
   A descriptive AMI name identifies the image used by SportBooking and its deployment version.


3. **Create the image**

   Review the volume and tags, then choose **Create image**.

   ![Create the image](/images/5-Workshop/5.5-Policy/launch-template-03.png)

   <p class="image-caption"><em>Figure 3: Create the image</em></p>
   This image is the base for ASG instances. An AMI that does not exist or remains pending cannot be used reliably by the Launch Template.


4. **Confirm that the AMI was created**

   Return to EC2 AMIs or the instance page and wait for the successful image-creation message.

   ![Confirm that the AMI was created](/images/5-Workshop/5.5-Policy/launch-template-04.png)

   <p class="image-caption"><em>Figure 4: Confirm that the AMI was created</em></p>
   Use the AMI only after its state becomes `Available` to prevent ASG launch failures.


5. **Start creating the Launch Template**

   Open **EC2 > Launch Templates**, then choose **Create launch template**.

   ![Start creating the Launch Template](/images/5-Workshop/5.5-Policy/launch-template-05.png)

   <p class="image-caption"><em>Figure 5: Start creating the Launch Template</em></p>
   The Launch Template standardizes the AMI, instance type, IAM Role, Security Group, and User Data for every EC2 instance in the ASG.


6. **Enter the Launch Template name**

   Enter `sportbooking-lt` as the Launch Template name and `v1` as the version description. Enable the guidance option for EC2 Auto Scaling.

   ![Enter the Launch Template name](/images/5-Workshop/5.5-Policy/launch-template-06.png)

   <p class="image-caption"><em>Figure 6: Enter the Launch Template name</em></p>
   The ASG references `sportbooking-lt` and default version `1`.


7. **Select the AMI**

   Open the AMI section and select the AMI created from the seed instance.

   ![Select the AMI for the Launch Template](/images/5-Workshop/5.5-Policy/launch-template-07.png)

   <p class="image-caption"><em>Figure 7: Select the AMI for the Launch Template</em></p>
   The AMI ensures that new instances use the same base runtime as the validated environment.


8. **Select the instance type**

   Select `t3.small` with 2 vCPUs and 2 GiB of memory.

   ![Select the instance type](/images/5-Workshop/5.5-Policy/launch-template-08.png)

   <p class="image-caption"><em>Figure 8: Select the instance type</em></p>
   Compute runs continuously in the ASG. A small instance limits cost while providing sufficient capacity for Spring Boot.


9. **Configure the key pair and network security**

   Select **Don't include in launch template** for the key pair, then select the `sportbooking-app` Security Group.

   ![Configure the key pair and network security](/images/5-Workshop/5.5-Policy/launch-template-09.png)

   <p class="image-caption"><em>Figure 9: Configure the key pair and network security</em></p>
   `sportbooking-app` permits port `8080` only from `sportbooking-alb`; public SSH remains closed.


10. **Attach the IAM instance profile**

   Open **Advanced details**, then select `sportbooking-ec2-role` as the IAM instance profile.

   ![Attach the IAM instance profile](/images/5-Workshop/5.5-Policy/launch-template-10.png)

   <p class="image-caption"><em>Figure 10: Attach the IAM instance profile</em></p>
   The role allows EC2 to read the S3 artifact, read Secrets Manager, and register with Session Manager at startup.


11. **Enter the User Data bootstrap**

   Paste the User Data script that installs the runtime, retrieves the secret, downloads the JAR from S3, and creates the systemd service. Verify the S3 URI, secret ID, and Region.

   ![Enter the User Data bootstrap](/images/5-Workshop/5.5-Policy/launch-template-11.png)

   <p class="image-caption"><em>Figure 11: Enter the User Data bootstrap</em></p>
   User Data turns the Launch Template into an automated deployment process. Each new instance retrieves its configuration and artifact without manual intervention.


12. **Create the Launch Template**

   Review the configuration, then choose **Create launch template**.

   ![Create the Launch Template](/images/5-Workshop/5.5-Policy/launch-template-12.png)

   <p class="image-caption"><em>Figure 12: Create the Launch Template</em></p>
   The template is an input to the ASG and Instance Refresh. Create a new version whenever the AMI or User Data changes.



#### Target Group and Application Load Balancer

The Target Group health-checks the EC2 application on port `8080`. The Application Load Balancer is the public entry point that accepts HTTP/HTTPS requests and forwards them to the Target Group.

| Component | Configuration |
|---|---|
| Target type | Instances |
| Target Group protocol/port | HTTP: `8080` |
| Health check path | `/actuator/health` |
| ALB scheme | Internet-facing |
| ALB subnets | Two public subnets in two AZs |
| ALB Security Group | Allows inbound 80/443 from the Internet |
| Initial listener | HTTP:80 forwards to `sportbooking-tg` |

Recorded result: the Target Group uses the correct health check path, the ALB is active, and its DNS name can be tested before the domain is configured.

#### Target Group and ALB Procedure

Create and validate the Target Group health check before the public ALB sends traffic to the application, then test the website through the ALB DNS name.

**Procedure:**

1. **Start creating the Target Group**

   Open **EC2 > Target Groups**, then choose **Create target group**.

   ![Start creating the Target Group](/images/5-Workshop/5.5-Policy/alb-01.png)

   <p class="image-caption"><em>Figure 1: Start creating the Target Group</em></p>
   The Target Group receives ALB requests for the EC2 application and performs health checks.


2. **Select the target type and protocol**

   Select **Instances** as the target type, enter the Target Group name, and select HTTP on port `8080`.

   ![Select the target type and protocol](/images/5-Workshop/5.5-Policy/alb-02.png)

   <p class="image-caption"><em>Figure 2: Select the target type and protocol</em></p>
   The Spring Boot application runs on EC2 port 8080. The ALB accepts public traffic on 80/443 and forwards it internally to 8080.


3. **Select the VPC and protocol version**

   Select the SportBooking VPC and retain HTTP1 unless the application requires another protocol version.

   ![Select the VPC and protocol version](/images/5-Workshop/5.5-Policy/alb-03.png)

   <p class="image-caption"><em>Figure 3: Select the VPC and protocol version</em></p>
   The Target Group must be in the same VPC as the EC2 application before instances can be registered.


4. **Configure the health check**

   Enter `/actuator/health` as the health check path and retain HTTP as the protocol.

   ![Configure the health check](/images/5-Workshop/5.5-Policy/alb-04.png)

   <p class="image-caption"><em>Figure 4: Configure the health check</em></p>
   The health check allows the ALB to send traffic only to healthy application instances. `/actuator/health` reports Spring Boot status more accurately than `/`.


5. **Continue to target registration**

   Review the target selection and attributes, then choose **Next**.

   ![Complete the Target Group configuration](/images/5-Workshop/5.5-Policy/alb-05.png)

   <p class="image-caption"><em>Figure 5: Complete the Target Group configuration and continue to target registration</em></p>
   After protocol and health check settings are complete, register the seed EC2 instance to test the ALB before creating the ASG.


6. **Select the seed EC2 instance as the initial target**

   Select `sportbooking-seed`, enter port `8080`, and choose **Include as pending below**.

   ![Select the instance when registering a target manually](/images/5-Workshop/5.5-Policy/alb-06.png)

   <p class="image-caption"><em>Figure 6: Select sportbooking-seed as the target on port 8080</em></p>
   Register the seed instance to test the Target Group and ALB first; the Auto Scaling Group subsequently manages registration for the instances it creates.


7. **Review the target**

   Verify `sportbooking-seed`, port `8080`, and the `Running` state in the pending-target list, then choose **Create target group**.

   ![Review the registered target](/images/5-Workshop/5.5-Policy/alb-07.png)

   <p class="image-caption"><em>Figure 7: Review the target</em></p>
   An incorrect target port is a common cause of failed health checks.


8. **Confirm Target Group creation**

   Review the `/actuator/health` health check and `sportbooking-seed` target, then choose **Create target group**.

   ![Confirm Target Group creation](/images/5-Workshop/5.5-Policy/alb-08.png)

   <p class="image-caption"><em>Figure 8: Confirm Target Group creation</em></p>
   This final step makes the Target Group available for the ALB and ASG.


9. **Confirm the pending target registration**

   When the AWS Console displays the pending-target confirmation dialog, choose **Continue**.

   ![Confirm the pending target registration](/images/5-Workshop/5.5-Policy/alb-09.png)

   <p class="image-caption"><em>Figure 9: Confirm the pending target registration</em></p>
   AWS warns that the target is awaiting a health check. The state changes to Healthy after the application runs successfully for several minutes.


10. **Verify the Target Group**

   Open the new Target Group and inspect its **Targets** and health check settings.

   ![Verify the Target Group](/images/5-Workshop/5.5-Policy/alb-10.png)

   <p class="image-caption"><em>Figure 10: Verify the Target Group</em></p>
   A Healthy target is an important prerequisite before the domain points to the ALB.


11. **Start creating the Load Balancer**

   Open **EC2 > Load Balancers**, then choose **Create load balancer**.

   ![Start creating the Load Balancer](/images/5-Workshop/5.5-Policy/alb-11.png)

   <p class="image-caption"><em>Figure 11: Start creating the Load Balancer</em></p>
   The Load Balancer is the website's public entry point, replacing direct access to private EC2 instances.


12. **Select Application Load Balancer**

   Select **Application Load Balancer**, then choose **Create**.

   ![Select Application Load Balancer](/images/5-Workshop/5.5-Policy/alb-12.png)

   <p class="image-caption"><em>Figure 12: Select Application Load Balancer</em></p>
   An ALB supports HTTP/HTTPS web traffic, listeners, rules, Target Groups, and TLS termination.


13. **Enter the basic ALB configuration**

   Enter `sportbooking-alb` as the name, select the **Internet-facing** scheme, and use the IPv4 address type.

   ![Enter the basic ALB configuration](/images/5-Workshop/5.5-Policy/alb-13.png)

   <p class="image-caption"><em>Figure 13: Enter the basic ALB configuration</em></p>
   The Internet-facing ALB receives user traffic while the EC2 application remains private.


14. **Select the VPC and public subnets**

   Select the SportBooking VPC and two public subnets in two AZs.

   ![Select the VPC and public subnets](/images/5-Workshop/5.5-Policy/alb-14.png)

   <p class="image-caption"><em>Figure 14: Select the VPC and public subnets</em></p>
   The ALB requires public subnets to receive Internet requests and multiple AZs for improved availability.


15. **Select the Security Group and listener**

   Select the ALB Security Group that allows ports 80 and 443. Create the initial HTTP:80 listener and configure it to forward to `sportbooking-tg`.

   ![Select the Security Group and listener](/images/5-Workshop/5.5-Policy/alb-15.png)

   <p class="image-caption"><em>Figure 15: Select the Security Group and listener</em></p>
   The ALB Security Group is the only public access point. The listener forwards requests to the application through the Target Group.


16. **Review and create the ALB**

   Review the VPC, subnets, Security Group, listener, and Target Group, then choose **Create load balancer**.

   ![Review and create the ALB](/images/5-Workshop/5.5-Policy/alb-16.png)

   <p class="image-caption"><em>Figure 16: Review and create the ALB</em></p>
   Validate the VPC, two public subnets, Security Group, listener, and Target Group before creating the ALB.


17. **Confirm that the ALB was created**

   Open the new ALB and record `sportbooking-alb-1198360694.us-east-1.elb.amazonaws.com` as its DNS name.

   ![Confirm that the ALB was created](/images/5-Workshop/5.5-Policy/alb-17.png)

   <p class="image-caption"><em>Figure 17: Confirm that the ALB was created</em></p>
   Test with the ALB DNS name before configuring the Route 53 Alias.


18. **Verify listeners and security**

   Open the listener and security tabs and confirm that the HTTP listener forwards to the correct Target Group.

   ![Verify ALB listeners and security](/images/5-Workshop/5.5-Policy/alb-18.png)

   <p class="image-caption"><em>Figure 18: Verify listeners and security</em></p>
   An incorrect listener action can make the ALB reachable from a browser while preventing requests from reaching the application.


19. **Test the website through the ALB**

   Open the ALB DNS name or the configured domain and confirm that the SportBooking home page loads.

   ![Test the website through the ALB](/images/5-Workshop/5.5-Policy/alb-19.png)

   <p class="image-caption"><em>Figure 19: Test the website through the ALB</em></p>
   This is end-to-end evidence of the path: user -> ALB -> Target Group -> EC2 application -> RDS/S3.



#### Auto Scaling Group

The Auto Scaling Group uses the Launch Template to create EC2 application instances in private application subnets and attaches those instances to the Target Group created earlier.

| Item | Configuration |
|---|---|
| Launch Template | `sportbooking-lt`, version `Default (1)`. |
| Subnets | Two private application subnets in two AZs. |
| Desired capacity | `2`. |
| Min/Max | Min `2`, Max `4`. |
| Target Group | `sportbooking-tg`. |
| Health check | EC2 and Elastic Load Balancing; `300`-second grace period. |
| Scaling policy | Target tracking on Average CPU utilization, target `60`, `300`-second warmup. |

{{% notice info %}}
Set both desired and minimum capacity to `2` so the ASG maintains two instances across two Availability Zones. Maximum capacity `4` leaves room for the target tracking policy to scale out when average CPU exceeds the target.
{{% /notice %}}

#### Auto Scaling Group Procedure

Use the completed Launch Template to deploy instances in private application subnets and register them automatically with the Target Group for health checking.

**Procedure:**

1. **Start creating the Auto Scaling Group**

   Open **EC2 > Auto Scaling Groups**, then choose **Create Auto Scaling group**.

   ![Start creating the Auto Scaling Group](/images/5-Workshop/5.5-Policy/auto-scaling-group-01.png)

   <p class="image-caption"><em>Figure 1: Start creating the Auto Scaling Group</em></p>
   The ASG maintains EC2 application capacity, replaces unhealthy instances, and supports rollouts through Instance Refresh.


2. **Select the Launch Template**

   Enter `sportbooking-asg` as the ASG name, select `sportbooking-lt` as the Launch Template, and select version `Default (1)`.

   ![Select the Launch Template](/images/5-Workshop/5.5-Policy/auto-scaling-group-02.png)

   <p class="image-caption"><em>Figure 2: Select the Launch Template</em></p>
   The ASG creates instances from the Launch Template. An incorrect version can cause instances to use outdated User Data or an outdated AMI.


3. **Confirm the template version**

   Review the AMI, instance type, key pair, and Security Group summary, then choose **Next**.

   ![Confirm the Launch Template version](/images/5-Workshop/5.5-Policy/auto-scaling-group-03.png)

   <p class="image-caption"><em>Figure 3: Confirm the template version</em></p>
   This is the final template check before the ASG creates instances in private subnets.


4. **Select the VPC and private application subnets**

   Select the SportBooking VPC and private application subnets in multiple AZs.

   ![Select the VPC and private application subnets](/images/5-Workshop/5.5-Policy/auto-scaling-group-04.png)

   <p class="image-caption"><em>Figure 4: Select the VPC and private application subnets</em></p>
   EC2 application instances do not need public IP addresses. Private subnet placement ensures that only the ALB can call the application.


5. **Configure advanced capacity options**

   Select **Balanced best effort** as the distribution strategy, retain **Default** for Capacity Reservation preference, and choose **Next**.

   ![Configure advanced capacity options](/images/5-Workshop/5.5-Policy/auto-scaling-group-05.png)

   <p class="image-caption"><em>Figure 5: Configure advanced capacity options</em></p>
   This deployment prioritizes a simple and stable configuration. A production environment can use mixed instances or Spot Instances to optimize cost.


6. **Attach load balancing**

   Enable load balancer integration, then attach the ASG to the existing `sportbooking-tg` Target Group.

   ![Attach load balancing](/images/5-Workshop/5.5-Policy/auto-scaling-group-06.png)

   <p class="image-caption"><em>Figure 6: Attach load balancing</em></p>
   The ASG must register EC2 instances with the Target Group so the ALB can forward traffic and perform health checks.


7. **Configure health checks**

   Enable Elastic Load Balancing health checks, set the health check grace period to `300` seconds, and choose **Next**.

   ![Configure health checks](/images/5-Workshop/5.5-Policy/auto-scaling-group-07.png)

   <p class="image-caption"><em>Figure 7: Configure health checks</em></p>
   A grace period that is too short can cause the ASG to replace an instance before the application finishes starting.


8. **Configure group size and scaling limits**

   Set desired capacity to `2`, minimum capacity to `2`, and maximum capacity to `4`.

   ![Configure group size and scaling limits](/images/5-Workshop/5.5-Policy/auto-scaling-group-08.png)

   <p class="image-caption"><em>Figure 8: Configure group size and scaling</em></p>
   Maintain two baseline instances while allowing the ASG to scale out to four instances.


9. **Configure the target tracking policy**

   Select **Target tracking scaling policy**, select **Average CPU utilization**, set the target value to `60`, and set instance warmup to `300` seconds.

   ![Configure the target tracking policy](/images/5-Workshop/5.5-Policy/auto-scaling-group-09.png)

   <p class="image-caption"><em>Figure 9: Configure target tracking for 60% average CPU</em></p>
   This policy adjusts desired capacity between two and four instances based on average CPU utilization across the Auto Scaling Group.


10. **Complete the Auto Scaling Group options**

   Keep deletion protection at **None (default)** and do not use a placement group. Review monitoring, warmup, and instance-maintenance options, then choose **Next**.

   ![Complete the Auto Scaling Group options](/images/5-Workshop/5.5-Policy/auto-scaling-group-10.png)

   <p class="image-caption"><em>Figure 10: Review maintenance options and continue</em></p>
   Complete these options before configuring notifications and tags.


11. **Skip notifications during ASG creation**

   No notification is added during ASG creation. Choose **Next**.

   ![Skip or add notifications](/images/5-Workshop/5.5-Policy/auto-scaling-group-11.png)

   <p class="image-caption"><em>Figure 11: Skip or add notifications</em></p>
   The SNS channel is created separately in section 5.7 and used as an action for the CloudWatch alarm.


12. **Add an ASG tag**

   Add a tag such as `Project=SportBooking`, then choose **Next**.

   ![Add an ASG tag](/images/5-Workshop/5.5-Policy/auto-scaling-group-12.png)

   <p class="image-caption"><em>Figure 12: Add a tag to the ASG</em></p>
   Tags support resource filtering, cost management, and identification of project instances.


13. **Review and create the ASG**

   Review the Launch Template, subnets, Target Group, capacity, and tags, then choose **Create Auto Scaling group**.

   ![Review and create the ASG](/images/5-Workshop/5.5-Policy/auto-scaling-group-13.png)

   <p class="image-caption"><em>Figure 13: Review and create the ASG</em></p>
   Verify the Launch Template, private subnets, Target Group, capacity, and tags before creating the ASG.


14. **Verify the ASG after creation**

   Open the new ASG and inspect its instances, desired capacity, Launch Template, and health status.

   ![Verify the ASG after creation](/images/5-Workshop/5.5-Policy/auto-scaling-group-14.png)

   <p class="image-caption"><em>Figure 14: Verify the ASG after creation</em></p>
   This provides evidence that the compute layer is running and ready to receive traffic through the Target Group and ALB.



#### Instance Refresh Procedure

When updating the application version, build the JAR again and overwrite the release object in S3:

```powershell
.\mvnw.cmd clean package -DskipTests

aws s3 cp target\SportBooingSystem-0.0.1-SNAPSHOT.jar `
  s3://sportbooking-artifacts-3stars-us-east-1/releases/SportBooingSystem-0.0.1-SNAPSHOT.jar `
  --region us-east-1
```

Then open **EC2 > Auto Scaling Groups > sportbooking-asg > Instance refresh > Start** and use:

- Use current Auto Scaling group configuration.
- Instance warmup: 300 seconds.
- Skip matching: Off.
- Monitor the Target Group until the replacement instances become `Healthy`.

{{% notice info %}}
When an empty Launch Template version caused an error, update the ASG to a valid version before restarting the refresh with the current ASG configuration.
{{% /notice %}}

