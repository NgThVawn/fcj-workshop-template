---
title : "Route 53, ACM, HTTPS, and Testing"
date : 2024-01-01
weight : 6
chapter : false
pre : " <b> 5.6. </b> "
---

#### Deployment Scope

Configure the `sport.younglilliu.id.vn` domain, issue a certificate with ACM, attach an HTTPS listener to the ALB, and test the deployed website.

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

#### Test DNS and HTTPS with PowerShell

```powershell
nslookup -type=NS younglilliu.id.vn
nslookup sport.younglilliu.id.vn

curl.exe -I http://sport.younglilliu.id.vn
curl.exe -I https://sport.younglilliu.id.vn
curl.exe -I https://sport.younglilliu.id.vn/auth/login
curl.exe -I https://sport.younglilliu.id.vn/auth/register
curl.exe -I https://sport.younglilliu.id.vn/actuator/health
```

Recorded results:

- `http://sport.younglilliu.id.vn` returns `HTTP/1.1 301` with a `Location: https://...` header.
- `https://sport.younglilliu.id.vn` returns `HTTP/1.1 200`.
- The session cookie includes secure attributes when served through HTTPS.

#### Test a New EC2 Instance

Connect to EC2 through Session Manager:

```bash
sudo -i
systemctl status sportbooking --no-pager
curl -I http://127.0.0.1:8080/
curl http://127.0.0.1:8080/actuator/health
grep -n -E "APP_BASE_URL|APP_UPLOADS_BASE_URL|SERVER_FORWARD_HEADERS_STRATEGY|APP_STORAGE_TYPE|APP_STORAGE_S3_BUCKET|APP_STORAGE_S3_KEY_PREFIX|AWS_REGION|AWS_DEFAULT_REGION|AWS_S3_REGION" /opt/sportbooking/app.env
sha256sum /opt/sportbooking/app.jar
aws s3 cp s3://sportbooking-artifacts-3stars-us-east-1/releases/SportBooingSystem-0.0.1-SNAPSHOT.jar /tmp/latest-app.jar --region us-east-1
sha256sum /opt/sportbooking/app.jar /tmp/latest-app.jar
```

Recorded results:

- The `sportbooking` service is `active (running)`.
- The local health check returns OK.
- The application environment contains the HTTPS domain, S3 storage settings, and `us-east-1` Region.
- The JAR hash on EC2 matches the hash of the JAR downloaded from S3.

#### Route 53, ACM, and HTTPS Procedure

Publish the website by creating the hosted zone and configuring its name servers, adding the Alias record and ACM DNS validation, completing the HTTPS listener, and testing the domain.

**Procedure:**

1. **Start creating the Hosted Zone**

   Open **Route 53 > Hosted zones**, then choose **Create hosted zone**.

   ![Start creating the Hosted Zone](/images/5-Workshop/5.6-Cleanup/route-53-01.png)

   <p class="image-caption"><em>Figure 1: Start creating the Hosted Zone</em></p>
   The Hosted Zone manages DNS for the SportBooking website domain.


2. **Enter the Hosted Zone domain**

   Enter the root domain `younglilliu.id.vn` and select **Public hosted zone**.

   ![Enter the Hosted Zone domain](/images/5-Workshop/5.6-Cleanup/route-53-02.png)

   <p class="image-caption"><em>Figure 2: Enter the Hosted Zone domain</em></p>
   A public hosted zone allows the Internet to resolve the domain to the ALB. Do not create a hosted zone for a misspelled domain.


3. **Create the Hosted Zone**

   Verify the domain name, then choose **Create hosted zone**.

   ![Create the Hosted Zone](/images/5-Workshop/5.6-Cleanup/route-53-03.png)

   <p class="image-caption"><em>Figure 3: Create the Hosted Zone</em></p>
   AWS generates a set of name servers that must be configured at the domain provider.


4. **Record the name servers**

   Open the new hosted zone and record the four NS records assigned by Route 53.

   ![Record the Route 53 name servers](/images/5-Workshop/5.6-Cleanup/route-53-04.png)

   <p class="image-caption"><em>Figure 4: Record the name servers</em></p>
   Route 53 manages the domain only after the domain provider delegates it to these four name servers.


