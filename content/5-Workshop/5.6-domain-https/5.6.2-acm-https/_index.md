---
title : "ACM and HTTPS"
date : 2024-01-01
weight : 2
chapter : false
pre : " <b> 5.6.2 </b> "
---

#### Issued ACM Certificate

| Item | Configuration |
|---|---|
| Certificate type | Public certificate |
| Domain name | `sport.younglilliu.id.vn` |
| Validation method | DNS validation |
| Region | `us-east-1` |
| Recorded status | Issued |

Create the CNAME validation record in Route 53. After it appears in public DNS, ACM changes from `Pending validation` to `Issued`.

{{% notice note %}}
If ACM remains pending, verify the domain spelling, confirm that the hosted zone is public, and verify that the domain provider uses the Route 53 name servers.
{{% /notice %}}

#### ALB HTTPS Listener

| Item | Result/Configuration |
|---|---|
| HTTPS listener 443 | Forwards to `sportbooking-tg` and uses the `sport.younglilliu.id.vn` ACM certificate. |
| HTTP listener 80 | Redirects to HTTPS port 443 with status HTTP_301. |
| Security Group | The ALB Security Group allows inbound 80 and 443 from `0.0.0.0/0`. |

Recorded results:

- HTTP returns a 301 redirect.
- HTTPS returns 200 OK.
- The browser displays the website through the primary domain instead of the ALB DNS name.

#### Procedure

1. **Start requesting the ACM certificate**

   Open **AWS Certificate Manager**, then choose **Request a certificate**.

   ![Start requesting the ACM certificate](/images/5-Workshop/5.6-domain-https/route-53-08.png)

   <p class="image-caption"><em>Figure 1: Start requesting the ACM certificate</em></p>
   The ACM certificate enables HTTPS on the ALB.


2. **Select a public certificate**

   Select **Request a public certificate**, then choose **Next**.

   ![Select a public certificate](/images/5-Workshop/5.6-domain-https/route-53-09.png)

   <p class="image-caption"><em>Figure 2: Select a public certificate</em></p>
   A public website requires a publicly trusted certificate issued by ACM.


3. **Enter the certificate domain**

   Enter `sport.younglilliu.id.vn` and select DNS validation.

   ![Enter the certificate domain](/images/5-Workshop/5.6-domain-https/route-53-10.png)

   <p class="image-caption"><em>Figure 3: Enter the certificate domain</em></p>
   The certificate must match the domain used by visitors. DNS validation integrates directly with Route 53.


4. **Review and request the certificate**

   Review the key algorithm and tags, then choose **Request**.

   ![Review and request the certificate](/images/5-Workshop/5.6-domain-https/route-53-11.png)

   <p class="image-caption"><em>Figure 4: Review and request the certificate</em></p>
   An incorrect domain at this stage causes a name mismatch or leaves ACM waiting for validation.


5. **Open the certificate pending validation**

   Open the requested certificate and inspect its validation status.

   ![Open the certificate pending validation](/images/5-Workshop/5.6-domain-https/route-53-12.png)

   <p class="image-caption"><em>Figure 5: Open the certificate pending validation</em></p>
   ACM requires a public CNAME validation record before the status changes to Issued.


6. **Create the DNS validation record**

   Choose **Create records in Route 53**.

   ![Create the DNS validation record](/images/5-Workshop/5.6-domain-https/route-53-13.png)

   <p class="image-caption"><em>Figure 6: Create the DNS validation record</em></p>
   This action creates the CNAME validation record in the corresponding hosted zone and reduces copy errors.


7. **Confirm validation record creation**

   Review the validation CNAME record, then choose **Create records**.

   ![Confirm validation record creation](/images/5-Workshop/5.6-domain-https/route-53-14.png)

   <p class="image-caption"><em>Figure 7: Confirm validation record creation</em></p>
   The validation record proves control of the domain so ACM can issue the certificate.


8. **Open the ALB to add an HTTPS listener**

   Open **EC2 > Load Balancers**, select the SportBooking ALB, and choose **Add listener**.

   ![Open the ALB to add an HTTPS listener](/images/5-Workshop/5.6-domain-https/route-53-15.png)

   <p class="image-caption"><em>Figure 8: Open the ALB to add an HTTPS listener</em></p>
   After the certificate is issued, add a port 443 listener for browser HTTPS access.


9. **Configure the HTTPS listener**

   Select **HTTPS**, set port `443`, and configure forwarding to `sportbooking-tg`.

   ![Configure the HTTPS listener](/images/5-Workshop/5.6-domain-https/route-53-16.png)

   <p class="image-caption"><em>Figure 9: Configure the HTTPS listener</em></p>
   The ALB terminates TLS on port 443 and forwards internal HTTP traffic to application port 8080.


10. **Select the ACM certificate**

   Select the `sport.younglilliu.id.vn` certificate, then choose **Add**.

   ![Select the ACM certificate](/images/5-Workshop/5.6-domain-https/route-53-17.png)

   <p class="image-caption"><em>Figure 10: Select the ACM certificate</em></p>
   Confirm that the certificate is `Issued` before attaching it to the HTTPS listener.


11. **Verify the new listener**

   Open **Listeners and rules** and confirm that listener 443 forwards to the correct Target Group.

   ![Verify the listener after creation](/images/5-Workshop/5.6-domain-https/route-53-18.png)

   <p class="image-caption"><em>Figure 11: Verify the listener after creation</em></p>
   This verifies that HTTPS is enabled on the ALB.


12. **Change the HTTP listener to a redirect**

   Select the HTTP:80 listener, then choose **Edit listener**.

   ![Change the HTTP listener to a redirect](/images/5-Workshop/5.6-domain-https/route-53-19.png)

   <p class="image-caption"><em>Figure 12: Change the HTTP listener to a redirect</em></p>
   Change the HTTP listener from forward to redirect after the HTTPS listener is operational.


13. **Configure the HTTP-to-HTTPS redirect**

   Select **Redirect to URL**, set the protocol to HTTPS and port to 443, then choose **Save changes**.

   ![Configure the HTTP-to-HTTPS redirect](/images/5-Workshop/5.6-domain-https/route-53-20.png)

   <p class="image-caption"><em>Figure 13: Configure the HTTP-to-HTTPS redirect</em></p>
   A 301 redirect sends all HTTP requests to HTTPS and supports OAuth callbacks and secure cookies.


14. **Verify listeners 80 and 443**

   Inspect the HTTP and HTTPS listeners and confirm that port 80 redirects while port 443 forwards to the Target Group.

   ![Verify listeners 80 and 443](/images/5-Workshop/5.6-domain-https/route-53-21.png)

   <p class="image-caption"><em>Figure 14: Verify listeners 80 and 443</em></p>
   The final state is HTTP port `80` redirecting to HTTPS and HTTPS port `443` forwarding to the Target Group.
