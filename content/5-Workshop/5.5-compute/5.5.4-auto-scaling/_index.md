---
title : "Auto Scaling Group"
date : 2024-01-01
weight : 4
chapter : false
pre : " <b> 5.5.4 </b> "
---

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

   ![Start creating the Auto Scaling Group](/images/5-Workshop/5.5-compute/auto-scaling-group-01.png)

   <p class="image-caption"><em>Figure 1: Start creating the Auto Scaling Group</em></p>
   The ASG maintains EC2 application capacity, replaces unhealthy instances, and supports rollouts through Instance Refresh.


2. **Select the Launch Template**

   Enter `sportbooking-asg` as the ASG name, select `sportbooking-lt` as the Launch Template, and select version `Default (1)`.

   ![Select the Launch Template](/images/5-Workshop/5.5-compute/auto-scaling-group-02.png)

   <p class="image-caption"><em>Figure 2: Select the Launch Template</em></p>
   The ASG creates instances from the Launch Template. An incorrect version can cause instances to use outdated User Data or an outdated AMI.


3. **Confirm the template version**

   Review the AMI, instance type, key pair, and Security Group summary, then choose **Next**.

   ![Confirm the Launch Template version](/images/5-Workshop/5.5-compute/auto-scaling-group-03.png)

   <p class="image-caption"><em>Figure 3: Confirm the template version</em></p>
   This is the final template check before the ASG creates instances in private subnets.


4. **Select the VPC and private application subnets**

   Select the SportBooking VPC and private application subnets in multiple AZs.

   ![Select the VPC and private application subnets](/images/5-Workshop/5.5-compute/auto-scaling-group-04.png)

   <p class="image-caption"><em>Figure 4: Select the VPC and private application subnets</em></p>
   EC2 application instances do not need public IP addresses. Private subnet placement ensures that only the ALB can call the application.


5. **Configure advanced capacity options**

   Select **Balanced best effort** as the distribution strategy, retain **Default** for Capacity Reservation preference, and choose **Next**.

   ![Configure advanced capacity options](/images/5-Workshop/5.5-compute/auto-scaling-group-05.png)

   <p class="image-caption"><em>Figure 5: Configure advanced capacity options</em></p>
   This deployment prioritizes a simple and stable configuration. A production environment can use mixed instances or Spot Instances to optimize cost.


6. **Attach load balancing**

   Enable load balancer integration, then attach the ASG to the existing `sportbooking-tg` Target Group.

   ![Attach load balancing](/images/5-Workshop/5.5-compute/auto-scaling-group-06.png)

   <p class="image-caption"><em>Figure 6: Attach load balancing</em></p>
   The ASG must register EC2 instances with the Target Group so the ALB can forward traffic and perform health checks.


7. **Configure health checks**

   Enable Elastic Load Balancing health checks, set the health check grace period to `300` seconds, and choose **Next**.

   ![Configure health checks](/images/5-Workshop/5.5-compute/auto-scaling-group-07.png)

   <p class="image-caption"><em>Figure 7: Configure health checks</em></p>
   A grace period that is too short can cause the ASG to replace an instance before the application finishes starting.


8. **Configure group size and scaling limits**

   Set desired capacity to `2`, minimum capacity to `2`, and maximum capacity to `4`.

   ![Configure group size and scaling limits](/images/5-Workshop/5.5-compute/auto-scaling-group-08.png)

   <p class="image-caption"><em>Figure 8: Configure group size and scaling</em></p>
   Maintain two baseline instances while allowing the ASG to scale out to four instances.


9. **Configure the target tracking policy**

   Select **Target tracking scaling policy**, select **Average CPU utilization**, set the target value to `60`, and set instance warmup to `300` seconds.

   ![Configure the target tracking policy](/images/5-Workshop/5.5-compute/auto-scaling-group-09.png)

   <p class="image-caption"><em>Figure 9: Configure target tracking for 60% average CPU</em></p>
   This policy adjusts desired capacity between two and four instances based on average CPU utilization across the Auto Scaling Group.


10. **Complete the Auto Scaling Group options**

   Keep deletion protection at **None (default)** and do not use a placement group. Review monitoring, warmup, and instance-maintenance options, then choose **Next**.

   ![Complete the Auto Scaling Group options](/images/5-Workshop/5.5-compute/auto-scaling-group-10.png)

   <p class="image-caption"><em>Figure 10: Review maintenance options and continue</em></p>
   Complete these options before configuring notifications and tags.


11. **Skip notifications during ASG creation**

   No notification is added during ASG creation. Choose **Next**.

   ![Skip or add notifications](/images/5-Workshop/5.5-compute/auto-scaling-group-11.png)

   <p class="image-caption"><em>Figure 11: Skip or add notifications</em></p>
   The SNS channel is created separately in section 5.7 and used as an action for the CloudWatch alarm.


12. **Add an ASG tag**

   Add a tag such as `Project=SportBooking`, then choose **Next**.

   ![Add an ASG tag](/images/5-Workshop/5.5-compute/auto-scaling-group-12.png)

   <p class="image-caption"><em>Figure 12: Add a tag to the ASG</em></p>
   Tags support resource filtering, cost management, and identification of project instances.


13. **Review and create the ASG**

   Review the Launch Template, subnets, Target Group, capacity, and tags, then choose **Create Auto Scaling group**.

   ![Review and create the ASG](/images/5-Workshop/5.5-compute/auto-scaling-group-13.png)

   <p class="image-caption"><em>Figure 13: Review and create the ASG</em></p>
   Verify the Launch Template, private subnets, Target Group, capacity, and tags before creating the ASG.


14. **Verify the ASG after creation**

   Open the new ASG and inspect its instances, desired capacity, Launch Template, and health status.

   ![Verify the ASG after creation](/images/5-Workshop/5.5-compute/auto-scaling-group-14.png)

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
