amazon rds
relational database service

* postgres
* mysql
* mariadb
* oracle
* microsoft sql server
* ibm db2
* aurora (from aws)

benefits:
* automated provisioning, os patching
* continues backup, restore to specific timestamp (point in time restore)
* monitoring
* read replicas for improved read performance
* multi az for disaster recovery
* maintenance periods for upgrades
* scaling vertical and horizontal
* storage from EBS

we cannot ssh in our ec2 instances

--

rds storage auto scaling
* automatically increase storage of database when rds detect we run out of space
* we need to set maximum storage thresshold (max limit in gb)
* automatically modify storage if:
    - free storage is less than 10% of allocated storage
    - low storage lasts at least 5 minutes
    - 6 hours have passed since last modification
* usefull for unpredictable workloads!!

--

rds read replicas for read scalability

* up to 15 read replicas
* within AZ, cross AZ, cross region
* replication is ASYNCRHONOUS
* replicas can be promoted to their own db
* applications need to change the connection string to the one of replicas to read them

perfect for running analytics for example.
we create a replica and run the processes in the replica
we can only use SELECT statements since its only with read permissions
production database is unaffected

its free for replication inside the same region. for cross regions replication, there are extra costs.

--

multi AZ

* there is a database in a different AZ that in case of master failover, it gets promoted to master
* the application has only one connection string
* SYNCRONOUS replication = when performed an operation in master, then it MUST happen to the stand-by (multi az) database to be accepted.
* increased availability 
* not used for scaling

The read replicas can be set us multi az for disaster recovery!!!!! (important)

so having multi az on read replicas what does that mean? we have a master, we also have 5 replicas of this master, and for each replica we have another stand-by db with multi az enabled. ?

from single az to multi az:
zero downtime, we just modify
behind the scenes: snapshot from master, create new db from snapshot, establish sync oeprations from master to new db

--

rds custom

only for oracle and microsoft sql server databases
* has all automations from classic rds
* we get access to os, we can ssh to instance (ssh or ssm session manager) and can configure settings, install patches, enable features
* deactive automation mode before customization
* better to create snapshot before customize
* has full admin access to the OS and db, -> in classic rds, everything is managed from aws

--


aurora

its from aws (not open sourced)
with drivers its like mysql(x5 better) or postgresql (x3 better)
storage automatically increaments with 10 GBs, up to 256 TB
up to 15 replications, but faster
failovers are also faster because its high availability natively
its more efficient but cost 20% more than rds
6 copies across 3 az
 - 4 copies out of 6 needed to write
 - 3 copies out of 6 needed to read
has self healing
one master
failover of master can be replaced in less than 30 seconds
cross region replication

in a cluster:
there is the writer endpoint which talks with the master
then there is the reader endpoint which reads from all the read replicas
its like a load balancer among the read replicas
we can also have auto group scaling to have the number of replicas we want each time
all that, have a shared storage volume (automatically increased when needed)
with autoscaling, any new replica will extend reader endpoing
we can also create custom endpoints that will group specific replicas. for example we can have bigger instances for our heavy analytics queries, so we will perform them in the custom endpoint. the other queris we will create another custom endpoint that will replace the reader endpoint.

features:
backup, security, automated patch with 0 downtime
monitoring, routine maintenance, 
backtrack: restore data from any point of time without using backups

-- 

aurora serverless

automated instantiation and scaling based on actual usage
good for infrequent, unpredictable workloads
no capacity planning
can be cost-effective, pay per second

these are happenign because the client talks with a Proxy Fleet that handle all the talking of the multiple databases

-- 

aurora global

cross region read replicas
recommended:
* 1 primary region (read/write)
* up to 10 secondary regions (read-only, lag less than 1 second)
* up to 16 read replicas per secondary region
* promoting another region in less than 1 minute
* typical cross-region replication takes less than 1 second (important!!)

-- 

aurora machine learning

