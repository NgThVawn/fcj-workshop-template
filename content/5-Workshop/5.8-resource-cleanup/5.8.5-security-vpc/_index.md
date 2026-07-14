---
title : "Security Groups and VPC"
date : 2024-01-01
weight : 5
chapter : false
pre : " <b> 5.8.5 </b> "
---

#### 16. Remove Referencing Rules and Delete the Security Groups

A Security Group cannot be deleted while it is attached to an ENI or referenced by another Security Group. The ALB, EC2 instances, and RDS database were deleted before this section was performed.

**Procedure:**

1. **Open the outbound rules of the ALB Security Group**

   Open the `sportbooking-alb` Security Group. Select the **Outbound rules** tab, then select **Edit outbound rules**.

   ![Open the outbound rules of the ALB Security Group](/images/5-Workshop/5.8-resource-cleanup/cleanup-security-group-01.png)

   <p class="image-caption"><em>Figure 1: Open the outbound rules of the ALB Security Group</em></p>
   The outbound rule on port `8080` references the EC2 App Security Group. This reference must be removed before the project Security Groups can be deleted together.


2. **Delete the referencing outbound rule**

   On the console, perform the following in order: **(1)** select **Delete** for the port `8080` rule that targets the App Security Group; and **(2)** select **Save rules**.

   ![Delete the outbound rule that references the App Security Group](/images/5-Workshop/5.8-resource-cleanup/cleanup-security-group-02.png)

   <p class="image-caption"><em>Figure 2: Delete the outbound rule that references the App Security Group</em></p>
   Removing the rule breaks the reference from the ALB Security Group to the App Security Group.


3. **Open the inbound rules of the ALB Security Group**

   Select the **Inbound rules** tab, then select **Edit inbound rules**.

   ![Open the inbound rules of the ALB Security Group](/images/5-Workshop/5.8-resource-cleanup/cleanup-security-group-03.png)

   <p class="image-caption"><em>Figure 3: Open the inbound rules of the ALB Security Group</em></p>
   The two public HTTP and HTTPS rules were removed to close all inbound access to the Security Group before deletion.


4. **Delete the remaining inbound rules**

   On the console, perform the following in order: **(1)** delete the HTTPS rule on port `443`; **(2)** delete the HTTP rule on port `80`; and **(3)** select **Save rules**.

   ![Delete the inbound rules of the ALB Security Group](/images/5-Workshop/5.8-resource-cleanup/cleanup-security-group-04.png)

   <p class="image-caption"><em>Figure 4: Delete the inbound rules of the ALB Security Group</em></p>
   CIDR-based rules do not create dependencies between Security Groups, but deleting them confirms that the Security Group permits no traffic while cleanup is in progress.


5. **Select the project Security Groups**

   Review and remove any remaining references among `sportbooking-app`, `sportbooking-rds`, and `sportbooking-alb` in the same manner. Then **(1)** select the three project Security Groups; and **(2)** open **Actions** and select **Delete security groups**.

   ![Select the SportBooking Security Groups](/images/5-Workshop/5.8-resource-cleanup/cleanup-security-group-05.png)

   <p class="image-caption"><em>Figure 5: Select the SportBooking Security Groups</em></p>
   Do not select the `default` Security Group or a Security Group from another VPC. If AWS reports a dependency, check the remaining ENIs and referencing rules before retrying.


6. **Confirm Security Group deletion**

   Verify that the list contains `sportbooking-rds`, `sportbooking-app`, and `sportbooking-alb`. Then **(1)** enter `delete`; and **(2)** select **Delete**.

   ![Confirm deletion of the Security Groups](/images/5-Workshop/5.8-resource-cleanup/cleanup-security-group-06.png)

   <p class="image-caption"><em>Figure 6: Confirm deletion of the Security Groups</em></p>
   Deleting the Security Groups together after removing references prevents unused network configurations from remaining.