5. **Start creating the subdomain record**

   In the hosted zone, choose **Create record**.

   ![Start creating the subdomain record](/images/5-Workshop/5.6-Cleanup/route-53-05.png)

   <p class="image-caption"><em>Figure 5: Start creating the subdomain record</em></p>
   The `sport.younglilliu.id.vn` record sends users to the ALB instead of requiring the ALB DNS name.


6. **Create an Alias record for the ALB**

   Enter `sport` as the record name, select type `A`, and enable **Alias**. Select the project's Application Load Balancer, then choose **Create records**.

   ![Create an Alias record for the ALB](/images/5-Workshop/5.6-Cleanup/route-53-06.png)

   <p class="image-caption"><em>Figure 6: Create an Alias record for the ALB</em></p>
   An Alias record is the correct way to point a domain to an ALB because an ALB has no fixed static IP address.


7. **Confirm that the record was created**

   Inspect the record list and confirm that `sport` appears alongside the NS and SOA records.

   ![Confirm that the Alias record was created](/images/5-Workshop/5.6-Cleanup/route-53-07.png)

   <p class="image-caption"><em>Figure 7: Confirm that the record was created</em></p>
   If the record is missing or was created in a hosted zone for another domain, browsers cannot resolve the website.


8. **Start requesting the ACM certificate**

   Open **AWS Certificate Manager**, then choose **Request a certificate**.

   ![Start requesting the ACM certificate](/images/5-Workshop/5.6-Cleanup/route-53-08.png)

   <p class="image-caption"><em>Figure 8: Start requesting the ACM certificate</em></p>
   The ACM certificate enables HTTPS on the ALB.


9. **Select a public certificate**

   Select **Request a public certificate**, then choose **Next**.

   ![Select a public certificate](/images/5-Workshop/5.6-Cleanup/route-53-09.png)

   <p class="image-caption"><em>Figure 9: Select a public certificate</em></p>
   A public website requires a publicly trusted certificate issued by ACM.


10. **Enter the certificate domain**

   Enter `sport.younglilliu.id.vn` and select DNS validation.

   ![Enter the certificate domain](/images/5-Workshop/5.6-Cleanup/route-53-10.png)

   <p class="image-caption"><em>Figure 10: Enter the certificate domain</em></p>
   The certificate must match the domain used by visitors. DNS validation integrates directly with Route 53.


11. **Review and request the certificate**

   Review the key algorithm and tags, then choose **Request**.

   ![Review and request the certificate](/images/5-Workshop/5.6-Cleanup/route-53-11.png)

   <p class="image-caption"><em>Figure 11: Review and request the certificate</em></p>
   An incorrect domain at this stage causes a name mismatch or leaves ACM waiting for validation.


12. **Open the certificate pending validation**

   Open the requested certificate and inspect its validation status.

   ![Open the certificate pending validation](/images/5-Workshop/5.6-Cleanup/route-53-12.png)

   <p class="image-caption"><em>Figure 12: Open the certificate pending validation</em></p>
   ACM requires a public CNAME validation record before the status changes to Issued.


13. **Create the DNS validation record**

   Choose **Create records in Route 53**.

   ![Create the DNS validation record](/images/5-Workshop/5.6-Cleanup/route-53-13.png)

   <p class="image-caption"><em>Figure 13: Create the DNS validation record</em></p>
   This action creates the CNAME validation record in the corresponding hosted zone and reduces copy errors.


14. **Confirm validation record creation**

   Review the validation CNAME record, then choose **Create records**.

   ![Confirm validation record creation](/images/5-Workshop/5.6-Cleanup/route-53-14.png)

   <p class="image-caption"><em>Figure 14: Confirm validation record creation</em></p>
   The validation record proves control of the domain so ACM can issue the certificate.


15. **Open the ALB to add an HTTPS listener**

   Open **EC2 > Load Balancers**, select the SportBooking ALB, and choose **Add listener**.

   ![Open the ALB to add an HTTPS listener](/images/5-Workshop/5.6-Cleanup/route-53-15.png)

   <p class="image-caption"><em>Figure 15: Open the ALB to add an HTTPS listener</em></p>
   After the certificate is issued, add a port 443 listener for browser HTTPS access.


