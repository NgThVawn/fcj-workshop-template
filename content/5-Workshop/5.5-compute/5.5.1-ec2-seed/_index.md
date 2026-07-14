---
title : "Seed EC2 Instance"
date : 2024-01-01
weight : 1
chapter : false
pre : " <b> 5.5.1 </b> "
---

#### Seed EC2 Configuration

Create the seed EC2 instance after `releases/SportBooingSystem-0.0.1-SNAPSHOT.jar` exists in the artifact bucket. Use this instance to verify Amazon Linux, the Java runtime, Session Manager, the IAM Role, and access to foundational services before standardizing the configuration in a Launch Template.

#### Seed EC2 Deployment Procedure

Provision the seed instance only after S3, IAM, RDS, and Secrets Manager are ready so the validation has the required artifact, permissions, and application configuration.

**Procedure:**

1. **Start creating the seed EC2 instance**

   Open **EC2 > Instances**, then choose **Launch instances**.

   ![Start creating the seed EC2 instance](/images/5-Workshop/5.5-compute/ec2-seed-01.png)

   <p class="image-caption"><em>Figure 1: Start creating the seed EC2 instance</em></p>
   The seed instance validates the runtime, IAM, networking, and application behavior before an AMI is created for the Launch Template.


2. **Name the instance and select an AMI**

   Enter a project-specific instance name, then select Amazon Linux 2023.

   ![Name the instance and select an AMI](/images/5-Workshop/5.5-compute/ec2-seed-02.png)

   <p class="image-caption"><em>Figure 2: Name the instance and select an AMI</em></p>
   The name distinguishes the seed instance from EC2 instances launched by the ASG. The AMI must support the application's Java runtime.


3. **Confirm the AMI and select the instance type**

   Confirm the selected AMI, then select `t3.small` as the instance type.

   ![Confirm the AMI and select the instance type](/images/5-Workshop/5.5-compute/ec2-seed-03.png)

   <p class="image-caption"><em>Figure 3: Confirm the AMI and select the instance type</em></p>
   The seed instance uses `t3.small`, which is sufficient for validating the project runtime and application.


4. **Configure the key pair and network**

   Select **Proceed without a key pair** because Session Manager administers the instance. Select `sportbooking-vpc` and the `sportbooking-app-a` private subnet, then disable **Auto-assign public IP**.

   ![Configure the key pair and network](/images/5-Workshop/5.5-compute/ec2-seed-04.png)

   <p class="image-caption"><em>Figure 4: Configure the key pair and network</em></p>
   The instance has no public IP address and uses no key pair; all administrative sessions use Session Manager.


5. **Attach the Security Group and configure storage**

   Select the `sportbooking-app` Security Group, then configure a `20 GiB` `gp3` root volume.

   ![Attach the Security Group and configure storage](/images/5-Workshop/5.5-compute/ec2-seed-05.png)

   <p class="image-caption"><em>Figure 5: Attach the Security Group and configure storage</em></p>
   The Security Group accepts application traffic only from the ALB. The root volume stores the runtime, configuration file, and artifact retrieved from S3.


6. **Attach the IAM instance profile**

   Open **Advanced details**, select `sportbooking-ec2-role` as the IAM Role, and choose **Launch instance**.

   ![Attach the IAM instance profile](/images/5-Workshop/5.5-compute/ec2-seed-06.png)

   <p class="image-caption"><em>Figure 6: Attach the IAM instance profile</em></p>
   The role allows the instance to read the S3 artifact, read the secret, access the upload bucket, and use Session Manager without storing access keys.


7. **Verify the instance after launch**

   Open the new instance and wait for the `Running` state and successful status checks.

   ![Verify the instance after launch](/images/5-Workshop/5.5-compute/ec2-seed-07.png)

   <p class="image-caption"><em>Figure 7: Verify the instance after launch</em></p>
   Open a connection only after the instance reaches `Running` and passes its status checks.


8. **Choose Connect**

   Select the instance, then choose **Connect**.

   ![Choose Connect for the seed instance](/images/5-Workshop/5.5-compute/ec2-seed-08.png)

   <p class="image-caption"><em>Figure 8: Choose Connect</em></p>
   The session is used to verify IAM permissions, load the secret, start the application, and test local HTTP.


9. **Connect through Session Manager**

   Select the **Session Manager** tab, then choose **Connect**.

   ![Connect through Session Manager](/images/5-Workshop/5.5-compute/ec2-seed-09.png)

   <p class="image-caption"><em>Figure 9: Connect through Session Manager</em></p>
   Session Manager is appropriate for private EC2 instances because it requires neither inbound SSH nor a bastion host.


10. **Verify the EC2 terminal**

   Read the `sportbooking/prod/app-env` secret and create `/opt/sportbooking/app.env`. Restart the `sportbooking` service and run `curl -I http://127.0.0.1:8080/`. Record the `HTTP/1.1 200` response after the service starts successfully.

   ![Verify the EC2 terminal](/images/5-Workshop/5.5-compute/ec2-seed-10.png)

   <p class="image-caption"><em>Figure 10: Verify the EC2 terminal</em></p>
   The terminal confirms that EC2 has the correct IAM permissions, network access, and runtime before Launch Template automation is introduced.
