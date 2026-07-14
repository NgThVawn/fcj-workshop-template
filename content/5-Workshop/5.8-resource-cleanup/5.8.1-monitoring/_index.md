---
title : "Monitoring Cleanup"
date : 2024-01-01
weight : 1
chapter : false
pre : " <b> 5.8.1 </b> "
---

#### 1. Delete the CloudWatch Alarm

The CloudWatch alarm was deleted before SNS so that no alarm action continued to reference the notification topic.

**Procedure:**

1. **Select the alarm**

   On the console, perform the following in order: **(1)** select the `SB-ALB-UnhealthyTargets` alarm; **(2)** open **Actions**; and **(3)** select **Delete**.

   ![Select the CloudWatch alarm to delete](/images/5-Workshop/5.8-resource-cleanup/cleanup-cloudwatch-alarm-01.png)

   <p class="image-caption"><em>Figure 1: Select the CloudWatch alarm to delete</em></p>
   This alarm monitors the SportBooking Target Group and sends notifications to SNS. Deleting the alarm first removes the monitoring link before the source resources and notification channel are deleted.


2. **Confirm alarm deletion**

   Verify the name `SB-ALB-UnhealthyTargets`, then select **Delete**.

   ![Confirm CloudWatch alarm deletion](/images/5-Workshop/5.8-resource-cleanup/cleanup-cloudwatch-alarm-02.png)

   <p class="image-caption"><em>Figure 2: Confirm CloudWatch alarm deletion</em></p>
   This action removes only the alarm configuration; it does not delete the historical metrics of the source service.


#### 2. Delete the SNS Subscription

The subscription was deleted before the topic to explicitly disconnect the email endpoint from the alert channel.

**Procedure:**

1. **Select the subscription**

   On the console, perform the following in order: **(1)** open **SNS > Subscriptions** and select the confirmed project subscription; and **(2)** select **Delete**.

   ![Select the SNS subscription to delete](/images/5-Workshop/5.8-resource-cleanup/cleanup-sns-subscription-01.png)

   <p class="image-caption"><em>Figure 3: Select the SNS subscription to delete</em></p>
   The subscription connects the email address that receives alerts to the `sportbooking-prod-alerts` topic.


2. **Confirm subscription deletion**

   Verify the endpoint and subscription ID, then select **Delete**.

   ![Confirm SNS subscription deletion](/images/5-Workshop/5.8-resource-cleanup/cleanup-sns-subscription-02.png)

   <p class="image-caption"><em>Figure 4: Confirm SNS subscription deletion</em></p>
   After this step, the email endpoint no longer receives notifications from the topic.


3. **Verify the result**

   Refresh the subscription list. Confirm the **Subscription deleted successfully** message and verify that the project subscription no longer appears.

   ![Verify that the SNS subscription has been deleted](/images/5-Workshop/5.8-resource-cleanup/cleanup-sns-subscription-03.png)

   <p class="image-caption"><em>Figure 5: Verify that the SNS subscription has been deleted</em></p>
   This verification prevents an unused endpoint from remaining in the account.


#### 3. Delete the SNS Topic

**Procedure:**

1. **Select the alert topic**

   On the console, perform the following in order: **(1)** open **SNS > Topics** and select `sportbooking-prod-alerts`; and **(2)** select **Delete**.

   ![Select the SNS topic to delete](/images/5-Workshop/5.8-resource-cleanup/cleanup-sns-topic-01.png)

   <p class="image-caption"><em>Figure 6: Select the SNS topic to delete</em></p>
   The CloudWatch alarm and subscription have already been removed, so the topic has no remaining dependency within the project scope.


2. **Confirm topic deletion**

   On the console, perform the following in order: **(1)** enter `delete me` as required; and **(2)** select **Delete**.

   ![Confirm SNS topic deletion](/images/5-Workshop/5.8-resource-cleanup/cleanup-sns-topic-02.png)

   <p class="image-caption"><em>Figure 7: Confirm SNS topic deletion</em></p>
   Topic deletion cannot be undone. The confirmation text reduces the risk of deleting the wrong notification channel.


3. **Verify that the topic has been deleted**

   Confirm the **Topic deleted successfully** message, then verify that the project topic no longer appears in the list.

   ![Verify that the SNS topic has been deleted](/images/5-Workshop/5.8-resource-cleanup/cleanup-sns-topic-03.png)

   <p class="image-caption"><em>Figure 8: Verify that the SNS topic has been deleted</em></p>
   This result completes the cleanup of the SNS alert channel.
