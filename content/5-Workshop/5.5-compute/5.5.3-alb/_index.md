---
title : "Target Group and ALB"
date : 2024-01-01
weight : 3
chapter : false
pre : " <b> 5.5.3 </b> "
---

#### Target Group and Application Load Balancer

The Target Group health-checks the EC2 application on port `8080`. The Application Load Balancer is the public entry point that accepts HTTP/HTTPS requests and forwards them to the Target Group.

| Component | Configuration |
|---|---|
| Target type | Instances |
| Target Group protocol/port | HTTP: `8080` |
| Health check path | `/actuator/health` |
| ALB scheme | Internet-facing |
| ALB subnets | Two public subnets in two AZs |
| ALB Security Group | Allows inbound 80/443 from the Internet |
| Initial listener | HTTP:80 forwards to `sportbooking-tg` |

Recorded result: the Target Group uses the correct health check path, the ALB is active, and its DNS name can be tested before the domain is configured.

#### Target Group and ALB Procedure

Create and validate the Target Group health check before the public ALB sends traffic to the application, then test the website through the ALB DNS name.

**Procedure:**

1. **Start creating the Target Group**

   Open **EC2 > Target Groups**, then choose **Create target group**.

   ![Start creating the Target Group](/images/5-Workshop/5.5-compute/alb-01.png)

   <p class="image-caption"><em>Figure 1: Start creating the Target Group</em></p>
   The Target Group receives ALB requests for the EC2 application and performs health checks.


2. **Select the target type and protocol**

   Select **Instances** as the target type, enter the Target Group name, and select HTTP on port `8080`.

   ![Select the target type and protocol](/images/5-Workshop/5.5-compute/alb-02.png)

   <p class="image-caption"><em>Figure 2: Select the target type and protocol</em></p>
   The Spring Boot application runs on EC2 port 8080. The ALB accepts public traffic on 80/443 and forwards it internally to 8080.


3. **Select the VPC and protocol version**

   Select the SportBooking VPC and retain HTTP1 unless the application requires another protocol version.

   ![Select the VPC and protocol version](/images/5-Workshop/5.5-compute/alb-03.png)

   <p class="image-caption"><em>Figure 3: Select the VPC and protocol version</em></p>
   The Target Group must be in the same VPC as the EC2 application before instances can be registered.


4. **Configure the health check**

   Enter `/actuator/health` as the health check path and retain HTTP as the protocol.

   ![Configure the health check](/images/5-Workshop/5.5-compute/alb-04.png)

   <p class="image-caption"><em>Figure 4: Configure the health check</em></p>
   The health check allows the ALB to send traffic only to healthy application instances. `/actuator/health` reports Spring Boot status more accurately than `/`.


5. **Continue to target registration**

   Review the target selection and attributes, then choose **Next**.

   ![Complete the Target Group configuration](/images/5-Workshop/5.5-compute/alb-05.png)

   <p class="image-caption"><em>Figure 5: Complete the Target Group configuration and continue to target registration</em></p>
   After protocol and health check settings are complete, register the seed EC2 instance to test the ALB before creating the ASG.


6. **Select the seed EC2 instance as the initial target**

   Select `sportbooking-seed`, enter port `8080`, and choose **Include as pending below**.

   ![Select the instance when registering a target manually](/images/5-Workshop/5.5-compute/alb-06.png)

   <p class="image-caption"><em>Figure 6: Select sportbooking-seed as the target on port 8080</em></p>
   Register the seed instance to test the Target Group and ALB first; the Auto Scaling Group subsequently manages registration for the instances it creates.


7. **Review the target**

   Verify `sportbooking-seed`, port `8080`, and the `Running` state in the pending-target list, then choose **Create target group**.

   ![Review the registered target](/images/5-Workshop/5.5-compute/alb-07.png)

   <p class="image-caption"><em>Figure 7: Review the target</em></p>
   An incorrect target port is a common cause of failed health checks.


8. **Confirm Target Group creation**

   Review the `/actuator/health` health check and `sportbooking-seed` target, then choose **Create target group**.

   ![Confirm Target Group creation](/images/5-Workshop/5.5-compute/alb-08.png)

   <p class="image-caption"><em>Figure 8: Confirm Target Group creation</em></p>
   This final step makes the Target Group available for the ALB and ASG.


