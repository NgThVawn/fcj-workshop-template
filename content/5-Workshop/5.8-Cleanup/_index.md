---
title : "Resource Cleanup"
date : 2024-01-01
weight : 8
chapter : false
pre : " <b> 5.8. </b> "
---

#### Cleanup Scope

After deployment, testing, and evidence collection were completed, the SportBooking resources were decommissioned in dependency order. The artifact bucket and the compute and network layers were removed. The upload bucket, final RDS snapshot, and retained automated backup were intentionally preserved to protect data required for recovery.

{{% notice warning %}}
Deleting the Auto Scaling Group, RDS database, S3 bucket, VPC, and IAM role can cause data loss or make the website completely unavailable. Perform these actions only after the required data has been backed up, the report has been completed, and the environment is confirmed to be no longer in use.
{{% /notice %}}

#### Cleanup Order

| Phase | Resources | Dependency |
|---|---|---|
| 1 | Route 53, CloudWatch, and SNS | Stop directing users to the application; delete the alarm before the SNS topic. |
| 2 | ASG, ALB, Target Group, and Launch Template | Delete the ASG before the Launch Template; delete the ALB before the Target Group. |
| 3 | IAM, RDS, S3, Secrets Manager, and ACM | EC2 has stopped; ACM is no longer used by the ALB; no CloudFront distribution was created. |
| 4 | NAT Gateway, Elastic IP, and VPC Endpoint | Wait until each NAT Gateway reaches `Deleted` before releasing its Elastic IP. |
| 5 | Security Group, DB Subnet Group, and VPC | The ALB, EC2 instances, RDS database, endpoint, and Security Group references have been removed. |
| 6 | Remaining-resource review | No snapshot, backup, log group, hosted zone, or policy remains outside the retention plan. |

{{% notice info %}}
Verify the correct AWS account and Region `us-east-1` before every action. Select only resources named for the SportBooking project; do not delete default or shared resources.
{{% /notice %}}

#### Preparation Before Deletion

1. Identify the final RDS snapshot, retained automated backup, and data in the upload bucket that must be preserved.
2. Stop the Alias record `sport.younglilliu.id.vn` from routing to the ALB before deleting the load-balancing layer.
3. Confirm that no CloudFront distribution was created, so no CloudFront dependency needs to be removed.
4. Retain the public hosted zone because the domain remains managed; remove only records that are no longer used.
5. Record every retained storage resource so that it is not mistaken for an overlooked resource.

### Monitoring Resource Cleanup

#### 1. Delete the CloudWatch Alarm

The CloudWatch alarm was deleted before SNS so that no alarm action continued to reference the notification topic.

**Procedure:**

1. **Select the alarm**

   On the console, perform the following in order: **(1)** select the `SB-ALB-UnhealthyTargets` alarm; **(2)** open **Actions**; and **(3)** select **Delete**.

   ![Select the CloudWatch alarm to delete](/images/5-Workshop/5.8-Cleanup/cleanup-cloudwatch-alarm-01.png)

   <p class="image-caption"><em>Figure 1: Select the CloudWatch alarm to delete</em></p>
   This alarm monitors the SportBooking Target Group and sends notifications to SNS. Deleting the alarm first removes the monitoring link before the source resources and notification channel are deleted.


2. **Confirm alarm deletion**

   Verify the name `SB-ALB-UnhealthyTargets`, then select **Delete**.

   ![Confirm CloudWatch alarm deletion](/images/5-Workshop/5.8-Cleanup/cleanup-cloudwatch-alarm-02.png)

   <p class="image-caption"><em>Figure 2: Confirm CloudWatch alarm deletion</em></p>
   This action removes only the alarm configuration; it does not delete the historical metrics of the source service.


#### 2. Delete the SNS Subscription

The subscription was deleted before the topic to explicitly disconnect the email endpoint from the alert channel.

**Procedure:**

1. **Select the subscription**

   On the console, perform the following in order: **(1)** open **SNS > Subscriptions** and select the confirmed project subscription; and **(2)** select **Delete**.

   ![Select the SNS subscription to delete](/images/5-Workshop/5.8-Cleanup/cleanup-sns-subscription-01.png)

   <p class="image-caption"><em>Figure 3: Select the SNS subscription to delete</em></p>
   The subscription connects the email address that receives alerts to the `sportbooking-prod-alerts` topic.


2. **Confirm subscription deletion**

   Verify the endpoint and subscription ID, then select **Delete**.

   ![Confirm SNS subscription deletion](/images/5-Workshop/5.8-Cleanup/cleanup-sns-subscription-02.png)

   <p class="image-caption"><em>Figure 4: Confirm SNS subscription deletion</em></p>
   After this step, the email endpoint no longer receives notifications from the topic.


