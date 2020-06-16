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
	
Amazon ElastiCache Overview
- The same way RDS is to get managed Relational Databases...
- ElastiCache is to get managed Redis or Memcached
- Caches are in-memory databases with really high performance, low latency
- Helps reduce load off of databases for read intensive workloads
- Helps make your application stateless
- AWS takes care of OS maintenance / patching, optimizations, setup, configuration, monitoring, failure recovery and backups
- Using ElastiCache involves heavy application code changes

ElastiCache Solution Architecture - DB Cache
- Applications queries ElastiCache, if not available, get from RDS and store in ElastiCache.
- Helps relieve load in RDS.
- Cache must have an invalidation strategy to make sure only the most current data is used in there.

ElastiCache Solution Architecture – User Session Store
- User logs into any of the application
- The application writes the session data into ElastiCache
- The user hits another instance of our application
- The instance retrieves the data and the user is already logged in

ElastiCache – Redis vs Memcached
- REDIS
	- Multi AZ with Auto-Failover
	- Read Replicas to scale reads and have high availability
	- Data Durability using AOF persistence
	- Backup and restore features

- MEMCACHED
	- Multi-node for partitioning of data (sharding)
	- Non persistent
	- No backup and restore
	- Multi-threaded architecture
	
Caching Implementation
- Lazy Loading / Cache-Aside / Lazy Population
	- Pros
		- Only requested data is cached (the cache isn’t filled up with unused data)
		- Node failures are not fatal (just increased latency to warm the cache)
	- Cons
		- Cache miss penalty that results in 3 round trips, noticeable delay for that request
		- Stale data: data can be updated in the database and outdated in the cache
		
Write Through – Add or Update cache when database is updated
	- Pros:
		- Data in cache is never stale, reads are quick
		- Write penalty vs Read penalty (each write requires 2 calls)
	- Cons:
		- Missing Data until it is added / updated in the DB. Mitigation is to implement Lazy Loading strategy as well
		- Cache churn – a lot of the data will never be read
		
Cache Evictions and Time-to-live (TTL)
- Cache eviction can occur in three ways: 
	- You delete the item explicitly in the cache
	- Item is evicted because the memory is full and it’s not recently used (LRU) 
	- You set an item time-to-live(orTTL)
- TTL are helpful for any kind of data: 
	- Leaderboards
	- Comments
	- Activity streams
- TTL can range from few seconds to hours or days
- If too many evictions happen due to memory, you should scale up or out
