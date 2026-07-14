---
title : "Application Cleanup"
date : 2024-01-01
weight : 2
chapter : false
pre : " <b> 5.8.2 </b> "
---

#### 4. Scale In and Delete the Auto Scaling Group

The Auto Scaling Group had to be processed before the Launch Template. Reducing capacity to `0` allowed the EC2 instances to terminate in a controlled manner before the scaling configuration was deleted.

**Procedure:**

1. **Open the capacity settings**

   On the console, perform the following in order: **(1)** select `sportbooking-asg`; and **(2)** select **Edit** in **Capacity overview**.

   ![Open the Auto Scaling Group capacity settings](/images/5-Workshop/5.8-resource-cleanup/cleanup-asg-01.png)

   <p class="image-caption"><em>Figure 1: Open the Auto Scaling Group capacity settings</em></p>
   The ASG is maintaining two healthy EC2 instances. Without reducing capacity, it continues to launch a replacement whenever an instance is terminated.


2. **Set capacity to 0**

   On the console, perform the following in order: **(1)** set **Desired capacity** to `0`; **(2)** set **Min desired capacity** to `0`; and **(3)** select **Update**.

   ![Set the Auto Scaling Group capacity to 0](/images/5-Workshop/5.8-resource-cleanup/cleanup-asg-02.png)

   <p class="image-caption"><em>Figure 2: Set the Auto Scaling Group capacity to 0</em></p>
   Setting both desired and minimum capacity to `0` allows the ASG to terminate every EC2 instance without launching a replacement. Maximum capacity can remain unchanged because the ASG is deleted in a later step.


3. **Verify that the ASG has no instances**

   Refresh the page until the **Instances** column shows `0`. Verify that **Desired capacity** and the minimum limit are also `0`.

   ![Verify that the Auto Scaling Group has scaled down to 0](/images/5-Workshop/5.8-resource-cleanup/cleanup-asg-03.png)

   <p class="image-caption"><em>Figure 3: Verify that the Auto Scaling Group has scaled down to 0</em></p>
   Delete the ASG only after its EC2 instances have terminated to reduce the risk of leaving an ENI or EBS volume in use.


4. **Start deleting the ASG**

   On the console, perform the following in order: **(1)** open **Actions**; and **(2)** select **Delete**.

   ![Start deleting the Auto Scaling Group](/images/5-Workshop/5.8-resource-cleanup/cleanup-asg-04.png)

   <p class="image-caption"><em>Figure 4: Start deleting the Auto Scaling Group</em></p>
   With no remaining instances, the ASG can be deleted without affecting data on a running EC2 instance.


5. **Confirm ASG deletion**

   On the console, perform the following in order: **(1)** enter `delete` in the confirmation field; and **(2)** select **Delete**.

   ![Confirm Auto Scaling Group deletion](/images/5-Workshop/5.8-resource-cleanup/cleanup-asg-05.png)

   <p class="image-caption"><em>Figure 5: Confirm Auto Scaling Group deletion</em></p>
   After the ASG is deleted, the Launch Template is no longer referenced by the automatic scaling configuration.


{{% notice tip %}}
After deleting the ASG, also check **EC2 > Instances** and **Elastic Block Store > Volumes**. Retain only volumes or snapshots explicitly identified as required backups.
{{% /notice %}}

#### 5. Delete the Application Load Balancer

The ALB had to be deleted before the Target Group because its listeners referenced the Target Group.

**Procedure:**

1. **Select and delete the ALB**

   On the console, perform the following in order: **(1)** open **EC2 > Load Balancers** and select `sportbooking-alb`; **(2)** open **Actions**; and **(3)** select **Delete load balancer**, then confirm in the displayed dialog.

   ![Select the Application Load Balancer to delete](/images/5-Workshop/5.8-resource-cleanup/cleanup-alb-01.png)

   <p class="image-caption"><em>Figure 6: Select the Application Load Balancer to delete</em></p>
   Deleting the ALB also removes its HTTP and HTTPS listeners and releases the association with the ACM certificate. Wait until the ALB disappears from the list before deleting the Target Group.


#### 6. Delete the Target Group

**Procedure:**