3. **Verify the result**

   Refresh the subscription list. Confirm the **Subscription deleted successfully** message and verify that the project subscription no longer appears.

   ![Verify that the SNS subscription has been deleted](/images/5-Workshop/5.8-Cleanup/cleanup-sns-subscription-03.png)

   <p class="image-caption"><em>Figure 5: Verify that the SNS subscription has been deleted</em></p>
   This verification prevents an unused endpoint from remaining in the account.


#### 3. Delete the SNS Topic

**Procedure:**

1. **Select the alert topic**

   On the console, perform the following in order: **(1)** open **SNS > Topics** and select `sportbooking-prod-alerts`; and **(2)** select **Delete**.

   ![Select the SNS topic to delete](/images/5-Workshop/5.8-Cleanup/cleanup-sns-topic-01.png)

   <p class="image-caption"><em>Figure 6: Select the SNS topic to delete</em></p>
   The CloudWatch alarm and subscription have already been removed, so the topic has no remaining dependency within the project scope.


2. **Confirm topic deletion**

   On the console, perform the following in order: **(1)** enter `delete me` as required; and **(2)** select **Delete**.

   ![Confirm SNS topic deletion](/images/5-Workshop/5.8-Cleanup/cleanup-sns-topic-02.png)

   <p class="image-caption"><em>Figure 7: Confirm SNS topic deletion</em></p>
   Topic deletion cannot be undone. The confirmation text reduces the risk of deleting the wrong notification channel.


3. **Verify that the topic has been deleted**

   Confirm the **Topic deleted successfully** message, then verify that the project topic no longer appears in the list.

   ![Verify that the SNS topic has been deleted](/images/5-Workshop/5.8-Cleanup/cleanup-sns-topic-03.png)

   <p class="image-caption"><em>Figure 8: Verify that the SNS topic has been deleted</em></p>
   This result completes the cleanup of the SNS alert channel.


### Application and Load-Balancing Cleanup

#### 4. Scale In and Delete the Auto Scaling Group

The Auto Scaling Group had to be processed before the Launch Template. Reducing capacity to `0` allowed the EC2 instances to terminate in a controlled manner before the scaling configuration was deleted.

**Procedure:**

1. **Open the capacity settings**

   On the console, perform the following in order: **(1)** select `sportbooking-asg`; and **(2)** select **Edit** in **Capacity overview**.

   ![Open the Auto Scaling Group capacity settings](/images/5-Workshop/5.8-Cleanup/cleanup-asg-01.png)

   <p class="image-caption"><em>Figure 9: Open the Auto Scaling Group capacity settings</em></p>
   The ASG is maintaining two healthy EC2 instances. Without reducing capacity, it continues to launch a replacement whenever an instance is terminated.


2. **Set capacity to 0**

   On the console, perform the following in order: **(1)** set **Desired capacity** to `0`; **(2)** set **Min desired capacity** to `0`; and **(3)** select **Update**.

   ![Set the Auto Scaling Group capacity to 0](/images/5-Workshop/5.8-Cleanup/cleanup-asg-02.png)

   <p class="image-caption"><em>Figure 10: Set the Auto Scaling Group capacity to 0</em></p>
   Setting both desired and minimum capacity to `0` allows the ASG to terminate every EC2 instance without launching a replacement. Maximum capacity can remain unchanged because the ASG is deleted in a later step.


3. **Verify that the ASG has no instances**

   Refresh the page until the **Instances** column shows `0`. Verify that **Desired capacity** and the minimum limit are also `0`.

   ![Verify that the Auto Scaling Group has scaled down to 0](/images/5-Workshop/5.8-Cleanup/cleanup-asg-03.png)

   <p class="image-caption"><em>Figure 11: Verify that the Auto Scaling Group has scaled down to 0</em></p>
   Delete the ASG only after its EC2 instances have terminated to reduce the risk of leaving an ENI or EBS volume in use.


4. **Start deleting the ASG**

   On the console, perform the following in order: **(1)** open **Actions**; and **(2)** select **Delete**.

   ![Start deleting the Auto Scaling Group](/images/5-Workshop/5.8-Cleanup/cleanup-asg-04.png)

   <p class="image-caption"><em>Figure 12: Start deleting the Auto Scaling Group</em></p>
   With no remaining instances, the ASG can be deleted without affecting data on a running EC2 instance.


5. **Confirm ASG deletion**

   On the console, perform the following in order: **(1)** enter `delete` in the confirmation field; and **(2)** select **Delete**.

   ![Confirm Auto Scaling Group deletion](/images/5-Workshop/5.8-Cleanup/cleanup-asg-05.png)

   <p class="image-caption"><em>Figure 13: Confirm Auto Scaling Group deletion</em></p>
   After the ASG is deleted, the Launch Template is no longer referenced by the automatic scaling configuration.


{{% notice tip %}}
After deleting the ASG, also check **EC2 > Instances** and **Elastic Block Store > Volumes**. Retain only volumes or snapshots explicitly identified as required backups.
{{% /notice %}}

