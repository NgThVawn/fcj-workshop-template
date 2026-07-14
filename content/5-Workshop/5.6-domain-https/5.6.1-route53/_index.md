---
title : "Route 53"
date : 2024-01-01
weight : 1
chapter : false
pre : " <b> 5.6.1 </b> "
---

#### Hosted Zone and Name Server Configuration

| Item | Recorded configuration |
|---|---|
| Hosted zone | `younglilliu.id.vn` |
| Application subdomain | `sport.younglilliu.id.vn` |
| PA Vietnam | The four Route 53 name servers replaced all previous name servers. |
| Verified error | A spelling mismatch between `youngliliu.id.vn` and `younglilliu.id.vn` prevented DNS resolution and kept ACM in the pending state. |

Verify delegation with:

```powershell
nslookup -type=NS younglilliu.id.vn
```

#### ALB Alias Record

| Item | Configuration |
|---|---|
| Record name | `sport.younglilliu.id.vn` |
| Type | A |
| Alias | Yes |
| Route traffic to | Application Load Balancer `sportbooking-alb-1198360694.us-east-1.elb.amazonaws.com` |
| Reason for Alias | A Route 53 Alias can point to an ALB without requiring a static IP address. |

Verify the record with:

```powershell
nslookup sport.younglilliu.id.vn
```

#### Procedure

Publish the website by creating the hosted zone and configuring its name servers, adding the Alias record and ACM DNS validation, completing the HTTPS listener, and testing the domain.

**Procedure:**

1. **Start creating the Hosted Zone**

   Open **Route 53 > Hosted zones**, then choose **Create hosted zone**.

   ![Start creating the Hosted Zone](/images/5-Workshop/5.6-domain-https/route-53-01.png)

   <p class="image-caption"><em>Figure 1: Start creating the Hosted Zone</em></p>
   The Hosted Zone manages DNS for the SportBooking website domain.


2. **Enter the Hosted Zone domain**

   Enter the root domain `younglilliu.id.vn` and select **Public hosted zone**.

   ![Enter the Hosted Zone domain](/images/5-Workshop/5.6-domain-https/route-53-02.png)

   <p class="image-caption"><em>Figure 2: Enter the Hosted Zone domain</em></p>
   A public hosted zone allows the Internet to resolve the domain to the ALB. Do not create a hosted zone for a misspelled domain.


3. **Create the Hosted Zone**

   Verify the domain name, then choose **Create hosted zone**.

   ![Create the Hosted Zone](/images/5-Workshop/5.6-domain-https/route-53-03.png)

   <p class="image-caption"><em>Figure 3: Create the Hosted Zone</em></p>
   AWS generates a set of name servers that must be configured at the domain provider.


4. **Record the name servers**

   Open the new hosted zone and record the four NS records assigned by Route 53.

   ![Record the Route 53 name servers](/images/5-Workshop/5.6-domain-https/route-53-04.png)

   <p class="image-caption"><em>Figure 4: Record the name servers</em></p>
   Route 53 manages the domain only after the domain provider delegates it to these four name servers.


5. **Start creating the subdomain record**

   In the hosted zone, choose **Create record**.

   ![Start creating the subdomain record](/images/5-Workshop/5.6-domain-https/route-53-05.png)

   <p class="image-caption"><em>Figure 5: Start creating the subdomain record</em></p>
   The `sport.younglilliu.id.vn` record sends users to the ALB instead of requiring the ALB DNS name.


6. **Create an Alias record for the ALB**

   Enter `sport` as the record name, select type `A`, and enable **Alias**. Select the project's Application Load Balancer, then choose **Create records**.

   ![Create an Alias record for the ALB](/images/5-Workshop/5.6-domain-https/route-53-06.png)

   <p class="image-caption"><em>Figure 6: Create an Alias record for the ALB</em></p>
   An Alias record is the correct way to point a domain to an ALB because an ALB has no fixed static IP address.


7. **Confirm that the record was created**

   Inspect the record list and confirm that `sport` appears alongside the NS and SOA records.

   ![Confirm that the Alias record was created](/images/5-Workshop/5.6-domain-https/route-53-07.png)

   <p class="image-caption"><em>Figure 7: Confirm that the record was created</em></p>
   If the record is missing or was created in a hosted zone for another domain, browsers cannot resolve the website.