1. **Select the Target Group**

   On the console, perform the following in order: **(1)** open **EC2 > Target Groups** and select `sportbooking-tg`; **(2)** open **Actions**; and **(3)** select **Delete**.

   ![Select the Target Group to delete](/images/5-Workshop/5.8-resource-cleanup/cleanup-target-group-01.png)

   <p class="image-caption"><em>Figure 7: Select the Target Group to delete</em></p>
   The Target Group can be deleted only after the ALB and listeners no longer reference it. If AWS reports that the resource is still in use, check for a remaining ALB or listener rule.


2. **Confirm Target Group deletion**

   Verify the name `sportbooking-tg`, then select **Delete**.

   ![Confirm Target Group deletion](/images/5-Workshop/5.8-resource-cleanup/cleanup-target-group-02.png)

   <p class="image-caption"><em>Figure 8: Confirm Target Group deletion</em></p>
   The target EC2 instances terminated when ASG capacity was reduced to `0`, so the Target Group has no backend to maintain.


#### 7. Delete the Launch Template

**Procedure:**

1. **Select the Launch Template**

   On the console, perform the following in order: **(1)** open **EC2 > Launch Templates** and select `sportbooking-lt`; **(2)** open **Actions**; and **(3)** select **Delete template**.

   ![Select the Launch Template to delete](/images/5-Workshop/5.8-resource-cleanup/cleanup-launch-template-01.png)

   <p class="image-caption"><em>Figure 9: Select the Launch Template to delete</em></p>
   The Launch Template is deleted after the ASG so that no configuration continues to reference the template or any of its versions.


2. **Confirm Launch Template deletion**

   On the console, perform the following in order: **(1)** enter `Delete` with the exact capitalization requested on the screen; and **(2)** select **Delete**.

   ![Confirm Launch Template deletion](/images/5-Workshop/5.8-resource-cleanup/cleanup-launch-template-02.png)

   <p class="image-caption"><em>Figure 10: Confirm Launch Template deletion</em></p>
   Deleting the template removes every version of `sportbooking-lt` and cannot be undone.


#### 8. Detach Policies and Delete the IAM Role

The IAM role was deleted only after all EC2 instances had terminated and no instance profile remained in use.

**Procedure:**

1. **Select the policies attached to the role**

   On the console, perform the following in order: **(1)** open the `sportbooking-ec2-role` role and select every attached policy; and **(2)** select **Remove**.

   ![Select the policies to detach from the IAM role](/images/5-Workshop/5.8-resource-cleanup/cleanup-iam-role-01.png)

   <p class="image-caption"><em>Figure 11: Select the policies to detach from the IAM role</em></p>
   The role shown in the figure has AWS managed policies and the `sportbooking-ec2-policy` customer managed policy attached. These associations must be removed before the role is deleted.


2. **Confirm policy removal**

   Verify the number of selected policies, then select **Remove**.

   ![Confirm policy removal from the IAM role](/images/5-Workshop/5.8-resource-cleanup/cleanup-iam-role-02.png)

   <p class="image-caption"><em>Figure 12: Confirm policy removal from the IAM role</em></p>
   AWS managed policies are detached from the role but are not deleted from the AWS account.


3. **Select the role to delete**

   On the console, perform the following in order: **(1)** return to **IAM > Roles** and select `sportbooking-ec2-role`; and **(2)** select **Delete**.

   ![Select the IAM role to delete](/images/5-Workshop/5.8-resource-cleanup/cleanup-iam-role-03.png)

   <p class="image-caption"><em>Figure 13: Select the IAM role to delete</em></p>
   Select only the project role. Do not delete a service-linked role or a role shared with another resource.


4. **Confirm role deletion**

   Enter `sportbooking-ec2-role` exactly, then select **Delete**.

   ![Confirm IAM role deletion](/images/5-Workshop/5.8-resource-cleanup/cleanup-iam-role-04.png)

   <p class="image-caption"><em>Figure 14: Confirm IAM role deletion</em></p>
   Deleting the role removes the permissions that EC2 used to access S3, Secrets Manager, Systems Manager, and CloudWatch.


{{% notice info %}}
The `sportbooking-ec2-policy` customer managed policy remains after it is detached. If the policy is used only by this project, open **IAM > Policies**, verify that it has no attached entity, and delete the policy together with its non-default versions.
{{% /notice %}}