#### 5. Delete the Application Load Balancer

The ALB had to be deleted before the Target Group because its listeners referenced the Target Group.

**Procedure:**

1. **Select and delete the ALB**

   On the console, perform the following in order: **(1)** open **EC2 > Load Balancers** and select `sportbooking-alb`; **(2)** open **Actions**; and **(3)** select **Delete load balancer**, then confirm in the displayed dialog.

   ![Select the Application Load Balancer to delete](/images/5-Workshop/5.8-Cleanup/cleanup-alb-01.png)

   <p class="image-caption"><em>Figure 14: Select the Application Load Balancer to delete</em></p>
   Deleting the ALB also removes its HTTP and HTTPS listeners and releases the association with the ACM certificate. Wait until the ALB disappears from the list before deleting the Target Group.


#### 6. Delete the Target Group

**Procedure:**

1. **Select the Target Group**

   On the console, perform the following in order: **(1)** open **EC2 > Target Groups** and select `sportbooking-tg`; **(2)** open **Actions**; and **(3)** select **Delete**.

   ![Select the Target Group to delete](/images/5-Workshop/5.8-Cleanup/cleanup-target-group-01.png)

   <p class="image-caption"><em>Figure 15: Select the Target Group to delete</em></p>
   The Target Group can be deleted only after the ALB and listeners no longer reference it. If AWS reports that the resource is still in use, check for a remaining ALB or listener rule.


2. **Confirm Target Group deletion**

   Verify the name `sportbooking-tg`, then select **Delete**.

   ![Confirm Target Group deletion](/images/5-Workshop/5.8-Cleanup/cleanup-target-group-02.png)

   <p class="image-caption"><em>Figure 16: Confirm Target Group deletion</em></p>
   The target EC2 instances terminated when ASG capacity was reduced to `0`, so the Target Group has no backend to maintain.


#### 7. Delete the Launch Template

**Procedure:**

1. **Select the Launch Template**

   On the console, perform the following in order: **(1)** open **EC2 > Launch Templates** and select `sportbooking-lt`; **(2)** open **Actions**; and **(3)** select **Delete template**.

   ![Select the Launch Template to delete](/images/5-Workshop/5.8-Cleanup/cleanup-launch-template-01.png)

   <p class="image-caption"><em>Figure 17: Select the Launch Template to delete</em></p>
   The Launch Template is deleted after the ASG so that no configuration continues to reference the template or any of its versions.


2. **Confirm Launch Template deletion**

   On the console, perform the following in order: **(1)** enter `Delete` with the exact capitalization requested on the screen; and **(2)** select **Delete**.

   ![Confirm Launch Template deletion](/images/5-Workshop/5.8-Cleanup/cleanup-launch-template-02.png)

   <p class="image-caption"><em>Figure 18: Confirm Launch Template deletion</em></p>
   Deleting the template removes every version of `sportbooking-lt` and cannot be undone.


#### 8. Detach Policies and Delete the IAM Role

The IAM role was deleted only after all EC2 instances had terminated and no instance profile remained in use.

**Procedure:**

1. **Select the policies attached to the role**

   On the console, perform the following in order: **(1)** open the `sportbooking-ec2-role` role and select every attached policy; and **(2)** select **Remove**.

   ![Select the policies to detach from the IAM role](/images/5-Workshop/5.8-Cleanup/cleanup-iam-role-01.png)

   <p class="image-caption"><em>Figure 19: Select the policies to detach from the IAM role</em></p>
   The role shown in the figure has AWS managed policies and the `sportbooking-ec2-policy` customer managed policy attached. These associations must be removed before the role is deleted.


2. **Confirm policy removal**

   Verify the number of selected policies, then select **Remove**.

   ![Confirm policy removal from the IAM role](/images/5-Workshop/5.8-Cleanup/cleanup-iam-role-02.png)

   <p class="image-caption"><em>Figure 20: Confirm policy removal from the IAM role</em></p>
   AWS managed policies are detached from the role but are not deleted from the AWS account.


3. **Select the role to delete**

   On the console, perform the following in order: **(1)** return to **IAM > Roles** and select `sportbooking-ec2-role`; and **(2)** select **Delete**.

   ![Select the IAM role to delete](/images/5-Workshop/5.8-Cleanup/cleanup-iam-role-03.png)

   <p class="image-caption"><em>Figure 21: Select the IAM role to delete</em></p>
   Select only the project role. Do not delete a service-linked role or a role shared with another resource.


4. **Confirm role deletion**

   Enter `sportbooking-ec2-role` exactly, then select **Delete**.

   ![Confirm IAM role deletion](/images/5-Workshop/5.8-Cleanup/cleanup-iam-role-04.png)

   <p class="image-caption"><em>Figure 22: Confirm IAM role deletion</em></p>
   Deleting the role removes the permissions that EC2 used to access S3, Secrets Manager, Systems Manager, and CloudWatch.


