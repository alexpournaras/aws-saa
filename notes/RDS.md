# 🗄️ Amazon RDS, Aurora & ElastiCache 

## 🧠 Amazon RDS (Relational Database Service)
- PostgreSQL
- MySQL
- MariaDB
- Oracle
- Microsoft SQL Server
- IBM DB2
- **Aurora (AWS-built)**

**Benefits:**
- 🔧 Automated provisioning & OS patching
- 💾 Continuous backups + **Point-in-time restore**
- 🧭 Monitoring
- 📚 **Read Replicas** for read scalability
- 🧱 **Multi-AZ** for disaster recovery / HA
- 🛠️ Maintenance windows for upgrades
- 📈 Vertical & horizontal scaling options
- 🗄️ Storage backed by **EBS**
- 🔒 **No SSH** access into RDS instances (managed service)

---

## 📦 RDS Storage Auto Scaling
- Automatically **increases storage** when space runs low
- You set a **maximum storage threshold** (GB)
- Auto-modification occurs when:
  - Free storage < **10%** of allocated
  - Low storage persists **≥ 5 minutes**
  - **≥ 6 hours** since last modification
- ✅ Great for **unpredictable workloads**

---

## 📚 RDS Read Replicas (Read Scalability)
- Up to **15** read replicas
- **Within AZ**, **cross-AZ**, or **cross-Region**
- Replication is **asynchronous**
- Replicas can be **promoted** to standalone DBs
- Apps must **change connection strings** to read from replicas
- Inside same region: **replication is free** (cross-region incurs extra cost)

**Typical use:** analytics/reporting on replicas → production remains unaffected  
(Read replicas are **read-only**: use `SELECT` only.)

---

## 🛡️ RDS Multi-AZ (High Availability, not Scaling)
- A **standby** database in a different AZ ready to be promoted in a failover.
- App uses **one connection string**; failover is transparent
- **Synchronous** replication (writes committed to both DBs in both AZs and need to be accepted by both)
- Purpose: **availability**, not for scaling

> 🔁 **Read Replicas + Multi-AZ:**  
> You can enable **Multi-AZ** on each read replica for their own disaster recovery.
> Example: 1 primary + 5 replicas; **each replica** can have a standby (Multi-AZ). This is for **resilience**, not read capacity.

**Convert Single-AZ → Multi-AZ (zero downtime):**
- Behind the scenes: RDS takes a snapshot, creates a standby from it, then establishes synchronous replication.

---

## 🧰 RDS Custom (Oracle & Microsoft SQL Server only)
- Classic RDS automations **plus**:
  - 🖥️ **OS access** (SSH / SSM)
  - Configure settings, install patches, enable features
- ⚠️ Disable **automation mode** before customizing
- 💡 Take a **snapshot first**
- You get **full admin** control (OS + DB)

---

# ⚡ Amazon Aurora

AWS-built, distributed relational database engine.

**Performance:**
- Aurora MySQL: **up to 5×** MySQL
- Aurora PostgreSQL: **up to 3×** PostgreSQL

**Storage & durability:**
- Auto-expands in **10 GB** increments, up to **256 TB**
- **6 copies** across **3 AZs**
  - Writes require **4/6** quorum
  - Reads require **3/6**
- Self-healing storage, fast failover (**< 30s**)
- Up to **15** replicas; cross-region replication supported
- Typically **~20% more expensive** than standard RDS (more efficient at scale)

## 🔗 Aurora Cluster Endpoints
- **Writer endpoint** → routes to the **primary (writer)**
- **Reader endpoint** → load balances across **all read replicas**
- **Auto Scaling for readers** → new replicas join the reader endpoint automatically (extended endpoint)
- **Shared storage volume** across the cluster (auto-expands)
- **Custom endpoints** → group selected read replicas (e.g., heavy analytics vs. general reads)
- For example we can have bigger instances like t2.large for our readers that will process analytics, and smaller instances for regular queries.

## 🔁 Aurora Features
- Backups, security, **zero-downtime patching**
- Monitoring & routine maintenance
- **Backtrack:** point-in-time restore **without** using backups

## 🌙 Aurora Serverless
- **Automatic** provisioning & scaling by actual usage
- **No capacity planning**, **pay-per-second**
- Ideal for **infrequent / unpredictable** workloads
- Uses a **Proxy Fleet** to manage connections to underlying DBs

## 🌍 Aurora Global Database
- 1 **primary** Region (read/write)
- Up to **10** secondary Regions (**read-only**; typical cross-region replication lag **< 1s**)
- Up to **16** read replicas **per secondary Region**
- **Promote** a secondary Region in **< 1 minute**

## 🤖 Aurora Machine Learning
- Invoke ML **directly from SQL**
- Integrations: **Amazon SageMaker** (any model), **Amazon Comprehend** (sentiment analysis)
- Use cases: fraud detection, ad targeting, product recommendations, sentiment analysis, etc.
- Flow: query data → send to ML service → return predictions into SQL result
- No need for machine learning experience 

