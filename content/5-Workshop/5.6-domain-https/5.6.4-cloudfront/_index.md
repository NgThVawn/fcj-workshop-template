---
title : "CloudFront Evaluation"
date : 2024-01-01
weight : 4
chapter : false
pre : " <b> 5.6.4 </b> "
---

#### CloudFront Evaluation Result

An attempt was made to configure CloudFront with the ALB as an origin to evaluate placing a CDN in front of the website. The AWS Console required account verification before a new CloudFront resource could be created. As a result, no distribution was created and CloudFront was not part of the tested request path.

{{% notice info %}}
The `sportbooking-alb-2038054256.us-east-1.elb.amazonaws.com` DNS name shown in the CloudFront image belongs to an earlier test ALB configuration. The completed configuration uses `sportbooking-alb-1198360694.us-east-1.elb.amazonaws.com`, and Route 53 points directly to this ALB.
{{% /notice %}}

**Procedure:**

1. **Record the account-verification error during CloudFront creation**

   Open **Create distribution** and select the ALB as the test origin. Review the cache settings and security protections, then record the message requiring account verification before CloudFront resources can be added.

   ![Record the account-verification error during CloudFront creation](/images/5-Workshop/5.6-domain-https/cloudfront-01.png)

   <p class="image-caption"><em>Figure 1: CloudFront was not created because account verification was incomplete</em></p>
   The evaluation stops at this screen. CloudFront and WAF remain target-architecture extensions and are not reported as deployed services.