{{% notice info %}}
The `sportbooking-ec2-policy` customer managed policy remains after it is detached. If the policy is used only by this project, open **IAM > Policies**, verify that it has no attached entity, and delete the policy together with its non-default versions.
{{% /notice %}}

### Data and Application Configuration Cleanup

#### 9. Delete the RDS MySQL Database

**Procedure:**

1. **Start deleting the DB instance**

   On the console, perform the following in order: **(1)** open **RDS > Databases** and select `sportbooking-mysql`; **(2)** open **Actions**; and **(3)** select **Delete**.

   ![Select the RDS MySQL database to delete](/images/5-Workshop/5.8-Cleanup/cleanup-rds-01.png)

   <p class="image-caption"><em>Figure 23: Select the RDS MySQL database to delete</em></p>
   The EC2 instances have terminated, so no application connection continues to use the database.


2. **Configure data retention and confirm**

   Select **Create final snapshot** and enter `sportbooking-mysql-snapshot` as the snapshot name. Select **Retain automated backups** to preserve recovery capability. Finally, **(1)** enter `delete me`; and **(2)** select **Delete**.

   ![Configure the snapshot and confirm RDS deletion](/images/5-Workshop/5.8-Cleanup/cleanup-rds-02.png)

   <p class="image-caption"><em>Figure 24: Configure the snapshot and confirm RDS deletion</em></p>
   The `sportbooking-mysql-snapshot` final snapshot and automated backup were retained according to the post-deletion storage plan.


3. **Verify that the DB instance has been deleted**

   Wait for the deletion process to finish, then confirm the **Successfully deleted DB instance sportbooking-mysql** message.

   ![Verify that the RDS MySQL database has been deleted](/images/5-Workshop/5.8-Cleanup/cleanup-rds-03.png)

   <p class="image-caption"><em>Figure 25: Verify that the RDS MySQL database has been deleted</em></p>
   The DB instance no longer incurs compute charges, but the selected snapshot and retained backup remain storage resources that require management.


{{% notice warning %}}
The final snapshot and retained automated backup can continue to incur storage charges after the DB instance is deleted. Monitor both resources according to the agreed retention period.
{{% /notice %}}

#### 10. Empty and Delete the S3 Bucket

The artifact bucket was emptied before deletion. The upload bucket was retained during this cleanup to preserve images uploaded by users.

**Procedure:**

1. **Select the bucket and start emptying it**

   On the console, perform the following in order: **(1)** select the `sportbooking-artifacts-3stars-us-east-1` bucket; and **(2)** select **Empty**.

   ![Select the S3 artifact bucket to empty](/images/5-Workshop/5.8-Cleanup/cleanup-s3-01.png)

   <p class="image-caption"><em>Figure 26: Select the S3 artifact bucket to empty</em></p>
   S3 does not allow a bucket to be deleted while it contains objects, object versions, or delete markers.


2. **Confirm that the bucket will be emptied**

   On the console, perform the following in order: **(1)** enter `permanently delete`; and **(2)** select **Empty**.

   ![Confirm that the S3 bucket will be emptied](/images/5-Workshop/5.8-Cleanup/cleanup-s3-02.png)

   <p class="image-caption"><em>Figure 27: Confirm that the S3 bucket will be emptied</em></p>
   Emptying the bucket permanently removes its data unless another backup is available.


3. **Verify that the bucket is empty**

   Confirm the **Successfully emptied bucket** status. Review the counts of successfully deleted and failed objects.

   ![Verify that the S3 bucket has been emptied](/images/5-Workshop/5.8-Cleanup/cleanup-s3-03.png)

   <p class="image-caption"><em>Figure 28: Verify that the S3 bucket has been emptied</em></p>
   In the recorded result, two objects totaling 82.5 MB were deleted and no object failed.


4. **Select the bucket for deletion**

   On the console, perform the following in order: **(1)** return to the bucket list and select the artifact bucket that was just emptied; and **(2)** select **Delete**.

   ![Select the S3 artifact bucket for deletion](/images/5-Workshop/5.8-Cleanup/cleanup-s3-04.png)

   <p class="image-caption"><em>Figure 29: Select the S3 artifact bucket for deletion</em></p>
   Delete only the project bucket. A `cf-templates-...` bucket may be used by CloudFormation or another environment and must not be deleted without verification.


5. **Confirm the bucket name**

   On the console, perform the following in order: **(1)** enter the exact bucket name; and **(2)** select **Delete bucket**.

   ![Confirm S3 artifact bucket deletion](/images/5-Workshop/5.8-Cleanup/cleanup-s3-05.png)

   <p class="image-caption"><em>Figure 30: Confirm S3 artifact bucket deletion</em></p>
   S3 bucket names are globally unique. After deletion, the same name is not guaranteed to remain available for reuse.


