---
title : "Create S3 Buckets, Upload the Artifact, and Configure the Gateway Endpoint"
date : 2024-01-01
weight : 1
chapter : false
pre : " <b> 5.4.1 </b> "
---

#### Deployment Configuration

Create both S3 buckets before RDS, Secrets Manager, the IAM Role, and EC2 so the deployment artifact is available before User Data runs.

| Item | Artifact bucket | Upload bucket |
|---|---|---|
| Bucket name | `sportbooking-artifacts-3stars-us-east-1` | `sportbooking-uploads-3stars-us-east-1` |
| Prefix | `releases/` | `uploads/` |
| Data | Release JAR | Football-pitch images uploaded by users |
| Public access | Block Public Access: On | Block Public Access: On |
| EC2 permissions | `s3:GetObject` | `s3:GetObject`, `s3:PutObject`, `s3:DeleteObject` |

{{% notice warning %}}
Keep Block Public Access enabled for both buckets. The Spring Boot application reads images from S3 and serves them through the `/uploads/**` route; do not make the entire bucket public.
{{% /notice %}}

#### Procedure

1. **Open the S3 bucket creation workflow**

   Open **S3 > Buckets**, then choose **Create bucket**.

   ![Open the S3 bucket creation workflow](/images/5-Workshop/5.4-S3-onprem/s3-01.png)

   <p class="image-caption"><em>Figure 1: Open the S3 bucket creation workflow</em></p>

   S3 separates deployment artifacts and uploaded data from the EC2 lifecycle. Data remains independent when the Auto Scaling Group replaces an instance.

2. **Enter the artifact bucket name and Region**

   Enter `sportbooking-artifacts-3stars-us-east-1` and select `us-east-1`. Keep **Object Ownership** configured without public ACLs.

   ![Enter the artifact bucket name and Region](/images/5-Workshop/5.4-S3-onprem/s3-02.png)

   <p class="image-caption"><em>Figure 2: Enter the artifact bucket name and Region</em></p>

   Use the same bucket name consistently in the IAM policy, User Data, and operational commands in later sections.

3. **Keep Block Public Access enabled and create the bucket**

   Enable **Block all public access**, retain the default S3 encryption setting, and choose **Create bucket**.

   ![Keep Block Public Access enabled and create the artifact bucket](/images/5-Workshop/5.4-S3-onprem/s3-03.png)

   <p class="image-caption"><em>Figure 3: Keep Block Public Access enabled and create the artifact bucket</em></p>

   The release JAR does not require public access because EC2 retrieves the object through its IAM Role.

4. **Verify the artifact bucket**

   Open the new bucket, inspect the **Objects** tab, and confirm the successful bucket-creation message.

   ![Verify the artifact bucket after creation](/images/5-Workshop/5.4-S3-onprem/s3-04.png)

   <p class="image-caption"><em>Figure 4: Verify the artifact bucket after creation</em></p>

   After the artifact bucket is ready, repeat the workflow to create `sportbooking-uploads-3stars-us-east-1` and keep Block Public Access enabled.

5. **Package the release JAR**

   Open the `SportBookingSystem` source code in VS Code and run `.\mvnw.cmd -Pprod clean package`. Maven finishes with **BUILD SUCCESS** and creates `target\SportBooingSystem-0.0.1-SNAPSHOT.jar` for upload to S3.

   ![Package the SportBooking release JAR](/images/5-Workshop/5.4-S3-onprem/s3-artifact-upload-01.png)

   <p class="image-caption"><em>Figure 5: Package the SportBooking release JAR</em></p>

   This release build is used by the seed EC2 instance, Launch Template, and Auto Scaling Group.

6. **Open the artifact bucket**

   Open the S3 bucket list and select `sportbooking-artifacts-3stars-us-east-1`.

   ![Open the artifact bucket](/images/5-Workshop/5.4-S3-onprem/s3-artifact-upload-02.png)

   <p class="image-caption"><em>Figure 6: Open the artifact bucket</em></p>

   The S3 list confirms that the artifact and upload buckets both exist before EC2 is provisioned.

7. **Create the releases prefix**

   Open the artifact bucket, then create and open the `releases/` prefix.

   ![Create the releases prefix in the artifact bucket](/images/5-Workshop/5.4-S3-onprem/s3-artifact-upload-03.png)

   <p class="image-caption"><em>Figure 7: Create the releases prefix in the artifact bucket</em></p>

   This prefix separates release files from other objects and matches the S3 URI used by User Data.

8. **Start the artifact upload**

   Choose **Upload** inside the `releases/` prefix.

   ![Start uploading the release JAR](/images/5-Workshop/5.4-S3-onprem/s3-artifact-upload-04.png)

   <p class="image-caption"><em>Figure 8: Start uploading the release JAR</em></p>

9. **Select the local JAR**

   Choose **Add files**, then select the JAR that was built in the `target` directory.

   ![Select the JAR to upload](/images/5-Workshop/5.4-S3-onprem/s3-artifact-upload-05.png)

   <p class="image-caption"><em>Figure 9: Select the JAR to upload</em></p>

10. **Confirm the upload destination**

   Verify that the destination is `sportbooking-artifacts-3stars-us-east-1/releases/`, then choose **Upload**.

   ![Confirm the artifact upload destination](/images/5-Workshop/5.4-S3-onprem/s3-artifact-upload-06.png)

   <p class="image-caption"><em>Figure 10: Confirm the artifact upload destination</em></p>

   Use this exact path in the Launch Template's `APP_ARTIFACT_S3_URI` variable.

11. **Verify the completed artifact upload**

   Confirm that the upload completed and that the JAR appears under the `releases/` prefix.

   ![Verify that the artifact upload completed](/images/5-Workshop/5.4-S3-onprem/s3-artifact-upload-07.png)

   <p class="image-caption"><em>Figure 11: Verify that the artifact upload completed</em></p>

#### Configure the S3 Gateway Endpoint

After the VPC and S3 buckets exist, create a Gateway Endpoint so EC2 instances in private application subnets can access S3 through the AWS network:

1. Open **VPC > Endpoints** and choose **Create endpoint**.
2. Enter `sportbooking-vpce-s3` as the name and select the `com.amazonaws.us-east-1.s3` service with type **Gateway**.
3. Select `sportbooking-vpc` and associate the endpoint with the route tables for both private application subnets.
4. Use an endpoint policy that allows the required operations; continue to restrict object access through the IAM Role and bucket policy.
5. Verify that the endpoint is **Available** before creating EC2 resources.

{{% notice info %}}
An S3 Gateway Endpoint provides a private network path to S3 but does not replace IAM. EC2 still requires `s3:GetObject` on the artifact bucket and the appropriate read/write permissions on the upload bucket.
{{% /notice %}}

#### Verification Commands

```powershell
aws s3 ls s3://sportbooking-artifacts-3stars-us-east-1/releases/ --region us-east-1
aws s3 ls s3://sportbooking-uploads-3stars-us-east-1/uploads/ --region us-east-1
```

The verification confirms that the release JAR is present in S3 before the remaining foundational services and EC2 layer are created.

