# ElastiCache

Overview:
The same way RDS is to get managed Relational Databases, ElastiCache is to get managed Redis or Memcached. Caches are in-memory databases with really high performance, low latency. They help reduce loads off of databases for read intensive workloads. They help make your application stateless. 
* Write scaling using shading. 
* Read scaling using Read Replicas
* Multi AZ with Failover Capability
* AWS takes care of OS maintenance / patching, optimizations, setup, configuration, monitoring, failure recovery and backups

#### Solution Architecture - DB Cache
* Applications queries ElastiCache, if not available, get from RDS and store in ElastiCache
* Helps relieve load in RDS
* Cache must have an invalidation strategy to make sure only the most current data is used in there
User Session Store
* User logs into any of the applications
* The application writes the session data into ElastiCache
* The user hits another instance of our application
* The instance retrieves the data and the user is already logged in

#### Redis Overview
* Redis is an in-memory key-value store
* Super low latency (sub ms)
* Cache survive reboots by default (it’s called persistence)
* Great to host
    * User sessions
    * Leaderboards (for gaming)
    * Distributed states
    * Relieve pressure on databases (such as RDS)
    * Pub / Sub capability for messaging 
* Multi AZ with Automatic failover for Disaster Recovery if you don’t want to lose your cache data
* Support for Read Replicas

#### Memcached Overview
* Memcached is an in-memory object store
* Cache doesn’t survive reboots
* Use cases:
* Quick retrieval of objects from memory
* Cache often accessed objects
* Overall, Redis has largely grown in popularity and has better feature sets than memcached
* Most likely, you’d probably only want to use Redis for caching needs