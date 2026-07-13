---
title : "Create the VPC, Subnets, and Route Tables"
date : 2024-01-01
weight : 1
chapter : false
pre : " <b> 5.3.1 </b> "
---

#### VPC and Subnet Configuration

Create a dedicated VPC for SportBooking and divide its subnets into three tiers:

- Public subnets: host the Application Load Balancer and NAT Gateways.
- Private application subnets: host EC2 application instances managed by the Auto Scaling Group.
- Private database subnets: host Amazon RDS for MySQL through a DB subnet group.

| Component | Configuration |
|---|---|
| VPC | `sportbooking-vpc`, CIDR `10.20.0.0/16`. |
| Public subnets | Two subnets in two AZs, with a route to the Internet Gateway. |
| Private application subnets | Two subnets in two AZs, with outbound access through NAT Gateways. |
| Private database subnets | Two subnets in two AZs, using only local VPC routing. |

#### Internet Gateway, NAT Gateways, and Route Tables

| Route/NAT | Configuration | Role |
|---|---|---|
| Public route table | `0.0.0.0/0 -> Internet Gateway` | Allows the public ALB to receive Internet requests. |
| Private application route table | `0.0.0.0/0 -> NAT Gateway` | Allows private EC2 instances to install packages, retrieve S3 objects, and call Secrets Manager when a dedicated endpoint is not available. |
| Private database route table | Local VPC route only | RDS does not require inbound or outbound Internet access in this baseline deployment. |
| NAT Gateways | Two NAT Gateways in two public subnets, each with an Elastic IP | Each private application subnet uses the NAT Gateway in the same AZ for an independent outbound path. |

#### Verify the Network Configuration

1. Open the VPC console and review the Resource map.
2. Confirm that each public subnet is associated with a route table whose default route targets the Internet Gateway.
3. Confirm that each private application subnet is associated with a route table whose default route targets a NAT Gateway.
4. Confirm that private database subnets are not public and use only the required internal routes.
5. Verify that both NAT Gateways are in the `Available` state.

{{% notice info %}}
After the workflow completes, compare the Resource map with the design: CIDR `10.20.0.0/16`, two Availability Zones, two public subnets, four private subnets, the default routes, and two NAT Gateways in the `Available` state.
{{% /notice %}}

#### Procedure

The procedure starts with the VPC, then creates the subnets, Internet Gateway, NAT Gateways, and route tables as one coordinated workflow for the SportBooking environment.

**Procedure:**

1. **Open the Create VPC page**

   From the AWS Management Console, open **VPC > Your VPCs**, verify that the Region is `us-east-1`, and choose **Create VPC** to begin creating the project's private network.

   ![Open the Create VPC page](/images/5-Workshop/5.3-S3-vpc/vpc-01.png)

   <p class="image-caption"><em>Figure 1: Open the Create VPC page</em></p>
   The VPC is the project's isolated network layer. A dedicated VPC separates the SportBooking infrastructure from the default VPC and makes subnets, route tables, and NAT Gateways easier to control.


2. **Select the VPC and more mode**

   Select **VPC and more** so AWS creates the VPC, subnets, route tables, and gateways in one workflow. Enter `sportbooking-vpc` as the VPC name, then set the private IPv4 CIDR to `10.20.0.0/16`.

   ![Select the VPC and more mode](/images/5-Workshop/5.3-S3-vpc/vpc-02.png)

   <p class="image-caption"><em>Figure 2: Select the VPC and more mode</em></p>
   This option creates the complete network foundation in a single workflow and reduces errors that can occur when each subnet and route table is created separately.


3. **Configure Availability Zones, public/private subnets, and NAT**

   Select **2 Availability Zones**. Create public subnets for the ALB and NAT Gateways, and private subnets for EC2 and RDS. Select **2 NAT gateways**, one in each Availability Zone.

   ![Configure Availability Zones, public and private subnets, and NAT](/images/5-Workshop/5.3-S3-vpc/vpc-03.png)

   <p class="image-caption"><em>Figure 3: Configure Availability Zones, public/private subnets, and NAT</em></p>
   Two AZs provide basic fault tolerance for the ALB and Auto Scaling Group. NAT Gateways give private EC2 instances outbound Internet access without allowing inbound Internet traffic.


4. **Review the configuration before creation**

   Review the VPC name, CIDR, number of AZs, subnet counts, and NAT Gateway count. Scroll to the bottom of the page and choose **Create VPC**.

   ![Review the configuration before creating the VPC](/images/5-Workshop/5.3-S3-vpc/vpc-04.png)

   <p class="image-caption"><em>Figure 4: Review the configuration before creating the VPC</em></p>
   Validate the network configuration before creating the ALB, ASG, and RDS to avoid selecting incorrect Availability Zones or subnets later.


5. **Monitor the Create VPC workflow**

   Monitor the list of resources being created and wait until each item reports successful completion.

   ![Monitor the Create VPC workflow](/images/5-Workshop/5.3-S3-vpc/vpc-05.png)

   <p class="image-caption"><em>Figure 5: Monitor the Create VPC workflow</em></p>
   The workflow identifies which resources completed successfully and which failed, particularly route tables, NAT Gateways, and subnet associations.


6. **Complete the workflow**

   Review the list of created resources, then choose **View VPC** or return to the VPC list.

   ![Complete the Create VPC workflow](/images/5-Workshop/5.3-S3-vpc/vpc-06.png)

   <p class="image-caption"><em>Figure 6: Complete the workflow</em></p>
   After the workflow succeeds, open the VPC to inspect the Resource map. This VPC is subsequently used by RDS, the ALB, Launch Template, and Auto Scaling Group.


7. **Inspect the VPC Resource map**

   Open the new VPC and select **Resource map**. Confirm that the public subnets, private subnets, route tables, Internet Gateway, and NAT Gateways match the design.

   ![Inspect the VPC Resource map](/images/5-Workshop/5.3-S3-vpc/vpc-07.png)

   <p class="image-caption"><em>Figure 7: Inspect the VPC Resource map</em></p>
   The Resource map provides clear visual evidence of the network architecture: the ALB is placed in public subnets, while EC2 and RDS are placed in private subnets.

