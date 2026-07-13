---
title : "Create the EC2 IAM Role"
date : 2024-01-01
weight : 4
chapter : false
pre : " <b> 5.4.4 </b> "
---

#### Deployment Scope

Create an IAM Role that allows the EC2 application to read the artifact from S3, read and write uploaded files, retrieve configuration from Secrets Manager, and connect through Systems Manager Session Manager. This removes the need to store access keys directly on EC2.

#### Role Configuration

| Item | Configuration value |
|---|---|
| Trusted entity | AWS service: EC2 |
| Role name | `sportbooking-ec2-role` |
| Managed policy | `AmazonSSMManagedInstanceCore` |
| Inline policy | Read the S3 artifact, read/write S3 uploads, and read the secret |
| Attached to EC2 | Through the IAM instance profile in the Launch Template |

#### Application Inline Policy

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "ReadArtifactJarFromS3",
      "Effect": "Allow",
      "Action": ["s3:GetObject"],
      "Resource": ["arn:aws:s3:::sportbooking-artifacts-3stars-us-east-1/releases/*"]
    },
    {
      "Sid": "AccessUploadFilesInS3",
      "Effect": "Allow",
      "Action": ["s3:GetObject", "s3:PutObject", "s3:DeleteObject"],
      "Resource": ["arn:aws:s3:::sportbooking-uploads-3stars-us-east-1/uploads/*"]
    },
    {
      "Sid": "ReadAppEnvSecret",
      "Effect": "Allow",
      "Action": ["secretsmanager:GetSecretValue"],
      "Resource": ["arn:aws:secretsmanager:us-east-1:<account-id>:secret:sportbooking/prod/app-env*"]
    }
  ]
}
```

{{% notice info %}}
Create the IAM Role after the S3 buckets and secret. This allows the custom policy to be checked directly against the artifact bucket, upload bucket, and `sportbooking/prod/app-env` ARNs before it is attached to the role.
{{% /notice %}}

#### Procedure

Start with an EC2 trust relationship, attach the SSM policy, and then attach a policy that limits access to the required S3 and Secrets Manager resources.

**Procedure:**

1. **Start creating the IAM Role**

   Open **IAM > Roles**, then choose **Create role**.

   ![Start creating the IAM Role](/images/5-Workshop/5.4-S3-onprem/iam-role-01.png)

   <p class="image-caption"><em>Figure 1: Start creating the IAM Role</em></p>
   EC2 requires permission to read S3, read Secrets Manager, and use Session Manager without storing access keys on the instance.


2. **Select EC2 as the trusted entity**

   Select **AWS service**, choose the **EC2** use case, and choose **Next**.

   ![Select EC2 as the trusted entity](/images/5-Workshop/5.4-S3-onprem/iam-role-02.png)

   <p class="image-caption"><em>Figure 2: Select EC2 as the trusted entity</em></p>
   The trusted entity determines which service can assume the role. Selecting EC2 allows instances to receive temporary credentials through the instance profile.


3. **Find the required managed policy**

   Search for the SSM policy under **Add permissions**, then select **AmazonSSMManagedInstanceCore**.

   ![Find the required managed policy](/images/5-Workshop/5.4-S3-onprem/iam-role-03.png)

   <p class="image-caption"><em>Figure 3: Find the required managed policy</em></p>
   The SSM policy supports private EC2 administration through Session Manager without opening public SSH.


4. **Create the application permissions policy**

   Open the policy creation workflow or add an inline policy. Enter the JSON permissions for reading the S3 artifact, reading and writing S3 uploads, and reading the secret.

   ![Create the application permissions policy](/images/5-Workshop/5.4-S3-onprem/iam-role-04.png)

   <p class="image-caption"><em>Figure 4: Create the application permissions policy</em></p>
   Grant access only to the SportBooking buckets, prefixes, and secret; do not grant `s3:*` or `secretsmanager:*` across the entire account.


5. **Name and create the policy**

   Enter `sportbooking-ec2-app-policy` as the policy name, review the permissions, and choose **Create policy**.

   ![Name and create the policy](/images/5-Workshop/5.4-S3-onprem/iam-role-05.png)

   <p class="image-caption"><em>Figure 5: Name and create the policy</em></p>
   A descriptive name makes the policy's purpose clear during an audit.


6. **Return to permission selection**

   Refresh the policy list after creating the new policy, then search for `sportbooking-ec2-app-policy`.

   ![Return to permission selection](/images/5-Workshop/5.4-S3-onprem/iam-role-06.png)

   <p class="image-caption"><em>Figure 6: Return to permission selection</em></p>
   The new policy appears after the list refresh and can be selected for the role.


7. **Attach the policies to the role**

   Select the custom policy, confirm that both the SSM and application policies are selected, and choose **Next**.

   ![Attach the policies to the role](/images/5-Workshop/5.4-S3-onprem/iam-role-07.png)

   <p class="image-caption"><em>Figure 7: Attach the policies to the role</em></p>
   The role requires both SSM operational permissions and application runtime permissions.


8. **Name the role**

   Enter `sportbooking-ec2-role` as the role name and add a description of its purpose.

   ![Name the IAM Role](/images/5-Workshop/5.4-S3-onprem/iam-role-08.png)

   <p class="image-caption"><em>Figure 8: Name the role</em></p>
   This role is selected as the IAM instance profile in the Launch Template and remains separate from roles used by other services.


9. **Review and create the role**

   Verify that the trust policy specifies EC2, review the permission policies, and choose **Create role**.

   ![Review and create the IAM Role](/images/5-Workshop/5.4-S3-onprem/iam-role-09.png)

   <p class="image-caption"><em>Figure 9: Review and create the role</em></p>
   The final review confirms that EC2 is the trusted entity and that the role has both operational and application permissions.


10. **Confirm that the role is ready**

   Find `sportbooking-ec2-role` in the role list, open it, and verify its permissions.

   ![Confirm that the IAM Role is ready](/images/5-Workshop/5.4-S3-onprem/iam-role-10.png)

   <p class="image-caption"><em>Figure 10: Confirm that the role is ready</em></p>
   The role is attached to the Launch Template so every EC2 instance in the Auto Scaling Group receives the required permissions at startup.

