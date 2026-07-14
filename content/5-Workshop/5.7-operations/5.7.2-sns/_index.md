---
title : "SNS Alerts"
date : 2024-01-01
weight : 2
chapter : false
pre : " <b> 5.7.2 </b> "
---

#### SNS Alerting

SNS provides the notification channel for the CloudWatch alarm. Create and confirm the topic and email subscription before creating the alarm so the action has a valid destination.

| Step | Verified information |
|---|---|
| 1 | The SNS topic is created in `us-east-1`. |
| 2 | The email subscription is `Confirmed`. |
| 3 | The CloudWatch alarm action uses the correct SNS topic. |
| 4 | An alert email is delivered when the alarm changes state. |

#### SNS Procedure

Configure the notification channel with an SNS topic, an email subscription, and confirmation of the recipient address.

**Procedure:**

1. **Start creating the SNS topic**

   Open **SNS > Topics**, then choose **Create topic**.

   ![Start creating the SNS topic](/images/5-Workshop/5.7-operations/sns-01.png)

   <p class="image-caption"><em>Figure 1: Start creating the SNS topic</em></p>
   The SNS topic receives notifications from the CloudWatch alarm.


2. **Select the topic type and enter a name**

   Select **Standard** and enter `sportbooking-prod-alerts` as the topic name.

   ![Select the topic type and enter a name](/images/5-Workshop/5.7-operations/sns-02.png)

   <p class="image-caption"><em>Figure 2: Select the topic type and enter a name</em></p>
   A Standard topic is suitable for simple email alerts that do not require FIFO ordering.


3. **Create the topic**

   Keep the delivery and tag settings at their defaults, then choose **Create topic**.

   ![Create the SNS topic](/images/5-Workshop/5.7-operations/sns-03.png)

   <p class="image-caption"><em>Figure 3: Create the topic</em></p>
   The Standard topic is used for CloudWatch alarm emails; advanced delivery settings are not enabled.


4. **Open the new topic**

   Verify the topic ARN and details.

   ![Open the new topic](/images/5-Workshop/5.7-operations/sns-04.png)

   <p class="image-caption"><em>Figure 4: Open the new topic</em></p>
   Select this topic ARN as the notification action in the CloudWatch alarm.


5. **Start creating a subscription**

   In the topic, choose **Create subscription**.

   ![Start creating a subscription](/images/5-Workshop/5.7-operations/sns-05.png)

   <p class="image-caption"><em>Figure 5: Start creating a subscription</em></p>
   Without a subscription, the alarm publishes to the topic but no endpoint receives the message.


6. **Enter the protocol and endpoint**

   Select **Email** as the protocol and enter the alert recipient address.

   ![Enter the subscription protocol and endpoint](/images/5-Workshop/5.7-operations/sns-06.png)

   <p class="image-caption"><em>Figure 6: Enter the protocol and endpoint</em></p>
   Email is the simplest notification method for this test deployment.


7. **Create the subscription**

   Verify the email address, then choose **Create subscription**.

   ![Create the SNS subscription](/images/5-Workshop/5.7-operations/sns-07.png)

   <p class="image-caption"><em>Figure 7: Create the subscription</em></p>
   AWS sends a confirmation email; the subscription remains pending until its confirmation link is opened.


8. **Open the confirmation email**

   Open the mailbox and select the **AWS Notification - Subscription Confirmation** email.

   ![Open the subscription confirmation email](/images/5-Workshop/5.7-operations/sns-08.png)

   <p class="image-caption"><em>Figure 8: Open the confirmation email</em></p>
   This security step confirms that the address agrees to receive alerts.


9. **Confirm the subscription**

   Select **Confirm subscription** in the email.

   ![Confirm the SNS subscription](/images/5-Workshop/5.7-operations/sns-09.png)

   <p class="image-caption"><em>Figure 9: Confirm the subscription</em></p>
   The endpoint receives messages from the SNS topic only after confirmation.


10. **Verify the confirmed subscription**

   Return to the SNS subscription and confirm that its status is **Confirmed**.

   ![Verify the confirmed subscription](/images/5-Workshop/5.7-operations/sns-10.png)

   <p class="image-caption"><em>Figure 10: Verify the confirmed subscription</em></p>
   Confirm the subscription before associating the topic with the CloudWatch alarm.
