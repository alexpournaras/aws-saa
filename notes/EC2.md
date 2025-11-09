# ⚙️ Amazon EC2 (Elastic Compute Cloud)

## ☁️ Overview
**Amazon EC2 (Elastic Compute Cloud)** is an **Infrastructure as a Service (IaaS)** offering that provides scalable virtual servers in the cloud.

### 💡 Key Features
- 💻 Rent virtual machines (**EC2 Instances**)
- 💾 Store data on virtual drives (**EBS**)
- ⚖️ Distribute traffic across servers (**ELB**)
- 🚀 Scale automatically with **Auto Scaling Groups (ASG)**

---

## 🧩 Configuration Options
When launching an EC2 instance, you can configure:

- 🧠 **Operating System:** Linux, Windows, or macOS  
- ⚙️ **Compute Resources:** CPU and RAM  
- 💽 **Storage:**
  - Network-attached: **EBS** (Elastic Block Store) & **EFS** (Elastic File System)
  - Local (hardware-based): **EC2 Instance Store**
- 🌐 **Networking:** Speed and Public IP Address  
- 🔒 **Firewall Rules:** Managed through **Security Groups**
- 🧰 **Bootstrap Configuration:** Use **EC2 User Data** to run setup scripts at first boot

### 📝 EC2 User Data
Bootstrap your instance at startup by providing a script that:
- Installs updates and packages
- Downloads dependencies or configurations
- Automates initial setup tasks

---

## 📦 Key Concepts

### 🖼️ Amazon Machine Image (AMI)
A preconfigured **OS image** used to launch EC2 instances.

### 🔑 Key Pair
Used for **SSH authentication** (typically `.pem` files with RSA keys).  
Example:
```bash
chmod 400 key.pem
ssh -i key.pem ec2-user@3.250.26.200
```

---

## 🛑 Stopping & Restarting Instances
- When an instance is **stopped**, you **don’t pay** for compute time.
- Upon **restart**, the **public IP address may change** ⚠️.

---

## 🧮 Instance Types

| Category | Description | Use Cases | Example |
|-----------|--------------|-----------|----------|
| 🧩 **General Purpose** | Balanced compute, memory, and networking | Web servers, development environments | `t2.micro` |
| ⚡ **Compute Optimized** | High CPU performance | Batch processing, media transcoding, High Performance Computing (HPC), Scientific modeling, machine learning, gaming servers | Start with `C` => `c5.large` |
| 🧠 **Memory Optimized** | Large in-memory datasets | Databases, caching, analytics, Business Intelligence, in-memory databases | Start with `R` => `r5.large` |
| 💾 **Storage Optimized** | High read/write throughput from storage | Online Transaction Processing (OLTP), Relational & NoSQL databases, cache (Redis), data warehousing, distributed file systems | Start with `I or D or H1` => `i3.large` |
| 🧮 **Accelerated Computing** | GPU or FPGA-based | Machine learning, HPC | `p3.2xlarge` |

---

## 🔍 Instance Naming Convention
Example: **m5.2xlarge**
- `m` → Instance class  
- `5` → Generation/version  
- `2xlarge` → Instance size for that class

---

## 🔐 Security Groups
Act as **virtual firewalls** controlling inbound and outbound traffic.

- Contain **only allow rules**
- Can reference **IP addresses** or **other security groups**
- Are tied to a specific **region/VPC**
- Operate **outside the EC2 instance**

### 🧱 Rules
- All **inbound** traffic is **blocked** by default
- All **outbound** traffic is **allowed**

### 🧾 Common Ports
| Port | Protocol | Description |
|------|-----------|-------------|
| 22 | SSH | Secure Shell (Linux) |
| 21 | FTP | File Transfer Protocol |
| 22 | SFTP | Secure File Transfer (via SSH) |
| 80 | HTTP | Web traffic |
| 443 | HTTPS | Secure web traffic |
| 3389 | RDP | Remote Desktop (Windows) |

⚠️ **Timeout** → likely a Security Group issue  
⚠️ **Connection refused** → likely an application-level issue

---

## 🌐 EC2 Instance Connect
Browser-based SSH access directly from the AWS Console (requires proper SSH rule in the Security Group).

---

## 🚫 Best Practices
- **Never configure AWS credentials directly** inside an EC2 instance.  
  Instead, assign permissions through **IAM roles**:
  - `Instance → Actions → Security → Modify IAM Role`
  - Example command after giving permissions:
    ```bash
    aws iam list-users
    ```

---

## 💸 EC2 Pricing Models

| Type | Description | Best For | Notes |
|------|--------------|----------|-------|
| 🕐 **On-Demand** | Pay per second (after 1st minute) | Short, predictable workloads | Highest cost, uninterrupted |
| 🗓️ **Reserved Instances (1 or 3 years)** | Prepaid for specific instance type, region, and OS | Long-term workloads | Buy/sell in marketplace |
| 🔄 **Convertible Reserved** | Flexibility to change instance type, OS, or scope | Long workloads needing flexibility | 1 or 3 years |
| 💰 **Savings Plans (1 or 3 years)** | Commit to usage ($/hour) | Consistent workloads | Beyond commitment => On-demand pricing, locked to region and specific family type |
| ⚡ **Spot Instances** | Up to 90% cheaper | Fault-tolerant, flexible workloads | Can be interrupted anytime |
| 🧱 **Dedicated Hosts** | Entire physical server | Compliance or license restrictions | Most expensive, Bring Your Own License (BYOL) |
| 🔒 **Dedicated Instances** | Hardware not shared with other customers | Security or compliance |  |
| 📍 **Capacity Reservations** | Reserve capacity in a specific AZ | On-demand flexibility | Pay for reserved capacity and use whenever we want. Example: we use auto-scaling and we know there is always a need for at least 2 instances running all the time. |