6. **Verify that the bucket has been deleted**

   Confirm the **Successfully deleted bucket** message, then verify that the artifact bucket no longer appears in the list.

   ![Verify that the S3 artifact bucket has been deleted](/images/5-Workshop/5.8-Cleanup/cleanup-s3-06.png)

   <p class="image-caption"><em>Figure 31: Verify that the S3 artifact bucket has been deleted</em></p>
   The `sportbooking-uploads-3stars-us-east-1` bucket remains in the list after the artifact bucket is deleted. This bucket is intentionally retained and is not an overlooked resource.


#### 11. Schedule Secret Deletion

**Procedure:**

1. **Select the secret for deletion**

   Open the `sportbooking/prod/app-env` secret. Then **(1)** open **Actions**; and **(2)** select **Delete secret**.

   ![Select the secret to delete](/images/5-Workshop/5.8-Cleanup/cleanup-secret-01.png)

   <p class="image-caption"><em>Figure 32: Select the secret to delete</em></p>
   The EC2 instances and IAM role have been deleted, so no workload within the project scope continues to read the secret.


2. **Set the recovery window and schedule deletion**

   On the console, perform the following in order: **(1)** set **Waiting period** to `7` days or the period required by the retention policy; and **(2)** select **Schedule deletion**.

   ![Schedule deletion of the Secrets Manager secret](/images/5-Workshop/5.8-Cleanup/cleanup-secret-02.png)

   <p class="image-caption"><em>Figure 33: Schedule deletion of the Secrets Manager secret</em></p>
   Secrets Manager does not delete the secret immediately. It places the secret in a recovery window during which the secret is disabled and can be restored if the deletion was accidental.


{{% notice tip %}}
Do not include secret values in screenshots or report content. If redeployment is required during the recovery window, restore the secret before updating it instead of creating a duplicate name.
{{% /notice %}}

#### 12. Delete the ACM Certificate

Delete the ACM certificate only when the **In use** column displays **No**. If an ALB or CloudFront distribution still uses the certificate, remove that association first.

**Procedure:**

1. **Select the certificate**

   On the console, perform the following in order: **(1)** select the certificate for `sport.younglilliu.id.vn` and verify **In use = No**; and **(2)** open **More actions** and select **Delete**.

   ![Select the ACM certificate to delete](/images/5-Workshop/5.8-Cleanup/cleanup-acm-01.png)

   <p class="image-caption"><em>Figure 34: Select the ACM certificate to delete</em></p>
   The ALB has been deleted, so its HTTPS listener no longer holds the certificate. No CloudFront distribution was created, so the certificate has no additional CloudFront dependency.


2. **Confirm certificate deletion**

   On the console, perform the following in order: **(1)** enter `delete`; and **(2)** select **Delete**.

   ![Confirm ACM certificate deletion](/images/5-Workshop/5.8-Cleanup/cleanup-acm-02.png)

   <p class="image-caption"><em>Figure 35: Confirm ACM certificate deletion</em></p>
   After deletion, the certificate can no longer secure HTTPS connections and cannot be restored.


3. **Verify that the certificate has been deleted**

   Confirm the **Successfully deleted certificate** message, then verify that the project certificate no longer appears in the list.

   ![Verify that the ACM certificate has been deleted](/images/5-Workshop/5.8-Cleanup/cleanup-acm-03.png)

   <p class="image-caption"><em>Figure 36: Verify that the ACM certificate has been deleted</em></p>
   The certificate-validation CNAME record can now be removed from Route 53 if no other certificate uses it.


### Network Layer Cleanup

#### 13. Delete the NAT Gateways

Both NAT Gateways were deleted before the Elastic IPs and VPC. Each NAT Gateway had to reach `Deleted` before its Elastic IP was no longer associated.

**Procedure:**

1. **Select the NAT Gateway in the first Availability Zone**

   On the console, perform the following in order: **(1)** select `sportbooking-nat-a`; **(2)** open **Actions**; and **(3)** select **Delete NAT gateway**.

   ![Select the first NAT Gateway to delete](/images/5-Workshop/5.8-Cleanup/cleanup-nat-gateway-01.png)

   <p class="image-caption"><em>Figure 37: Select the first NAT Gateway to delete</em></p>
   The private EC2 instances have terminated, so Internet access through the NAT Gateway is no longer required.


2. **Confirm deletion of the first NAT Gateway**

   On the console, perform the following in order: **(1)** enter `delete`; and **(2)** select **Delete**.

   ![Confirm deletion of the first NAT Gateway](/images/5-Workshop/5.8-Cleanup/cleanup-nat-gateway-02.png)

   <p class="image-caption"><em>Figure 38: Confirm deletion of the first NAT Gateway</em></p>
   The NAT Gateway changes to `Deleting` while AWS processes the request.


