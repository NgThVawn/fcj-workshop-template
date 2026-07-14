---
title : "Data Cleanup"
date : 2024-01-01
weight : 3
chapter : false
pre : " <b> 5.8.3 </b> "
---

#### 9. Delete the RDS MySQL Database

**Procedure:**

1. **Start deleting the DB instance**

   On the console, perform the following in order: **(1)** open **RDS > Databases** and select `sportbooking-mysql`; **(2)** open **Actions**; and **(3)** select **Delete**.

   ![Select the RDS MySQL database to delete](/images/5-Workshop/5.8-resource-cleanup/cleanup-rds-01.png)

   <p class="image-caption"><em>Figure 1: Select the RDS MySQL database to delete</em></p>
   The EC2 instances have terminated, so no application connection continues to use the database.


2. **Configure data retention and confirm**

   Select **Create final snapshot** and enter `sportbooking-mysql-snapshot` as the snapshot name. Select **Retain automated backups** to preserve recovery capability. Finally, **(1)** enter `delete me`; and **(2)** select **Delete**.

   ![Configure the snapshot and confirm RDS deletion](/images/5-Workshop/5.8-resource-cleanup/cleanup-rds-02.png)

   <p class="image-caption"><em>Figure 2: Configure the snapshot and confirm RDS deletion</em></p>
   The `sportbooking-mysql-snapshot` final snapshot and automated backup were retained according to the post-deletion storage plan.


3. **Verify that the DB instance has been deleted**

   Wait for the deletion process to finish, then confirm the **Successfully deleted DB instance sportbooking-mysql** message.

   ![Verify that the RDS MySQL database has been deleted](/images/5-Workshop/5.8-resource-cleanup/cleanup-rds-03.png)

   <p class="image-caption"><em>Figure 3: Verify that the RDS MySQL database has been deleted</em></p>
   The DB instance no longer incurs compute charges, but the selected snapshot and retained backup remain storage resources that require management.


{{% notice warning %}}
The final snapshot and retained automated backup can continue to incur storage charges after the DB instance is deleted. Monitor both resources according to the agreed retention period.
{{% /notice %}}

#### 10. Empty and Delete the S3 Bucket

The artifact bucket was emptied before deletion. The upload bucket was retained during this cleanup to preserve images uploaded by users.

**Procedure:**

1. **Select the bucket and start emptying it**

   On the console, perform the following in order: **(1)** select the `sportbooking-artifacts-3stars-us-east-1` bucket; and **(2)** select **Empty**.

   ![Select the S3 artifact bucket to empty](/images/5-Workshop/5.8-resource-cleanup/cleanup-s3-01.png)

   <p class="image-caption"><em>Figure 4: Select the S3 artifact bucket to empty</em></p>
   S3 does not allow a bucket to be deleted while it contains objects, object versions, or delete markers.


2. **Confirm that the bucket will be emptied**

   On the console, perform the following in order: **(1)** enter `permanently delete`; and **(2)** select **Empty**.

   ![Confirm that the S3 bucket will be emptied](/images/5-Workshop/5.8-resource-cleanup/cleanup-s3-02.png)

   <p class="image-caption"><em>Figure 5: Confirm that the S3 bucket will be emptied</em></p>
   Emptying the bucket permanently removes its data unless another backup is available.


3. **Verify that the bucket is empty**

   Confirm the **Successfully emptied bucket** status. Review the counts of successfully deleted and failed objects.

   ![Verify that the S3 bucket has been emptied](/images/5-Workshop/5.8-resource-cleanup/cleanup-s3-03.png)

   <p class="image-caption"><em>Figure 6: Verify that the S3 bucket has been emptied</em></p>
   In the recorded result, two objects totaling 82.5 MB were deleted and no object failed.


4. **Select the bucket for deletion**

   On the console, perform the following in order: **(1)** return to the bucket list and select the artifact bucket that was just emptied; and **(2)** select **Delete**.

   ![Select the S3 artifact bucket for deletion](/images/5-Workshop/5.8-resource-cleanup/cleanup-s3-04.png)

   <p class="image-caption"><em>Figure 7: Select the S3 artifact bucket for deletion</em></p>
   Delete only the project bucket. A `cf-templates-...` bucket may be used by CloudFormation or another environment and must not be deleted without verification.


