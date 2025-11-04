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