3. **Select the NAT Gateway in the second Availability Zone**

   Check the deletion notification for the first NAT Gateway. Then **(1)** select `sportbooking-nat-b`; **(2)** open **Actions**; and **(3)** select **Delete NAT gateway**.

   ![Select the second NAT Gateway to delete](/images/5-Workshop/5.8-Cleanup/cleanup-nat-gateway-03.png)

   <p class="image-caption"><em>Figure 39: Select the second NAT Gateway to delete</em></p>
   The architecture uses one NAT Gateway in each of two Availability Zones, so both resources must be removed.


4. **Confirm deletion of the second NAT Gateway**

   On the console, perform the following in order: **(1)** enter `delete`; and **(2)** select **Delete**.

   ![Confirm deletion of the second NAT Gateway](/images/5-Workshop/5.8-Cleanup/cleanup-nat-gateway-04.png)

   <p class="image-caption"><em>Figure 40: Confirm deletion of the second NAT Gateway</em></p>
   Deleting both NAT Gateways removes the network resources that otherwise continue to incur hourly charges.


5. **Verify the status of both NAT Gateways**

   Refresh the list until both `sportbooking-nat-a` and `sportbooking-nat-b` display **Deleted**.

   ![Verify that both NAT Gateways have been deleted](/images/5-Workshop/5.8-Cleanup/cleanup-nat-gateway-05.png)

   <p class="image-caption"><em>Figure 41: Verify that both NAT Gateways have been deleted</em></p>
   Release the Elastic IPs only after NAT Gateway deletion is complete. If attempted too early, the addresses remain associated and cannot be released.


{{% notice info %}}
Refresh the status periodically and release the Elastic IPs only after both NAT Gateways display `Deleted`.
{{% /notice %}}

#### 14. Release the Elastic IPs

**Procedure:**

1. **Select the Elastic IPs used by the NAT Gateways**

   On the console, perform the following in order: **(1)** select `sportbooking-eip-us-east-1a` and `sportbooking-eip-us-east-1b`; **(2)** open **Actions**; and **(3)** select **Release Elastic IP addresses**.

   ![Select the Elastic IPs to release](/images/5-Workshop/5.8-Cleanup/cleanup-elastic-ip-01.png)

   <p class="image-caption"><em>Figure 42: Select the Elastic IPs to release</em></p>
   After the NAT Gateways are deleted, the two Elastic IPs are no longer associated and should be returned to AWS.


2. **Confirm release of the Elastic IPs**

   Verify both IP addresses and Allocation IDs, then select **Release**.

   ![Confirm release of the Elastic IPs](/images/5-Workshop/5.8-Cleanup/cleanup-elastic-ip-02.png)

   <p class="image-caption"><em>Figure 43: Confirm release of the Elastic IPs</em></p>
   A released Elastic IP no longer belongs to the AWS account and can be allocated to another account.


3. **Verify the Elastic IP list**

   Confirm the **Elastic IP addresses released** message, then verify that no project Elastic IP remains in the Region.

   ![Verify that the Elastic IPs have been released](/images/5-Workshop/5.8-Cleanup/cleanup-elastic-ip-03.png)

   <p class="image-caption"><em>Figure 44: Verify that the Elastic IPs have been released</em></p>
   This result confirms that the NAT layer no longer has dedicated public IPv4 addresses allocated.


#### 15. Delete the S3 Gateway VPC Endpoint

**Procedure:**

1. **Select the VPC Endpoint**

   On the console, perform the following in order: **(1)** select `sportbooking-vpce-s3`; **(2)** open **Actions**; and **(3)** select **Delete VPC endpoints**.

   ![Select the S3 Gateway VPC Endpoint to delete](/images/5-Workshop/5.8-Cleanup/cleanup-vpc-endpoint-01.png)

   <p class="image-caption"><em>Figure 45: Select the S3 Gateway VPC Endpoint to delete</em></p>
   The EC2 instances and S3 artifact bucket have been processed, so the endpoint has no remaining traffic to serve. Deleting the endpoint also removes its entries from the associated route tables.


2. **Confirm endpoint deletion**

   On the console, perform the following in order: **(1)** enter `delete`; and **(2)** select **Delete**.

   ![Confirm S3 Gateway VPC Endpoint deletion](/images/5-Workshop/5.8-Cleanup/cleanup-vpc-endpoint-02.png)

   <p class="image-caption"><em>Figure 46: Confirm S3 Gateway VPC Endpoint deletion</em></p>
   The endpoint is permanently deleted and cannot be restored from its current configuration.


