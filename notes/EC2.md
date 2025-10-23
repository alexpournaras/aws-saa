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
| 📍 **Capacity Reservations** | Reserve capacity in a specific AZ | On-demand flexibility | Pay for reserved capacity and use whenever we want |

---

### 🌀 Spot Instances in Detail
- Ideal for **batch jobs**, **data analysis**, **image processing**, or **distributed workloads**
- Not suitable for **critical jobs** or **databases**
- If the **spot price** exceeds your bid, you get a **2-minute warning** to stop or terminate the instance.⏳
- Create them with **Spot Requests**:
  - One-time: When acquire instance: ends the request.
  - Persistent: When instance is interrupted, the Spot Request will be recreated.
  - Cancel of Spot Requests will **NOT** stop instances. May fall in a **loop**!
- **Spot Blocks:** Reserve for 1–6 hours (guaranteed availability)

### 🧠 Spot Fleet
* Sets of Spot Instances + (optional) On-demand Instances.
* Used to automatically request Spot Instances with the lowest price.

#### Strategies:
- **LowestPrice** → Cheapest pool, cost-optimized, short workloads  
- **Diversified** → Spread across pools, high availability, long workloads  
- **CapacityOptimized** → Best capacity pool with highest capacity available  
- **PriceCapacityOptimized** → Pools with highest capacity available -> then select the pool with the lowest price.