16. **Configure the HTTPS listener**

   Select **HTTPS**, set port `443`, and configure forwarding to `sportbooking-tg`.

   ![Configure the HTTPS listener](/images/5-Workshop/5.6-Cleanup/route-53-16.png)

   <p class="image-caption"><em>Figure 16: Configure the HTTPS listener</em></p>
   The ALB terminates TLS on port 443 and forwards internal HTTP traffic to application port 8080.


17. **Select the ACM certificate**

   Select the `sport.younglilliu.id.vn` certificate, then choose **Add**.

   ![Select the ACM certificate](/images/5-Workshop/5.6-Cleanup/route-53-17.png)

   <p class="image-caption"><em>Figure 17: Select the ACM certificate</em></p>
   Confirm that the certificate is `Issued` before attaching it to the HTTPS listener.


18. **Verify the new listener**

   Open **Listeners and rules** and confirm that listener 443 forwards to the correct Target Group.

   ![Verify the listener after creation](/images/5-Workshop/5.6-Cleanup/route-53-18.png)

   <p class="image-caption"><em>Figure 18: Verify the listener after creation</em></p>
   This verifies that HTTPS is enabled on the ALB.


19. **Change the HTTP listener to a redirect**

   Select the HTTP:80 listener, then choose **Edit listener**.

   ![Change the HTTP listener to a redirect](/images/5-Workshop/5.6-Cleanup/route-53-19.png)

   <p class="image-caption"><em>Figure 19: Change the HTTP listener to a redirect</em></p>
   Change the HTTP listener from forward to redirect after the HTTPS listener is operational.


20. **Configure the HTTP-to-HTTPS redirect**

   Select **Redirect to URL**, set the protocol to HTTPS and port to 443, then choose **Save changes**.

   ![Configure the HTTP-to-HTTPS redirect](/images/5-Workshop/5.6-Cleanup/route-53-20.png)

   <p class="image-caption"><em>Figure 20: Configure the HTTP-to-HTTPS redirect</em></p>
   A 301 redirect sends all HTTP requests to HTTPS and supports OAuth callbacks and secure cookies.


21. **Verify listeners 80 and 443**

   Inspect the HTTP and HTTPS listeners and confirm that port 80 redirects while port 443 forwards to the Target Group.

   ![Verify listeners 80 and 443](/images/5-Workshop/5.6-Cleanup/route-53-21.png)

   <p class="image-caption"><em>Figure 21: Verify listeners 80 and 443</em></p>
   The final state is HTTP port `80` redirecting to HTTPS and HTTPS port `443` forwarding to the Target Group.


22. **Test the website with the domain**

   Open `https://sport.younglilliu.id.vn` and confirm that the SportBooking page loads.

   ![Test the website with the domain](/images/5-Workshop/5.6-Cleanup/route-53-22.png)

   <p class="image-caption"><em>Figure 22: Test the website with the domain</em></p>
   This final step validates the complete DNS -> HTTPS -> ALB -> EC2 application path.



#### CloudFront Evaluation Result

An attempt was made to configure CloudFront with the ALB as an origin to evaluate placing a CDN in front of the website. The AWS Console required account verification before a new CloudFront resource could be created. As a result, no distribution was created and CloudFront was not part of the tested request path.

{{% notice info %}}
The `sportbooking-alb-2038054256.us-east-1.elb.amazonaws.com` DNS name shown in the CloudFront image belongs to an earlier test ALB configuration. The completed configuration uses `sportbooking-alb-1198360694.us-east-1.elb.amazonaws.com`, and Route 53 points directly to this ALB.
{{% /notice %}}

**Procedure:**

1. **Record the account-verification error during CloudFront creation**

   Open **Create distribution** and select the ALB as the test origin. Review the cache settings and security protections, then record the message requiring account verification before CloudFront resources can be added.

   ![Record the account-verification error during CloudFront creation](/images/5-Workshop/5.6-Cleanup/cloudfront-01.png)

   <p class="image-caption"><em>Figure 1: CloudFront was not created because account verification was incomplete</em></p>
   The evaluation stops at this screen. CloudFront and WAF remain target-architecture extensions and are not reported as deployed services.