3. **Verify that the endpoint has been deleted**

   Confirm the **Successfully deleted endpoints** message, then verify that `sportbooking-vpce-s3` no longer appears in the list.

   ![Verify that the S3 Gateway VPC Endpoint has been deleted](/images/5-Workshop/5.8-Cleanup/cleanup-vpc-endpoint-03.png)

   <p class="image-caption"><em>Figure 47: Verify that the S3 Gateway VPC Endpoint has been deleted</em></p>
   The VPC no longer contains a custom endpoint that could prevent the final deletion.


#### 16. Remove Referencing Rules and Delete the Security Groups

A Security Group cannot be deleted while it is attached to an ENI or referenced by another Security Group. The ALB, EC2 instances, and RDS database were deleted before this section was performed.

**Procedure:**

1. **Open the outbound rules of the ALB Security Group**

   Open the `sportbooking-alb` Security Group. Select the **Outbound rules** tab, then select **Edit outbound rules**.

   ![Open the outbound rules of the ALB Security Group](/images/5-Workshop/5.8-Cleanup/cleanup-security-group-01.png)

   <p class="image-caption"><em>Figure 48: Open the outbound rules of the ALB Security Group</em></p>
   The outbound rule on port `8080` references the EC2 App Security Group. This reference must be removed before the project Security Groups can be deleted together.


2. **Delete the referencing outbound rule**

   On the console, perform the following in order: **(1)** select **Delete** for the port `8080` rule that targets the App Security Group; and **(2)** select **Save rules**.

   ![Delete the outbound rule that references the App Security Group](/images/5-Workshop/5.8-Cleanup/cleanup-security-group-02.png)

   <p class="image-caption"><em>Figure 49: Delete the outbound rule that references the App Security Group</em></p>
   Removing the rule breaks the reference from the ALB Security Group to the App Security Group.


3. **Open the inbound rules of the ALB Security Group**

   Select the **Inbound rules** tab, then select **Edit inbound rules**.

   ![Open the inbound rules of the ALB Security Group](/images/5-Workshop/5.8-Cleanup/cleanup-security-group-03.png)

   <p class="image-caption"><em>Figure 50: Open the inbound rules of the ALB Security Group</em></p>
   The two public HTTP and HTTPS rules were removed to close all inbound access to the Security Group before deletion.


4. **Delete the remaining inbound rules**

   On the console, perform the following in order: **(1)** delete the HTTPS rule on port `443`; **(2)** delete the HTTP rule on port `80`; and **(3)** select **Save rules**.

   ![Delete the inbound rules of the ALB Security Group](/images/5-Workshop/5.8-Cleanup/cleanup-security-group-04.png)

   <p class="image-caption"><em>Figure 51: Delete the inbound rules of the ALB Security Group</em></p>
   CIDR-based rules do not create dependencies between Security Groups, but deleting them confirms that the Security Group permits no traffic while cleanup is in progress.


5. **Select the project Security Groups**

   Review and remove any remaining references among `sportbooking-app`, `sportbooking-rds`, and `sportbooking-alb` in the same manner. Then **(1)** select the three project Security Groups; and **(2)** open **Actions** and select **Delete security groups**.

   ![Select the SportBooking Security Groups](/images/5-Workshop/5.8-Cleanup/cleanup-security-group-05.png)

   <p class="image-caption"><em>Figure 52: Select the SportBooking Security Groups</em></p>
   Do not select the `default` Security Group or a Security Group from another VPC. If AWS reports a dependency, check the remaining ENIs and referencing rules before retrying.


6. **Confirm Security Group deletion**

   Verify that the list contains `sportbooking-rds`, `sportbooking-app`, and `sportbooking-alb`. Then **(1)** enter `delete`; and **(2)** select **Delete**.

   ![Confirm deletion of the Security Groups](/images/5-Workshop/5.8-Cleanup/cleanup-security-group-06.png)

   <p class="image-caption"><em>Figure 53: Confirm deletion of the Security Groups</em></p>
   Deleting the Security Groups together after removing references prevents unused network configurations from remaining.


7. **Verify the result**

   Confirm the **Successfully deleted 3 security groups** message, then verify that only default Security Groups or Security Groups from other environments remain.

   ![Verify that the Security Groups have been deleted](/images/5-Workshop/5.8-Cleanup/cleanup-security-group-07.png)

   <p class="image-caption"><em>Figure 54: Verify that the Security Groups have been deleted</em></p>
   This result confirms that the ENIs and Security Group associations of the ALB, EC2 instances, and RDS database have been released.


#### 17. Delete the DB Subnet Group

The DB Subnet Group was deleted after RDS because a DB instance using the subnet group prevents its deletion.

**Procedure:**

1. **Select the DB Subnet Group**

   On the console, perform the following in order: **(1)** open **RDS > Subnet groups** and select `sportbooking-db-subnet-group`; and **(2)** select **Delete**.

   ![Select the DB Subnet Group to delete](/images/5-Workshop/5.8-Cleanup/cleanup-db-subnet-group-01.png)

   <p class="image-caption"><em>Figure 55: Select the DB Subnet Group to delete</em></p>
   RDS deletion has completed, so no database continues to reference the subnet group.


