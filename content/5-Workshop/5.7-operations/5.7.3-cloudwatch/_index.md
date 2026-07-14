---
title : "CloudWatch Monitoring"
date : 2024-01-01
weight : 3
chapter : false
pre : " <b> 5.7.3 </b> "
---

#### CloudWatch Monitoring

CloudWatch monitors operational metrics for EC2, the ALB and Target Group, and RDS. Create `SB-ALB-UnhealthyTargets` after SNS and configure it to monitor `UnHealthyHostCount` for `sportbooking-tg`.

| Resource | Verified metrics | Purpose |
|---|---|---|
| EC2/ASG | CPUUtilization, StatusCheckFailed | Detects unhealthy or overloaded instances. |
| ALB | HTTPCode_Target_5XX_Count, TargetResponseTime, HealthyHostCount | Monitors backend errors and target health. |
| RDS | CPUUtilization, FreeStorageSpace, DatabaseConnections | Monitors the database during operation. |

#### CloudWatch Alarm Procedure

Use a Target Group metric, require the threshold for two consecutive periods, and send the alarm notification through SNS.

**Procedure:**

1. **Start creating the CloudWatch alarm**

   Open **CloudWatch > Alarms**, then choose **Create alarm**.

   ![Start creating the CloudWatch alarm](/images/5-Workshop/5.7-operations/cloudwatch-01.png)

   <p class="image-caption"><em>Figure 1: Start creating the CloudWatch alarm</em></p>
   An alarm provides early detection when EC2, the ALB, or RDS fails or a metric crosses its threshold.


2. **Select a metric**

   Choose **Select metric**, then select the **ApplicationELB** namespace.

   ![Select a CloudWatch metric](/images/5-Workshop/5.7-operations/cloudwatch-02.png)

   <p class="image-caption"><em>Figure 2: Select a metric</em></p>
   This alarm focuses on targets that fail health checks behind the ALB.


3. **Open the ApplicationELB namespace**

   Open the Application Load Balancer metric category and select the metric type associated with a Target Group or Load Balancer.

   ![Open the ApplicationELB namespace](/images/5-Workshop/5.7-operations/cloudwatch-03.png)

   <p class="image-caption"><em>Figure 3: Select the ApplicationELB namespace</em></p>
   ALB metrics indicate whether backends are healthy and whether users encounter 5xx errors.


4. **Select the specific metric**

   Select `UnHealthyHostCount` for `sportbooking-alb` and `sportbooking-tg`, then choose **Select metric**.

   ![Select the UnHealthyHostCount metric](/images/5-Workshop/5.7-operations/cloudwatch-04.png)

   <p class="image-caption"><em>Figure 4: Select the specific metric</em></p>
   This metric increases when one or more EC2 application instances behind the ALB fail their health checks.


5. **Set the alarm condition**

   Select **Maximum** as the statistic and set the period to **1 minute**.

   ![Set the alarm condition](/images/5-Workshop/5.7-operations/cloudwatch-05.png)

   <p class="image-caption"><em>Figure 5: Set the alarm condition</em></p>
   This statistic and period capture the highest unhealthy-target count in each one-minute interval.


6. **Configure the threshold and datapoints**

   Set the condition to `UnHealthyHostCount >= 1` and **Datapoints to alarm** to `2 out of 2`. Keep **Treat missing data as missing**, then choose **Next**.

   ![Configure the threshold and datapoints](/images/5-Workshop/5.7-operations/cloudwatch-06.png)

   <p class="image-caption"><em>Figure 6: Configure the threshold and datapoints</em></p>
   Requiring two datapoints reduces false alerts caused by one brief anomalous data point.


7. **Select the notification action**

   Select the alarm state that sends a notification, then select the SNS topic created earlier.

   ![Select the notification action](/images/5-Workshop/5.7-operations/cloudwatch-07.png)

   <p class="image-caption"><em>Figure 7: Select the notification action</em></p>
   An alarm requires a notification channel to be operationally useful. SNS sends email when the system has a problem.


8. **Skip unused additional actions**

   Do not configure EC2, Auto Scaling, or Systems Manager actions for this alarm. Choose **Next**.

   ![Skip unused additional actions](/images/5-Workshop/5.7-operations/cloudwatch-08.png)

   <p class="image-caption"><em>Figure 8: Skip unused additional actions</em></p>
   This alarm sends a notification through SNS and does not modify resources automatically.


9. **Name the alarm**

   Enter `SB-ALB-UnhealthyTargets` as the alarm name. Leave the description and tags empty for this creation, then choose **Next**.

   ![Name the CloudWatch alarm](/images/5-Workshop/5.7-operations/cloudwatch-09.png)

   <p class="image-caption"><em>Figure 9: Name the alarm</em></p>
   The alarm name should identify the resource and condition so the issue is immediately clear in an email.


10. **Review and create the alarm**

   Review the metric, condition, action, and name, then choose **Create alarm**.

   ![Review and create the CloudWatch alarm](/images/5-Workshop/5.7-operations/cloudwatch-10.png)

   <p class="image-caption"><em>Figure 10: Review and create the alarm</em></p>
   Validate the metric, condition, SNS action, and alarm name before creation.


11. **Verify the alarm in the list**

   Return to **Alarms** and confirm that `SB-ALB-UnhealthyTargets` appears with actions enabled and an initial `Insufficient data` state.

   ![Verify the alarm in the list](/images/5-Workshop/5.7-operations/cloudwatch-11.png)

   <p class="image-caption"><em>Figure 11: Verify the alarm in the list</em></p>
   The OK, Alarm, and Insufficient data states show whether CloudWatch has received metrics and evaluated the condition.