---

### 🌀 Spot Instances in Detail
- Ideal for **batch jobs**, **data analysis**, **image processing**, or **distributed workloads**
- Not suitable for **critical jobs** or **databases**
- If the **spot price** exceeds your bid, you get a **2-minute warning** to stop or terminate the instance.⏳
- Create them with **Spot Requests**:
  - One-time: When acquire instance: ends the request.
  - Persistent: When instance is interrupted, the Spot Request will be recreated.
  - Cancel of Spot Requests will **NOT** stop instances. May fall in a **loop**!
  - The Role should have the `AmazonEC2SpotFleetTaggingRole` service role and allow the `spotfleet.amazonaws.com` service assume the role.
- **Spot Blocks:** Reserve for 1–6 hours (guaranteed availability)

### 🧠 Spot Fleet
* Sets of Spot Instances + (optional) On-demand Instances.
* Used to automatically request Spot Instances with the lowest price.

#### Strategies:
- **LowestPrice** → Cheapest pool, cost-optimized, short workloads  
- **Diversified** → Spread across pools, high availability, long workloads  
- **CapacityOptimized** → Best capacity pool with highest capacity available  
- **PriceCapacityOptimized** → Pools with highest capacity available -> then select the pool with the lowest price.

---

## 🌐 EC2 Networking, Elastic IP

#### 🧠 IPv6 and IoT
IPv6 was designed to **solve address limitations** and support the **Internet of Things (IoT)** by providing a vastly larger address space.

---

### 🌍 Public vs 🏠 Private IPs

#### 🌍 Public IP
- **Unique across the entire Internet**
- Can be **geolocated**

#### 🏠 Private IP
- **Unique only within a private network**
- Different private networks can reuse the **same IP ranges**
- Connect to the Internet through a **gateway or proxy**

> 💡 When an EC2 instance restarts, its **public IP may change**.

---

## 🔁 Elastic IP
An **Elastic IP (EIP)** solves the issue of changing public IPs.

### ✅ Key Points:
- Remains **persistent** even after EC2 restarts  
- Can be **re-associated** with another instance easily  
- Tied to a **private IP** within your VPC  
- Useful for maintaining a **fixed endpoint** (e.g., web server)

Example use case:
> You can detach the Elastic IP from one EC2 instance and attach it to another, keeping the same public address.

---

## 🏗️ Placement Groups
Define how instances are **physically placed** within the AWS infrastructure.  
They help you control **latency**, **fault tolerance**, and **throughput**.

### ⚡ Types of Placement Strategies

| Type | Description | Best Use Cases |
|------|--------------|----------------|
| **Cluster** | Groups instances close together within a single AZ for low latency and high network speed. Enhanced Networking | Big data processing, high-performance computing, or applications needing very fast interconnects. |
| **Spread** | Spreads instances across different hardware to reduce failure risk (max 7 instances per group per AZ). | Critical workloads needing high availability and fault tolerance. |
| **Partition** | Distributes instances across multiple partitions (different racks/machines) within an AZ. Each partition is isolated from others. Up to 100 EC2 per group, up to 7 particions per AZ | Large-scale systems like Cassandra or Apache Kafka that need redundancy. |

> 🧩 **AZ failure in a Cluster Group** means all instances in that group fail too.  
> Can get partition information from Metadata service.

---

## 🧩 Elastic Network Interfaces (ENI)

An **Elastic Network Interface (ENI)** represents a **virtual network card** (like `eth0`, `eth1`, etc.) for your EC2 instances.

### ⚙️ ENI Components
- At least **one private IPv4 address**
- Optional **Elastic IP (EIP)** or **public IPv4**
- One or more **Security Groups**
- A **MAC address**
- Bound to a **specific Availability Zone**

### 🔄 Flexibility
- You can **attach or detach** ENIs between instances.

### 🧠 Use Case Example
> If an instance fails, launch a new one and attach the same ENI — traffic continues smoothly without reconfiguration.  
> Same for MAC-based licensing

---

## 💤 EC2 Hibernate

### 🧩 What It Does
EC2 Hibernate **preserves the in-memory (RAM) state** of the instance when it stops.

When the instance starts again:
- It **resumes quickly**, as the memory state is restored.
- The **RAM contents** are saved to a file on the **root EBS volume**.

### ⚙️ Requirements
- The **root EBS volume must be encrypted** 🔐

### 🕒 Limitations
- Hibernate cannot last more than **60 days**

### 🚀 Benefits
- Faster boot time than stopping/starting instances
- Useful for **long-running processes** or **services that take time to initialize**

---