7. **Verify the result**

   Confirm the **Successfully deleted 3 security groups** message, then verify that only default Security Groups or Security Groups from other environments remain.

   ![Verify that the Security Groups have been deleted](/images/5-Workshop/5.8-resource-cleanup/cleanup-security-group-07.png)

   <p class="image-caption"><em>Figure 7: Verify that the Security Groups have been deleted</em></p>
   This result confirms that the ENIs and Security Group associations of the ALB, EC2 instances, and RDS database have been released.


#### 17. Delete the DB Subnet Group

The DB Subnet Group was deleted after RDS because a DB instance using the subnet group prevents its deletion.

**Procedure:**

1. **Select the DB Subnet Group**

   On the console, perform the following in order: **(1)** open **RDS > Subnet groups** and select `sportbooking-db-subnet-group`; and **(2)** select **Delete**.

   ![Select the DB Subnet Group to delete](/images/5-Workshop/5.8-resource-cleanup/cleanup-db-subnet-group-01.png)

   <p class="image-caption"><em>Figure 8: Select the DB Subnet Group to delete</em></p>
   RDS deletion has completed, so no database continues to reference the subnet group.


2. **Confirm DB Subnet Group deletion**

   Verify the name `sportbooking-db-subnet-group`, then select **Delete**.

   ![Confirm DB Subnet Group deletion](/images/5-Workshop/5.8-resource-cleanup/cleanup-db-subnet-group-02.png)

   <p class="image-caption"><em>Figure 9: Confirm DB Subnet Group deletion</em></p>
   This action deletes only the RDS DB Subnet Group configuration; it does not directly delete the VPC subnets.


3. **Verify that the DB Subnet Group has been deleted**

   Confirm the successful deletion message, then verify that only default subnet groups or subnet groups from other environments remain.

   ![Verify that the DB Subnet Group has been deleted](/images/5-Workshop/5.8-resource-cleanup/cleanup-db-subnet-group-03.png)

   <p class="image-caption"><em>Figure 10: Verify that the DB Subnet Group has been deleted</em></p>
   The VPC no longer contains a custom RDS configuration that depends on the private subnets.


#### 18. Delete the VPC

The VPC was deleted last, after the compute, database, endpoint, NAT Gateway, and Security Group resources had been processed.

**Procedure:**

1. **Select the project VPC**

   On the console, perform the following in order: **(1)** open **VPC > Your VPCs** and select `sportbooking-vpc`; **(2)** open **Actions**; and **(3)** select **Delete VPC**.

   ![Select the SportBooking VPC to delete](/images/5-Workshop/5.8-resource-cleanup/cleanup-vpc-01.png)

   <p class="image-caption"><em>Figure 11: Select the SportBooking VPC to delete</em></p>
   The name, VPC ID, and CIDR were cross-checked to avoid selecting the default VPC or a VPC from another environment.


2. **Review dependent resources and confirm**

   Review the resources that will be deleted with the VPC, including the remaining Internet Gateway, subnets, and route tables. Then **(1)** enter `delete`; and **(2)** select **Delete**.

   ![Review dependent resources and confirm VPC deletion](/images/5-Workshop/5.8-resource-cleanup/cleanup-vpc-02.png)

   <p class="image-caption"><em>Figure 12: Review dependent resources and confirm VPC deletion</em></p>
   The dialog shows `sportbooking-vpc` and 12 related network resources in the deletion request. Every listed resource was verified as belonging to the project.


3. **Verify that the VPC has been deleted**

   Confirm the **successfully deleted sportbooking-vpc and 12 other resources** message, then verify that only the default VPC or other VPCs that must be retained remain.

   ![Verify that the SportBooking VPC has been deleted](/images/5-Workshop/5.8-resource-cleanup/cleanup-vpc-03.png)

   <p class="image-caption"><em>Figure 13: Verify that the SportBooking VPC has been deleted</em></p>
   This step completes the cleanup of the SportBooking network infrastructure.


{{% notice warning %}}
If the VPC cannot be deleted, check for remaining ENIs under **EC2 > Network Interfaces**, VPC Endpoints, NAT Gateways, Load Balancers, RDS resources, cross-referencing Security Groups, and resources managed by other services. Do not delete the default VPC to resolve a dependency error.
{{% /notice %}}
