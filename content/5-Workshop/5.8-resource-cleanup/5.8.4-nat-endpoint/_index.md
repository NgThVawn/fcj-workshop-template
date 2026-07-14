---
title : "NAT and Endpoint Cleanup"
date : 2024-01-01
weight : 4
chapter : false
pre : " <b> 5.8.4 </b> "
---

#### 13. Delete the NAT Gateways

Both NAT Gateways were deleted before the Elastic IPs and VPC. Each NAT Gateway had to reach `Deleted` before its Elastic IP was no longer associated.

**Procedure:**

1. **Select the NAT Gateway in the first Availability Zone**

   On the console, perform the following in order: **(1)** select `sportbooking-nat-a`; **(2)** open **Actions**; and **(3)** select **Delete NAT gateway**.

   ![Select the first NAT Gateway to delete](/images/5-Workshop/5.8-resource-cleanup/cleanup-nat-gateway-01.png)

   <p class="image-caption"><em>Figure 1: Select the first NAT Gateway to delete</em></p>
   The private EC2 instances have terminated, so Internet access through the NAT Gateway is no longer required.


2. **Confirm deletion of the first NAT Gateway**

   On the console, perform the following in order: **(1)** enter `delete`; and **(2)** select **Delete**.

   ![Confirm deletion of the first NAT Gateway](/images/5-Workshop/5.8-resource-cleanup/cleanup-nat-gateway-02.png)

   <p class="image-caption"><em>Figure 2: Confirm deletion of the first NAT Gateway</em></p>
   The NAT Gateway changes to `Deleting` while AWS processes the request.


3. **Select the NAT Gateway in the second Availability Zone**

   Check the deletion notification for the first NAT Gateway. Then **(1)** select `sportbooking-nat-b`; **(2)** open **Actions**; and **(3)** select **Delete NAT gateway**.

   ![Select the second NAT Gateway to delete](/images/5-Workshop/5.8-resource-cleanup/cleanup-nat-gateway-03.png)

   <p class="image-caption"><em>Figure 3: Select the second NAT Gateway to delete</em></p>
   The architecture uses one NAT Gateway in each of two Availability Zones, so both resources must be removed.


4. **Confirm deletion of the second NAT Gateway**

   On the console, perform the following in order: **(1)** enter `delete`; and **(2)** select **Delete**.

   ![Confirm deletion of the second NAT Gateway](/images/5-Workshop/5.8-resource-cleanup/cleanup-nat-gateway-04.png)

   <p class="image-caption"><em>Figure 4: Confirm deletion of the second NAT Gateway</em></p>
   Deleting both NAT Gateways removes the network resources that otherwise continue to incur hourly charges.


5. **Verify the status of both NAT Gateways**

   Refresh the list until both `sportbooking-nat-a` and `sportbooking-nat-b` display **Deleted**.

   ![Verify that both NAT Gateways have been deleted](/images/5-Workshop/5.8-resource-cleanup/cleanup-nat-gateway-05.png)

   <p class="image-caption"><em>Figure 5: Verify that both NAT Gateways have been deleted</em></p>
   Release the Elastic IPs only after NAT Gateway deletion is complete. If attempted too early, the addresses remain associated and cannot be released.


{{% notice info %}}
Refresh the status periodically and release the Elastic IPs only after both NAT Gateways display `Deleted`.
{{% /notice %}}

#### 14. Release the Elastic IPs

**Procedure:**

1. **Select the Elastic IPs used by the NAT Gateways**

   On the console, perform the following in order: **(1)** select `sportbooking-eip-us-east-1a` and `sportbooking-eip-us-east-1b`; **(2)** open **Actions**; and **(3)** select **Release Elastic IP addresses**.

   ![Select the Elastic IPs to release](/images/5-Workshop/5.8-resource-cleanup/cleanup-elastic-ip-01.png)

   <p class="image-caption"><em>Figure 6: Select the Elastic IPs to release</em></p>
   After the NAT Gateways are deleted, the two Elastic IPs are no longer associated and should be returned to AWS.


2. **Confirm release of the Elastic IPs**

   Verify both IP addresses and Allocation IDs, then select **Release**.

   ![Confirm release of the Elastic IPs](/images/5-Workshop/5.8-resource-cleanup/cleanup-elastic-ip-02.png)

   <p class="image-caption"><em>Figure 7: Confirm release of the Elastic IPs</em></p>
   A released Elastic IP no longer belongs to the AWS account and can be allocated to another account.


3. **Verify the Elastic IP list**

   Confirm the **Elastic IP addresses released** message, then verify that no project Elastic IP remains in the Region.

   ![Verify that the Elastic IPs have been released](/images/5-Workshop/5.8-resource-cleanup/cleanup-elastic-ip-03.png)

   <p class="image-caption"><em>Figure 8: Verify that the Elastic IPs have been released</em></p>
   This result confirms that the NAT layer no longer has dedicated public IPv4 addresses allocated.


#### 15. Delete the S3 Gateway VPC Endpoint

**Procedure:**

1. **Select the VPC Endpoint**

   On the console, perform the following in order: **(1)** select `sportbooking-vpce-s3`; **(2)** open **Actions**; and **(3)** select **Delete VPC endpoints**.

   ![Select the S3 Gateway VPC Endpoint to delete](/images/5-Workshop/5.8-resource-cleanup/cleanup-vpc-endpoint-01.png)

   <p class="image-caption"><em>Figure 9: Select the S3 Gateway VPC Endpoint to delete</em></p>
   The EC2 instances and S3 artifact bucket have been processed, so the endpoint has no remaining traffic to serve. Deleting the endpoint also removes its entries from the associated route tables.


2. **Confirm endpoint deletion**

   On the console, perform the following in order: **(1)** enter `delete`; and **(2)** select **Delete**.

   ![Confirm S3 Gateway VPC Endpoint deletion](/images/5-Workshop/5.8-resource-cleanup/cleanup-vpc-endpoint-02.png)

   <p class="image-caption"><em>Figure 10: Confirm S3 Gateway VPC Endpoint deletion</em></p>
   The endpoint is permanently deleted and cannot be restored from its current configuration.


3. **Verify that the endpoint has been deleted**

   Confirm the **Successfully deleted endpoints** message, then verify that `sportbooking-vpce-s3` no longer appears in the list.

   ![Verify that the S3 Gateway VPC Endpoint has been deleted](/images/5-Workshop/5.8-resource-cleanup/cleanup-vpc-endpoint-03.png)

   <p class="image-caption"><em>Figure 11: Verify that the S3 Gateway VPC Endpoint has been deleted</em></p>
   The VPC no longer contains a custom endpoint that could prevent the final deletion.
