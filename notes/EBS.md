# 💾 AWS EBS, Snapshots, EFS & Storage Concepts

## 🧱 EBS Volume (Elastic Block Store)

An **EBS Volume** is a **network storage drive** that can be **attached to an EC2 instance**.

### ⚙️ Key Characteristics
- Persists data even after **instance termination**
- **Bound to a specific Availability Zone (AZ)**
- **Can only be attached to one instance** at a time  
  *(Multi-Attach supported for io1/io2 — up to 16 instances)*
- **Billed for the full capacity**, whether used or not  
- Capacity can be **increased anytime** (cannot decrease)
- **Root EBS volume** deletes on instance termination by default  
  (additional volumes do **not** delete by default — configurable)

### 🌍 Moving Volumes
To move an EBS volume across AZs or regions:
> Create a **snapshot**, then (copy and) **restore** it in another AZ/region.

---

## 📸 EBS Snapshots

EBS Snapshots are **incremental backups** of your EBS volumes.

### 🔧 Features
- Can be created anytime (best to **detach** volume before snapshot)
- Can be **copied** across **AZs or regions**

| Feature | Description | Cost / Speed |
|----------|--------------|--------------|
| 🧊 **Snapshot Archive** | Moves snapshot to cheaper storage | 75% cheaper, 24–72h to restore |
| 🗑️ **Recycle Bin** | Retains deleted snapshots for 1 day–1 year | You pay like you use them. |
| ⚡ **Fast Snapshot Restore (FSR)** | Instantly restores snapshots to volumes | Very fast, very expensive |

---

## 🖼️ Amazon Machine Image (AMI)

An **AMI** is a **preconfigured image** used to launch EC2 instances with custom software and settings.

With AMIs we can create our own software, configurations, OS, monitoring, etc, and have **way faster** boot with our software pre-packed.

### 📦 Types of AMIs
- 🟢 **Public AMIs** (provided by AWS)
- 🧑‍💻 **Custom AMIs** (created by you)
- 🛒 **Marketplace AMIs** (community or vendor provided — may be paid)

### 💡 Notes
- AMIs are **built per region**, but can be **copied** to others.
- **Creating an AMI** generates a **snapshot**, which is why it launches quickly.

---

## ⚙️ EC2 Instance Store

A **high-performance hardware-based** disk directly attached to the EC2 host.

### ⚡ Characteristics
- **Extremely fast I/O performance** (could be millions..)
- **Data lost** when instance stops!
- Ideal for **cache, buffer, temporary data, or scratch space**
- Always **backup or replicate** critical data

---

## 📊 EBS Volume Types

| Type | Description | Use Cases | Max Size / IOPS |
|------|--------------|------------|-----------------|
| 💎 **gp2/gp3** | General Purpose SSD, balanced performance and cost | Dev/test, low-latency apps | Up to 16 TB / 16.000 IOPS |
| ⚙️ **io1/io2** | High-performance SSD (Block Express) | Databases, critical workloads, multi-attach support | io1 → 64.000 IOPS & 16 TBs / io2 → 256.000 IOPS and 64 TBs  |
| 📀 **st1** | HDD, low-cost, throughput optimized | Big data, log processing, data warehouses | Up to 16 TB / 500 IOPS |
| 🧊 **sc1** | Cold HDD, lowest cost | Archival, infrequent access | Up to 16 TB / 250 IOPS |

> Only **gp2/gp3/io1/io2** volumes can be **boot volumes**.  
> In gp3 we can set IOPS and throughput, in gp2 they are linked together.  
> io2 can have sub-milisecond latency. Extremly fast.

### 🔄 EBS Multi-Attach (io1/io2 only)
- Attach a single EBS volume to **up to 16 instances** in the same AZ  
- Instances have **full read/write permissions**
- Designed for **clustered or highly available applications**
- Requires a **cluster-aware file system** (not EXT4 or XFS)
- For applications that manage **concurrent write** operations.

---

## 🔐 EBS Encryption

- Uses **KMS (AES-256)** encryption  
- **Snapshots and derived volumes** are automatically encrypted  
- **Minimal impact on latency**  
- Encryption and decryption are handled transparently NOT from us.

### 🧭 To Encrypt an Unencrypted Volume
1. Create a **snapshot**
2. Copy the snapshot and enable **encryption**
3. Create a new **encrypted volume** from that snapshot

---

## 📁 Amazon EFS (Elastic File System)

A **managed network file system (NFS)** for **shared access** across multiple EC2 instances.

### 🌍 Key Features
- NFSv4.1 protocol
- Works across **multiple AZs** (e.g., `-1a`, `-1b`, `-1c`)
- Highly **available**, **scalable**, and **expensive** (~3× gp2)
- **Pay only for the storage used**
- Compatible **only with Linux-based AMIs**
- Uses **Security Groups** for access control
- Can be **encrypted with KMS**
- Fully **POSIX-compliant** (behaves like a Linux file system with API)
- Can scale automatically, no need to specify how much capacity we need.
- Use cases: Content management, web serving, data sharing, WordPress, etc.

> EFS allows mounting the same file system across multiple EC2 instances in **different AZs**.

### 📈 Scalability
- Supports **1000s of NFS clients** concurrently
- Automatically scales up to **petabytes**
- Up to **10+ GB/s** throughput

---

## ⚙️ Performance Modes (Set at EFS Creation)
| Mode | Description | Best For |
|------|--------------|----------|
| **General Purpose** | Low latency | Web servers, CMS |
| **Max I/O** | High latency, high throughput | Parallel data, Big data, media processing |

---

## ⚡ Throughput Modes
| Mode | Description |
|------|--------------|
| **Bursting** | 50 MiB/s per 1 TB + bursts up to 100 MiB/s |
| **Provisioned** | Fixed throughput regardless of storage size (e.g., 1 GiB/s for 1 TB) |
| **Elastic** | Auto-scales up and down based on demand (for unpredictable workloads) |

---

## 🪣 EFS Storage Classes & Lifecycle Management

| Tier | Description | Notes |
|------|--------------|-------|
| **Standard** | Frequently accessed files | Default storage |
| **Infrequent Access (EFS-IA)** | Lower storage cost, there is a retrieval fee |  |
| **Archive** | Rarely accessed data, ~50% cheaper |  |

🕒 **Lifecycle Policies** can automatically move files between storage tiers after *N* days.

---

## 🧱 Availability and Durability
| Type | Description | Use Case |
|------|--------------|----------|
| **Standard** | Multi-AZ redundancy | Production environments |
| **One Zone** | Single AZ, cheaper | Dev/test, backups enabled by default, compatible with IA |

> One Zone storage can offer **up to 90% cost savings**.

---

## 🔍 Comparison: EBS vs EFS

| Feature | EBS | EFS |
|----------|------|------|
| 🔗 Attachments | One instance (or 16 with multi-attach) | 100s of instances |
| 📦 Scope | AZ-specific | Multi-AZ |
| 💰 Cost | Lower | Higher (≈2× gp2) |

> ⚠️ During heavy traffic, avoid running **EBS snapshots/backups**, as they consume I/O and can degrade performance.  
> gp2 I/O increase if disk size increased.  
> gp3 and io1 can increase I/O independently.  
> EFS can leverage storage tiers for cost savings.  

---