## 🐟 Babelfish for Aurora PostgreSQL
- Run **Microsoft SQL Server apps on Aurora PostgreSQL** using **T-SQL** (driver & language)
- **Minimal code changes**
- Works well with migrations (AWS **SCT** & **DMS**)

---

## 💾 Backups & Restore (RDS & Aurora)
- **Automated backups** daily (during backup window)
- **Transaction logs** backed up every **5 minutes** → precise point-in-time restore
- Retention: **0–35 days** (Aurora: **1–35**, cannot be 0)
- **Manual snapshots** can be kept indefinitely

**Cost tip:** A **stopped RDS** instance still incurs **storage** costs.  
For long stops → **snapshot**, then **restore when needed**.

**Restore** always creates a **new DB**.

**Restoring from external backups:**
- **RDS engines:** upload backup to **S3**, restore into RDS
- **Aurora:** create backups with **Percona XtraBackup**, upload to **S3**, then restore to create a new Aurora DB

---

## 🧬 Aurora Database Cloning
- **Super Faster** than snapshot + restore
- Uses **Copy-on-Write**:
  - Initially shares storage with the source
  - Then more storage is allocated and data is copied to be seperated
- Great for **staging/testing** with minimal cost & no impact on production databases.

---

## 🔐 Encryption & Security
- If the **primary** is **unencrypted**, replicas **cannot** be encrypted
- If the **primary** is **encrypted**, replicas **must** be encrypted
- To encrypt an existing unencrypted DB:
  1. Create **snapshot**
  2. **Copy** snapshot with **encryption**
  3. **Restore** encrypted snapshot to a new DB
- **In-flight** encryption via **TLS** by default
- **IAM Authentication** supported (can also connect without username/password)
- **Security Groups** control network access
- Move **audit logs** to **CloudWatch** to keep them for longer periods of time.

---

# 🔗 RDS Proxy
- **Pools & shares** database connections
- Reduces DB stress (minimize open connections, can handle overloads of requests)
- **Serverless**, **auto-scaling**, **Multi-AZ** (high availability)
- Can reduce RDS/Aurora **failover time by ~66%**
- Supports: **MySQL**, **PostgreSQL**, **MariaDB**, **SQL Server**, **Aurora**
- Can enforce **IAM auth**; credentials in **AWS Secrets Manager**
- **Accessible only inside VPC** (not public)
- Extremely useful for **Lambda** & spiky workloads
- Often **no app code changes** required (just point to the proxy endpoint)

---

# ⚡ Amazon ElastiCache (Redis & Memcached)

- **VPC-only** (not Internet accessible)
- **In-memory** data store → **very low latency** / high performance
- Offload reads from databases; enable **stateless** app patterns (session storage)
- AWS manages OS, patches, setup, configuration, monitoring, recovery, backups
- ⚠️ Usually requires heavy **application code changes**

**Access pattern basics:**
- **Cache hit** → value found in cache
- **Cache miss** → Not found in cache.
- ⚠️ Ensure **correct cache invalidation** to avoid stale data

**Common example**: 
- Attempt to find in cache → Cache hit → all good
- Attempt to find in cache → Cache miss → fetch from DB (e.g., RDS) → **write to cache**

**Session example:** With a load balancer, a user may hit different instances.  
* Store sessions in ElastiCache → sessions are available everywhere.
* ELB with sticky sessions could solve the problem but what if an instance fail? That's why ElastiCache is a better option for that.

## 🔥 Redis
- **Multi-AZ** with **auto-failover**
- **Read replicas** for HA & scale
- Durability: **AOF** persistence; **backup & restore**
- Data structures: **sets**, **sorted sets** (great for leaderboards)
- Replication: replicas contain **full data**
- Security:
  - IAM (API-level)
  - **Redis AUTH** (username/password)
  - **Security Groups**
  - **SSL** for in-flight encryption

## ⚡ Memcached
- **Sharding/partitioning** across nodes (each node holds a **portion** of the cache/data)
- **No replication**, **no persistence**
- **No HA** built-in (except in serverless flavor with limited backup/restore)
- **Multithreaded** → extremely high throughput
- Auth: **SASL** (advanced)

---

## 🧠 ElastiCache Patterns
- **Lazy Loading:** All the read data is cached; simple, but cached data can become **stale**
- **Write-Through:** Add or update data in cache when written to DB (no state data)
- **Session Store:** store temporary data in cache with **TTL** expiration

**Redis Sorted Sets Example:**  
Real-time **leaderboards** with automatic ranking & ordering of new imported elements in live-time. — no custom code required.

---

## RDS Databases ports:
* PostgreSQL: 5432
* MySQL: 3306
* MariaDB: 3306 (same as MySQL)
* Oracle RDS: 1521
* MSSQL Server: 1433