2. **Confirm DB Subnet Group deletion**

   Verify the name `sportbooking-db-subnet-group`, then select **Delete**.

   ![Confirm DB Subnet Group deletion](/images/5-Workshop/5.8-Cleanup/cleanup-db-subnet-group-02.png)

   <p class="image-caption"><em>Figure 56: Confirm DB Subnet Group deletion</em></p>
   This action deletes only the RDS DB Subnet Group configuration; it does not directly delete the VPC subnets.


3. **Verify that the DB Subnet Group has been deleted**

   Confirm the successful deletion message, then verify that only default subnet groups or subnet groups from other environments remain.

   ![Verify that the DB Subnet Group has been deleted](/images/5-Workshop/5.8-Cleanup/cleanup-db-subnet-group-03.png)

   <p class="image-caption"><em>Figure 57: Verify that the DB Subnet Group has been deleted</em></p>
   The VPC no longer contains a custom RDS configuration that depends on the private subnets.


#### 18. Delete the VPC

The VPC was deleted last, after the compute, database, endpoint, NAT Gateway, and Security Group resources had been processed.

**Procedure:**

1. **Select the project VPC**

   On the console, perform the following in order: **(1)** open **VPC > Your VPCs** and select `sportbooking-vpc`; **(2)** open **Actions**; and **(3)** select **Delete VPC**.

   ![Select the SportBooking VPC to delete](/images/5-Workshop/5.8-Cleanup/cleanup-vpc-01.png)

   <p class="image-caption"><em>Figure 58: Select the SportBooking VPC to delete</em></p>
   The name, VPC ID, and CIDR were cross-checked to avoid selecting the default VPC or a VPC from another environment.


2. **Review dependent resources and confirm**

   Review the resources that will be deleted with the VPC, including the remaining Internet Gateway, subnets, and route tables. Then **(1)** enter `delete`; and **(2)** select **Delete**.

   ![Review dependent resources and confirm VPC deletion](/images/5-Workshop/5.8-Cleanup/cleanup-vpc-02.png)

   <p class="image-caption"><em>Figure 59: Review dependent resources and confirm VPC deletion</em></p>
   The dialog shows `sportbooking-vpc` and 12 related network resources in the deletion request. Every listed resource was verified as belonging to the project.


3. **Verify that the VPC has been deleted**

   Confirm the **successfully deleted sportbooking-vpc and 12 other resources** message, then verify that only the default VPC or other VPCs that must be retained remain.

   ![Verify that the SportBooking VPC has been deleted](/images/5-Workshop/5.8-Cleanup/cleanup-vpc-03.png)

   <p class="image-caption"><em>Figure 60: Verify that the SportBooking VPC has been deleted</em></p>
   This step completes the cleanup of the SportBooking network infrastructure.


{{% notice warning %}}
If the VPC cannot be deleted, check for remaining ENIs under **EC2 > Network Interfaces**, VPC Endpoints, NAT Gateways, Load Balancers, RDS resources, cross-referencing Security Groups, and resources managed by other services. Do not delete the default VPC to resolve a dependency error.
{{% /notice %}}

### Post-Cleanup Verification

| Service | Recorded status |
|---|---|
| EC2 / Auto Scaling | No `sportbooking-asg`, EC2 instance, Launch Template, or EBS volume remains outside the retention plan. |
| Elastic Load Balancing | `sportbooking-alb` and `sportbooking-tg` no longer exist. |
| RDS | The DB instance and DB Subnet Group no longer exist; retained snapshots and backups have been recorded. |
| S3 | The artifact bucket has been deleted; `sportbooking-uploads-3stars-us-east-1` is intentionally retained. |
| Secrets Manager | The secret is scheduled for deletion on a recorded date. |
| CloudWatch / SNS | No SportBooking alarm, subscription, or topic remains; CloudWatch Agent log groups are reviewed separately if configured. |
| VPC | No NAT Gateway, Elastic IP, VPC Endpoint, custom Security Group, or `sportbooking-vpc` remains. |
| IAM | `sportbooking-ec2-role` no longer exists; the project customer managed policy and instance profile have been processed. |
| ACM / Route 53 | The certificate has been deleted; the Alias record and validation CNAME no longer exist when unused; the hosted zone is retained only while the domain remains active. |
| CloudFront | No distribution requires deletion because CloudFront creation stopped at the account-verification error. |

{{% notice info %}}
Cleanup is complete only when the actual resource inventory matches the retention plan. Every remaining snapshot, backup, S3 object, CloudWatch log group, Route 53 hosted zone, and customer managed policy must have a documented purpose and retention period.
{{% /notice %}}

This step completes the SportBooking workflow from architecture design and service deployment through testing, operations, and AWS resource decommissioning.