9. **Confirm the pending target registration**

   When the AWS Console displays the pending-target confirmation dialog, choose **Continue**.

   ![Confirm the pending target registration](/images/5-Workshop/5.5-compute/alb-09.png)

   <p class="image-caption"><em>Figure 9: Confirm the pending target registration</em></p>
   AWS warns that the target is awaiting a health check. The state changes to Healthy after the application runs successfully for several minutes.


10. **Verify the Target Group**

   Open the new Target Group and inspect its **Targets** and health check settings.

   ![Verify the Target Group](/images/5-Workshop/5.5-compute/alb-10.png)

   <p class="image-caption"><em>Figure 10: Verify the Target Group</em></p>
   A Healthy target is an important prerequisite before the domain points to the ALB.


11. **Start creating the Load Balancer**

   Open **EC2 > Load Balancers**, then choose **Create load balancer**.

   ![Start creating the Load Balancer](/images/5-Workshop/5.5-compute/alb-11.png)

   <p class="image-caption"><em>Figure 11: Start creating the Load Balancer</em></p>
   The Load Balancer is the website's public entry point, replacing direct access to private EC2 instances.


12. **Select Application Load Balancer**

   Select **Application Load Balancer**, then choose **Create**.

   ![Select Application Load Balancer](/images/5-Workshop/5.5-compute/alb-12.png)

   <p class="image-caption"><em>Figure 12: Select Application Load Balancer</em></p>
   An ALB supports HTTP/HTTPS web traffic, listeners, rules, Target Groups, and TLS termination.


13. **Enter the basic ALB configuration**

   Enter `sportbooking-alb` as the name, select the **Internet-facing** scheme, and use the IPv4 address type.

   ![Enter the basic ALB configuration](/images/5-Workshop/5.5-compute/alb-13.png)

   <p class="image-caption"><em>Figure 13: Enter the basic ALB configuration</em></p>
   The Internet-facing ALB receives user traffic while the EC2 application remains private.


14. **Select the VPC and public subnets**

   Select the SportBooking VPC and two public subnets in two AZs.

   ![Select the VPC and public subnets](/images/5-Workshop/5.5-compute/alb-14.png)

   <p class="image-caption"><em>Figure 14: Select the VPC and public subnets</em></p>
   The ALB requires public subnets to receive Internet requests and multiple AZs for improved availability.


15. **Select the Security Group and listener**

   Select the ALB Security Group that allows ports 80 and 443. Create the initial HTTP:80 listener and configure it to forward to `sportbooking-tg`.

   ![Select the Security Group and listener](/images/5-Workshop/5.5-compute/alb-15.png)

   <p class="image-caption"><em>Figure 15: Select the Security Group and listener</em></p>
   The ALB Security Group is the only public access point. The listener forwards requests to the application through the Target Group.


16. **Review and create the ALB**

   Review the VPC, subnets, Security Group, listener, and Target Group, then choose **Create load balancer**.

   ![Review and create the ALB](/images/5-Workshop/5.5-compute/alb-16.png)

   <p class="image-caption"><em>Figure 16: Review and create the ALB</em></p>
   Validate the VPC, two public subnets, Security Group, listener, and Target Group before creating the ALB.


17. **Confirm that the ALB was created**

   Open the new ALB and record `sportbooking-alb-1198360694.us-east-1.elb.amazonaws.com` as its DNS name.

   ![Confirm that the ALB was created](/images/5-Workshop/5.5-compute/alb-17.png)

   <p class="image-caption"><em>Figure 17: Confirm that the ALB was created</em></p>
   Test with the ALB DNS name before configuring the Route 53 Alias.


18. **Verify listeners and security**

   Open the listener and security tabs and confirm that the HTTP listener forwards to the correct Target Group.

   ![Verify ALB listeners and security](/images/5-Workshop/5.5-compute/alb-18.png)

   <p class="image-caption"><em>Figure 18: Verify listeners and security</em></p>
   An incorrect listener action can make the ALB reachable from a browser while preventing requests from reaching the application.


19. **Test the website through the ALB**

   Open the ALB DNS name or the configured domain and confirm that the SportBooking home page loads.

   ![Test the website through the ALB](/images/5-Workshop/5.5-compute/alb-19.png)

   <p class="image-caption"><em>Figure 19: Test the website through the ALB</em></p>
   This is end-to-end evidence of the path: user -> ALB -> Target Group -> EC2 application -> RDS/S3.