5. **Confirm the bucket name**

   On the console, perform the following in order: **(1)** enter the exact bucket name; and **(2)** select **Delete bucket**.

   ![Confirm S3 artifact bucket deletion](/images/5-Workshop/5.8-resource-cleanup/cleanup-s3-05.png)

   <p class="image-caption"><em>Figure 8: Confirm S3 artifact bucket deletion</em></p>
   S3 bucket names are globally unique. After deletion, the same name is not guaranteed to remain available for reuse.


6. **Verify that the bucket has been deleted**

   Confirm the **Successfully deleted bucket** message, then verify that the artifact bucket no longer appears in the list.

   ![Verify that the S3 artifact bucket has been deleted](/images/5-Workshop/5.8-resource-cleanup/cleanup-s3-06.png)

   <p class="image-caption"><em>Figure 9: Verify that the S3 artifact bucket has been deleted</em></p>
   The `sportbooking-uploads-3stars-us-east-1` bucket remains in the list after the artifact bucket is deleted. This bucket is intentionally retained and is not an overlooked resource.


#### 11. Schedule Secret Deletion

**Procedure:**

1. **Select the secret for deletion**

   Open the `sportbooking/prod/app-env` secret. Then **(1)** open **Actions**; and **(2)** select **Delete secret**.

   ![Select the secret to delete](/images/5-Workshop/5.8-resource-cleanup/cleanup-secret-01.png)

   <p class="image-caption"><em>Figure 10: Select the secret to delete</em></p>
   The EC2 instances and IAM role have been deleted, so no workload within the project scope continues to read the secret.


2. **Set the recovery window and schedule deletion**

   On the console, perform the following in order: **(1)** set **Waiting period** to `7` days or the period required by the retention policy; and **(2)** select **Schedule deletion**.

   ![Schedule deletion of the Secrets Manager secret](/images/5-Workshop/5.8-resource-cleanup/cleanup-secret-02.png)

   <p class="image-caption"><em>Figure 11: Schedule deletion of the Secrets Manager secret</em></p>
   Secrets Manager does not delete the secret immediately. It places the secret in a recovery window during which the secret is disabled and can be restored if the deletion was accidental.


{{% notice tip %}}
Do not include secret values in screenshots or report content. If redeployment is required during the recovery window, restore the secret before updating it instead of creating a duplicate name.
{{% /notice %}}

#### 12. Delete the ACM Certificate

Delete the ACM certificate only when the **In use** column displays **No**. If an ALB or CloudFront distribution still uses the certificate, remove that association first.

**Procedure:**

1. **Select the certificate**

   On the console, perform the following in order: **(1)** select the certificate for `sport.younglilliu.id.vn` and verify **In use = No**; and **(2)** open **More actions** and select **Delete**.

   ![Select the ACM certificate to delete](/images/5-Workshop/5.8-resource-cleanup/cleanup-acm-01.png)

   <p class="image-caption"><em>Figure 12: Select the ACM certificate to delete</em></p>
   The ALB has been deleted, so its HTTPS listener no longer holds the certificate. No CloudFront distribution was created, so the certificate has no additional CloudFront dependency.


2. **Confirm certificate deletion**

   On the console, perform the following in order: **(1)** enter `delete`; and **(2)** select **Delete**.

   ![Confirm ACM certificate deletion](/images/5-Workshop/5.8-resource-cleanup/cleanup-acm-02.png)

   <p class="image-caption"><em>Figure 13: Confirm ACM certificate deletion</em></p>
   After deletion, the certificate can no longer secure HTTPS connections and cannot be restored.


3. **Verify that the certificate has been deleted**

   Confirm the **Successfully deleted certificate** message, then verify that the project certificate no longer appears in the list.

   ![Verify that the ACM certificate has been deleted](/images/5-Workshop/5.8-resource-cleanup/cleanup-acm-03.png)

   <p class="image-caption"><em>Figure 14: Verify that the ACM certificate has been deleted</em></p>
   The certificate-validation CNAME record can now be removed from Route 53 if no other certificate uses it.