ml-based predictions from sql
supported services: Amazon SageMaker (with any model) and Amazon Comprehend (sentiment analyis)
no need for machine learnign experiance 
use cases: fraud detection, ads targeting, sentiment analysis, product recommendations

basically the sql data results will go to the ml services and will repond with the ml results

-- 

babelfish for aurora postgres

with that microsft sql server applications can work on aurora postgresql with T-SQL (driver, language)
no-to-little code changes
the same applications can be used after a migration of your database (AWS SCT and DMS)  


-- 

RDS and aurora Backups

automated backups daily during the backup window
transaction logs are backed up by rds every 5 minutes, thats how we restore whenever we want
keep backups from 0 to 35 days. 0 disables automated backups (aurora cannot disable them. its 1 to 35)
but we can keep data for as long as we want with manual snapshots

trick: in stopped rds database, you will pay for storage. if you plan to stop for a long time, you should snapshot and restore when needed.

restoring a backup or snapshot, it will create a new database
to restore backups in rds, we simply get the backup from our instances, upload to S3 and create new database from there.
to restore backups in aurora, we need to create the backups with percona xtrabackup, upload to s3 and then from there restore to create the new database

-- 

aurora database cloning 

faster than snapshot and restore
uses copy-on-write protocol:
* initially the new db uses same storage as the original,
* then more storage is allocated and data is copied to be seperated
its very fast and cost efficiant
usefull to create staging databases from production databases without affecting them

--

if master is not encrypted then read replicas can not be encrypted
if master is encrypted we of cousrse cannot have unencrypted replicas
to encrypt an unencrypted database, we snapshot and restore an encrypted
in flight encryption: they by default use TLS certificates (ssl)
we can use IAM roles to connect to the databases (isntead of username/password)
we can control network with security groups
we can keep audit logs more by sending them to cloudwatch

--

rds proxy

allow apps to pool and share db connections established with the database
improve database efficiency by reducing stress (minimize open connections, can handle overloads of requests)
serverless, autoscalling, multi az => high availability
reduced rds and aurora failovers by 66%
support: mysql postgresql, mariadb, microsoft sql server and aurora (both versions)
no code needed to implement
can enforce IAM authentication and keep credentials to AWS Secrets manager
rds proxy is not published, only accces from vpc
very usefull for lambda functions

--

amazon elastiCache 

we use elasticache to use redis and memcached
they are in-memory dbs super high performance and super low latency
they help reduce load from databases
can make our applications stateless by keepign the sessions for example
aws takes care of os maintance, patching, optimizations, setup , configuration, monitoring,failure recoverry, backups
* they involve heavy changes for code implementation

some usecases:
cache hit = found result in cache
cache miss = we didnt founf the result in cache
query elasticache, if not found, get data from rds, then store to elasticache
that helps reduce load from rds
cache must be correctly invalidate data, because thats a risk for mistaken data consistency!!!! important!!

user loggs in but we have a load balancer so each time can go to another instance. but with elasticache if we save the session, he can loggin from everywhere because we will retrieve session from cache

redis:
multi az with auto failover
read replicas with high avialability 
data durability with aof persistance
backup and restore features
support sets and sorted sets
* they replicate. all of the replications have the data

memcached:
* multi-node partioning (sharding) = means one node has someting, some other node have something else. so in the end the cache is shared but retrieve from the multiple nodes. no replication here.
no high avialbility
no persistent
backup and restore features only for serverlss version
multithreaded architecture thought!!

for redis we can use iam roles to authenticate
iam policies are only for aws api-level security
we can also have redis auth (username password) on top
we can have security groups
support ssl in flight encryption

memcached supports SASL-based authentication (advanced)


--

patterns for elasicache
lazy loading: all the read data is cached, data can become stale in cache
write through: add or update data in cache when written to db (no state data)
session store: store temporary session data in cache (they will expire with TTL features)

another good use case of redis are the sorted sets that offers element ordering
new elements are ranekd and ordered in real time
so for a gaming leaderboard for example, we dont need to do something in code, we can get the leadeboard from elasicache redis.