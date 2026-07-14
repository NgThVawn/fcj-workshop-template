---
title : "Amazon RDS"
date : 2024-01-01
weight : 2
chapter : false
pre : " <b> 5.4.2 </b> "
aliases:
  - /5-workshop/5.4-s3-onprem/5.4.2-rds/
---

#### RDS Configuration

Create a DB subnet group containing two private database subnets in two Availability Zones, then deploy Amazon RDS for MySQL using a Multi-AZ configuration. The database does not use public subnets and is not publicly accessible.

| Item | Configuration |
|---|---|
| Subnet group | Two private database subnets in two AZs. |
| Deployment | Multi-AZ DB instance deployment with a primary and standby. |
| DB instance class | `db.t3.small`. |
| Storage | General Purpose SSD `gp3`, `20 GiB`. |
| Public access | No. |
| Security Group | `sportbooking-rds`, allowing `3306` only from `sportbooking-app`. |
| Database name | `sport_booking`. |
| Master username | `sportbooking_ad`. |
| Endpoint | `sportbooking-mysql.cudgu8qk8z19.us-east-1.rds.amazonaws.com`. |
| Monitoring | Database Insights Standard; Enhanced Monitoring every 60 seconds. |
| Backup | Automated backup with a seven-day retention period. |

#### Configuration Decisions

1. Select the MySQL engine and Production template.
2. Set the DB identifier to `sportbooking-mysql`.
3. Select `sportbooking-vpc` and the private DB subnet group.
4. Select Multi-AZ DB instance deployment.
5. Disable public access and attach `sportbooking-rds`.
6. Create the initial `sport_booking` database.

Verify the following information on the **Connectivity & security** tab:

- Record the endpoint and port `3306` for `DB_URL` in Secrets Manager.
- Confirm that **Publicly accessible** is `No`.
- Confirm that the VPC Security Group is `sportbooking-rds` and accepts connections only from `sportbooking-app`.

Test connectivity from EC2 only after the compute layer is deployed in section 5.5. Schema and data-query results are recorded in section 5.7, so no procedure attempts to use EC2 before EC2 exists.

{{% notice warning %}}
RDS is configured with **Public access: No**. The MySQL port `3306` rule accepts traffic only from the `sportbooking-app` Security Group.
{{% /notice %}}

#### Procedure

The RDS procedure starts with the DB subnet group, continues with the private MySQL database, and finishes by verifying the endpoint and Security Group.

**Procedure:**

1. **Open the DB subnet group workflow**

   From Amazon RDS, open the database management area and navigate to **Subnet groups** to prepare a dedicated DB subnet group for the data tier.

   ![Start creating the DB subnet group](/images/5-Workshop/5.4-data-access/rds-01.png)

   <p class="image-caption"><em>Figure 1: Start creating the DB subnet group</em></p>
   RDS requires a DB subnet group to determine which private database subnets can host the database.


2. **Initialize the DB subnet group**

   In **Subnet groups**, choose **Create DB subnet group**. This resource must exist before the database so RDS can distribute the two instances across private database subnets in two Availability Zones.

   ![Open the Subnet groups list](/images/5-Workshop/5.4-data-access/rds-02.png)

   <p class="image-caption"><em>Figure 2: Open the Subnet groups list</em></p>
   Creating the subnet group first makes the prepared private database subnet set available during database creation.


3. **Enter DB subnet group details**

   Enter `sportbooking-db-subnet-group` as the name, add a description, and select the SportBooking VPC.

   ![Enter DB subnet group details](/images/5-Workshop/5.4-data-access/rds-03.png)

   <p class="image-caption"><em>Figure 3: Enter DB subnet group details</em></p>
   The subnet group must be in the same VPC as the EC2 application. Selecting another VPC prevents private network connectivity between the application and database.


4. **Select Availability Zones and private database subnets**

   Select the Availability Zones used by the database, select the private database subnets, and choose **Create**.

   ![Select Availability Zones and private database subnets](/images/5-Workshop/5.4-data-access/rds-04.png)

   <p class="image-caption"><em>Figure 4: Select Availability Zones and private database subnets</em></p>
   Two private database subnets in two AZs provide the network foundation for RDS Multi-AZ and isolate the database from the Internet.


5. **Start creating the MySQL database**

   Choose **Create database**, select **MySQL** as the engine, and select the **Production** template.

   ![Start creating the MySQL database](/images/5-Workshop/5.4-data-access/rds-05.png)

   <p class="image-caption"><em>Figure 5: Start creating the MySQL database</em></p>
   SportBooking uses MySQL, so Amazon RDS for MySQL provides a suitable managed database and avoids operating a database directly on EC2.


