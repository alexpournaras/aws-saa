# 🚀 AWS CloudFront

## 🌍 Overview  
AWS **CloudFront** is a **Content Delivery Network (CDN)** that:  
- ⚡ Improves read performance (content cached at edge locations)  
- 😀 Improves user experience due to faster loading  
- 🛡️ Provides DDoS protection thanks to global distribution + AWS Shield + WAF (Web Application Firewall)  

---

## 🏗️ CloudFront Origins

### 🪣 1. S3 Bucket  
Use cases:  
- 📁 Distributing and caching files at edge locations  
- ⬆️ Uploading to S3 **through CloudFront**  
- 🔒 Secured using **Origin Access Control (OAC)**  

### 🛰️ 2. VPC Origin  
Applications hosted in **VPC private subnets**, such as:  
- Application Load Balancer  
- Network Load Balancer  
- EC2 Instances  

No need to expose them publicly — CloudFront communicates internally.

### 🌐 3. Custom Origin (HTTP)  
Examples:  
- S3 static website hosting  
- Any **public HTTP backend** you want  

---

## 🔁 How CloudFront Caching Works  
1. A user requests content from an **edge location**.  
2. CloudFront checks if it’s cached.  
3. If **not cached**, CloudFront fetches it from the origin (e.g., S3) and caches it.  
4. Next user in the **same edge location** gets it instantly from cache.  

---

## 🔐 Enforcing S3 Access via CloudFront Only  
To make a private S3 website accessible **only** through CloudFront:  
1. Create a CloudFront distribution  
2. Create an **OAC**  
3. Update S3 bucket policy so it **only accepts requests from this CloudFront distribution**  

---

## 🌎 CloudFront vs S3 Cross-Region Replication (CRR)

### 🎯 CloudFront  
- 🌐 Uses **global edge network** (216 points of presence)  
- ⏱️ Files cached with TTL  
- 👍 Best for **static content** needed worldwide  

### 🔁 S3 Cross-Region Replication  
- ⚙️ Must be set up **per region**  
- 🔄 Files updated in near real-time  
- 📖 Read-only  
- 👍 Best for **dynamic content** needed in multiple nearby regions at low-latency  

---

## 🏗️ CloudFront With ALB, NLB or EC2 via VPC Origins  
Allows delivery of content from apps hosted **in private subnets** — no public exposure needed.

Traffic flow:  
Users → CloudFront → VPC Origin → EC2 in private subnet

Previously, we had to manually allow CloudFront IPs via security groups.

---

## 🌐 CloudFront Geo Restriction  
Control who can access your distribution.

- ✔️ **Allowlist** — only selected countries allowed  
- ❌ **Blocklist** — selected countries blocked/banned  

Determined using **client IP**.  
📌 Use case: ✓ Copyright control, ✓ Licensing restrictions  

---

## 💰 CloudFront Pricing  
Data-out (data sent) cost varies by edge location (US/EU cheaper; India/China more expensive).

### 💵 Price Classes  
- **Price Class All** → max performance, all locations  
- **Price Class 200** → most regions, excludes expensive ones  
- **Price Class 100** → cheapest regions (US + EU)  

---

## 🧽 CloudFront Cache Invalidation  
CloudFront **doesn’t know** if content changed before TTL expires.

To force refresh:  
- 🗑️ Invalidate **all files**: `/*`  
- 🖼️ Invalidate **specific paths**: `/images/*`  

Used to **bypass TTL**.

---

# ⚡ AWS Global Accelerator

## 🧠 Key Concepts  
- **Unicast IP** → one server = one IP  
- **Anycast IP** → many edge servers share the same IP, user routed to nearest  

Global Accelerator gives you **two Anycast IPs**.

### 🏎️ How It Works  
- Traffic hits the Anycast IP → goes to nearest **edge location**  
- Edge sends traffic internally to AWS (fast private backbone)  
- Works with:
  - Elastic IPs  
  - EC2  
  - ALB / NLB  
  - Public or Private apps  

### 🎯 Benefits  
- ⚡ Routing to lowest latency  
- 🌍 Fast regional failover  
- 🔐 DDoS protection (AWS Shield)  
- ❤️ Client affinity (sticky routing - same client will be routed to same backend)  
- 📌 Static IPs (Only 2 external IPs need to be whitelisted)  
- ❤️ Health checks

---

## ⚔️ Global Accelerator vs CloudFront

## 🌀 CloudFront  
- Best for **cacheable content** (images, videos)  
- Also useful for **dynamic site delivery** and API acceleration  
- 📍 **Content is served at the edge location**!!!  

## 🚦 Global Accelerator  
- Accelerates **ANY** TCP/UDP application  
- 📍 **Traffic is proxied at the edge location** and **forwarded to AWS services internally/privately** !!! 
- Ideal for:
  - 🕹️ Non-HTTP workloads (gaming, IoT, VoIP)  
  - 🌐 HTTP workloads requiring **static IP addresses**  
  - ⚠️ Fast, deterministic regional failover  

---

## 🎉 Summary  
- **CloudFront** = Caches content at the edge location → reduces need to reach our application  
- **Global Accelerator** = Uses edge network as a **fast entry point** → routes to our application in AWS privately.  