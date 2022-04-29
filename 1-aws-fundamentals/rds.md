# RDS: Relational Database Service

A managed DB service for DB use SQL a query

It allows you to create databases in the cloud that are
* Postgres
* Oracle
* MySQL
* MariaDB
* Microsoft SQL Server
* Aurora (AWS proprietary database)

Advantages of RDS over deploying a database in EC2
* Managed service
* OS patching level
* Continuous backups and restore to specific timestamps (Point in Time Restore)
* Monitoring dashboards
* Read replicas for improved read performance
* Multi AZ setup for DR (Disaster Recovery)
* Maintenance windows for upgrades
* Scaling capability (vertical and horizontal)
* But you can’t SSH into your instances (amazon manages them for you)

RDS Read replicas for read scalability
* Up to 5 read replicas
* Within AZ, Cross AZ or Cross region
* Replication is Async, so reads are eventually consistent
* Replicas can be promoted to their own DB
* Applications must update the connection string to leverage read replicas
* To enable read replicas, you need to enable backups

RDS Read Replicas – Use Cases
- You have a production database that is taking on normal load
- You want to run a reporting application to run some analytics
- You create a Read Replica to run the new workload there
- The production application is unaffected
- Read replicas are used for SELECT (=read) only kind of statements (not INSERT, UPDATE, DELETE)

RDS Read Replicas – Network Cost
- In AWS there’s a network cost when data goes from one AZ to another 
- To reduce the cost, you can have your Read Replicas in the same AZ

RDS Multi AZ (Disaster Recovery)
* SYNC replication
* One DNS name - automatic app failover to standby
* Increase availability
* Failover in case of loss of AZ, loss of network, instance or storage failure
* No manual intervention in apps
* Not used for scaling (only disaster recovery)

RDS Backups
* Backups are automatically enabled in RDS
* Automated backups:
    * Daily full snapshot of the database
    * Capture transaction logs in real time
    * Ability to restore to any point in time
    * 7 days retention (can be increased to 35 days)
* DB Snapshots:
    * Manually triggered by the user
    * Retention of backup for as long as you want

RDS Encryption
* Encryption at rest capability with AWS KMS - AES-256 encryption
* SSL certificates to encrypt data to RDS in flight
* To enforce SSL:
    * PostgreSQL: rds.force_ssl=1 in the AWS RDS Console (parameter groups)
* TO connect using SSL:
    * Provide the SSL Trust certificate (can be downloaded from AWS)
    * Provide SSL options when connection to the database
	
	
RDS Encryption Operations
- Encrypting RDS backups
	- Snapshots of un-encrypted RDS databases are un-encrypted 
	- Snapshots of encrypted RDS databases are encrypted
	- Can copy a snapshot into an encrypted one
- To encrypt an un-encrypted RDS database: 
	- Create a snapshot of the un-encrypted database
	- Copy the snapshot and enable encryption for the snapshot
	- Restore the database from the encrypted snapshot
	- Migrate applications to the new database, and delete the old database

RDS Security
* RDS databases are usually deployed within a private subnet, not in a public one
* RDS Security works by leveraging security groups (the same concept as for EC2 instances) - it controls who can communicate with RDS
* IAM policies help control who can manage RDS
* Traditional username and password can be used to login to the database
* IAM users can now be used too (for MySQL / Aurora - New)

RDS vs. Aurora
* Aurora is a proprietary technology from AWS (not open sourced)
* Postgres and MySQL are both supported as Aurora DB (that means you r drivers will work as if Aurora was a Postgres or MySQL database)
* Aurora is “AWS cloud optimized” and claims 5x performance improvements over MySQL on RDS, over 3x the performance of Postgres on RDS
* Aurora storage automatically grows in increments of 10GB, up to 64 TB
* Aurora can have 15 replicas while MySQL has 5, and the replication process is faster (sub 10 ms replica lag)
* Failover in Aurora is instantaneous. It’s HA native.
* Aurora costs more than RDS (20% more) - but is more efficient
- Aurora support for cross region replication

Aurora DB Cluster
- writer endpoint : pointing to the master
- reader endpoint: connection load balancing

Aurora Security
- Similar to RDS because uses the same engines
- Encryption at rest using KMS
- Automated backups, snapshots and replicas are also encrypted
- Encryption in flight using SSL (same process as MySQL or Postgres) 
- Possibility to authenticate using IAM token (same method as RDS) 
- You are responsible for protecting the instance with security groups 
- You can’t SSH

 Aurora Serverless
- Automated database instantiation and auto- scaling based on actual usage
- Good for infrequent, intermittent or unpredictable workloads
- No capacity planning needed
- Pay per second, can be more cost-effective 

Global Aurora
- Aurora Cross Region Read Replicas: 
	- Useful for disaster recovery
	- Simple to put in place
- Aurora Global Database (recommended):
	- 1 Primary Region (read / write)
	- Up to 5 secondary (read-only) regions, replication lag is less than 1 second
	- Up to 16 Read Replicas per secondary region
	- Helps for decreasing latency
	• Promoting another region (for disaster recovery) has an RTO of < 1 minute
	