6. **Select the template and availability option**

   Select the **Production** template, then select **Multi-AZ DB instance deployment (2 instances)**.

   ![Select the template and availability option](/images/5-Workshop/5.4-data-access/rds-06.png)

   <p class="image-caption"><em>Figure 6: Select the template and availability option</em></p>
   This option creates a primary DB instance and an RDS-managed standby in another Availability Zone.


7. **Set the database identifier and credentials**

   Enter `sportbooking-mysql` as the DB identifier and select **Self managed** credentials. Enter `sportbooking_ad` as the master username; the password is concealed in the image.

   ![Set the database identifier and credentials](/images/5-Workshop/5.4-data-access/rds-07.png)

   <p class="image-caption"><em>Figure 7: Set the database identifier and credentials</em></p>
   The DB identifier and credentials are later used to define `DB_URL`, `DB_USERNAME`, and `DB_PASSWORD` in the application secret.


8. **Select the DB instance class**

   Select **Burstable classes**, then select `db.t3.small` with 2 vCPUs and 2 GiB of memory.

   ![Select the DB instance class](/images/5-Workshop/5.4-data-access/rds-08.png)

   <p class="image-caption"><em>Figure 8: Select the DB instance class</em></p>
   The `db.t3.small` configuration meets the functional testing scope and establishes a baseline for cost monitoring.


9. **Configure storage**

   Select **General Purpose SSD (gp3)** and set allocated storage to `20 GiB`.

   ![Configure RDS storage](/images/5-Workshop/5.4-data-access/rds-09.png)

   <p class="image-caption"><em>Figure 9: Configure storage</em></p>
   The `20 GiB` allocation supports the data scope of this deployment.


10. **Select the RDS network**

   Select **Don't connect to an EC2 compute resource** when configuring Security Groups manually. Select the SportBooking VPC and the private DB subnet group created earlier.

   ![Select the RDS network](/images/5-Workshop/5.4-data-access/rds-10.png)

   <p class="image-caption"><em>Figure 10: Select the RDS network</em></p>
   Selecting the VPC and subnet group manually ensures that the database remains in private database subnets and prevents unintended security rules from being added.


11. **Disable public access and select the Security Group**

   Select **Public access: No** and select or create the RDS Security Group. Confirm that it allows MySQL `3306` only from the EC2 App Security Group.

   ![Disable public access and select the Security Group](/images/5-Workshop/5.4-data-access/rds-11.png)

   <p class="image-caption"><em>Figure 11: Disable public access and select the Security Group</em></p>
   The database has no public access path; only EC2 instances associated with `sportbooking-app` can connect to MySQL.


12. **Configure monitoring**

   Select **Database Insights - Standard**, then enable **Enhanced Monitoring** with 60-second granularity.

   ![Configure RDS monitoring](/images/5-Workshop/5.4-data-access/rds-12.png)

   <p class="image-caption"><em>Figure 12: Configure monitoring</em></p>
   Monitoring provides operating-system and database metrics for CPU, connections, and storage.


13. **Set the database name and port**

   Open **Additional configuration**, enter `sport_booking` as the initial database name, and retain MySQL port `3306`.

   ![Set the database name and port](/images/5-Workshop/5.4-data-access/rds-13.png)

   <p class="image-caption"><em>Figure 13: Set the database name and port</em></p>
   The application requires the `sport_booking` database. If this field is empty or incorrect, the application may connect to the server but not find the expected schema.


14. **Configure backups**

   Enable automated backups and set the backup retention period to seven days.

   ![Configure RDS backups](/images/5-Workshop/5.4-data-access/rds-14.png)

   <p class="image-caption"><em>Figure 14: Configure backups</em></p>
   This setting allows RDS to retain automated backups for seven days for recovery.


15. **Review cost and create the database**

   Review the estimated monthly cost, public access setting, VPC, Security Group, and storage. Choose **Create database**.

   ![Review cost and create the database](/images/5-Workshop/5.4-data-access/rds-15.png)

   <p class="image-caption"><em>Figure 15: Review cost and create the database</em></p>
   The final review verifies the instance size, storage, and public-access status before the database is created.


16. **Monitor the database status**

   Return to **Databases** and wait until the status changes to **Available**.

   ![Monitor the database status](/images/5-Workshop/5.4-data-access/rds-16.png)

   <p class="image-caption"><em>Figure 16: Monitor the database status</em></p>
   The EC2 application can establish a stable connection only after the database becomes Available.


17. **Verify the endpoint and network**

   Open the database and inspect **Connectivity & security**. Record the endpoint, port, VPC, subnet group, and Security Group.

   ![Verify the RDS endpoint and network](/images/5-Workshop/5.4-data-access/rds-17.png)

   <p class="image-caption"><em>Figure 17: Verify the endpoint and network</em></p>
   Store the endpoint and port in Secrets Manager. The VPC and Security Group information provides evidence that the database is private and accessible only from the application layer.
