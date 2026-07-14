---
title : "Operations"
date : 2024-01-01
weight : 7
chapter : false
pre : " <b> 5.7. </b> "
aliases:
  - /5-workshop/5.7-operation/
---

#### Deployment Scope

This section presents the operational workflow after the website is available through HTTPS: update the artifact, update the secret, verify the database, configure OAuth, create an SNS notification channel, create a CloudWatch alarm, and document common errors.

#### Contents

1. [Application Updates](5.7.1-updates/)
2. [SNS Alerts](5.7.2-sns/)
3. [CloudWatch Monitoring](5.7.3-cloudwatch/)
4. [Troubleshooting](5.7.4-troubleshooting/)

{{% notice info %}}
The artifact and secret are updated before Instance Refresh begins. The SNS channel must be confirmed before it is attached to the CloudWatch alarm.
{{% /notice %}}
