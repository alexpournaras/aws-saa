# 🌍 Amazon Route 53 — Complete Notes

**DNS (Domain Name System)** translates **human-friendly hostnames** into **IP addresses** machines understand.

Examples:
- `google.com` → `142.250.74.46`
- `example.org` → `93.184.216.34`

### DNS Components
| Level | Example | Description |
|------|---------|-------------|
| **TLD (Top-Level Domain)** | `.com`, `.gr`, `.org` | End of domain name |
| **Second-Level Domain** | `amazon.com`, `google.com` | Name under the TLD |
| **Subdomain** | `api.amazon.com` | A prefix to further organize zones |

### DNS Resolution Path
Client → Local DNS Resolver → Root DNS → TLD DNS → Authoritative DNS (Hosted Zone) → Response

Root servers controlled by **ICANN**, TLD servers by **IANA**, domain-level DNS by **Registrar** (Route 53, GoDaddy, Papaki, etc.)

---

## 📡 Amazon Route 53 Overview

**Route 53 is:**
- ✅ **Scalable**, fully managed **DNS**
- ✅ **Authoritative** (we control records)
- ✅ Can act as a **Domain Registrar**
- ✅ **100% SLA** availability
- 53 refers to the traditional **DNS port (53)**

### Each DNS record contains:
- **Name** (domain/subdomain)
- **Record type**
- **Value** (IP / hostname / resource)
- **Routing policy** (how Route 53 responds to queries)
- **TTL** (Time To Live cache duration)

---

## 🧾 DNS Record Types

| Type | Purpose | Example |
|------|---------|---------|
| **A** | Maps hostname → IPv4 | `api.example.com → 54.2.1.5` |
| **AAAA** | Maps hostname → IPv6 | `api.example.com → 2001:db8::2` |
| **CNAME** | Redirects hostname → another hostname | `app.example.com → login.myapp.com` |
| **NS** | Defines **authoritative name servers** for a zone | Used when delegating domain to Route 53 |

⚠️ **CNAME cannot be used at the domain root (zone apex)**  
Example of domain apex: `example.com`

---

## 🏡 Hosted Zones

| Type | Example | Use |
|------|---------|-----|
| **Public Hosted Zone** | `app.mydomain.com` | Accessible on the public internet |
| **Private Hosted Zone** | `db.company.internal` | Only accessible inside a VPC |

💰 Cost: **$0.50/month per hosted zone**  
Domain registration ≈ **$12/year** minimum

---

## ⏱ TTL (Time To Live)
How long DNS results are cached.

| TTL | Behavior |
|------|----------|
| **High (e.g., 24h)** | Lower cost, but old records persist longer |
| **Low (e.g., 60s)** | Faster failover & updates, increased DNS query cost |

TTL is **required** for all DNS records **except Alias**.

```bash
sudo yum install -y bind-utils
dig api.example.com
```

```bash
winget install ISC.Bind
dig api.example.com
```

---

## 🔁 CNAME vs Alias

### **CNAME**
- Maps hostname → **another hostname**
- ✅ Use for **subdomains only**
- ❌ Cannot be used at **domain root**

### **Alias (AWS Extension)**
- Maps hostname → **AWS resources**
- Works for both **root domain & subdomains**
- **No TTL** required (AWS manages automatically)
- **Free & native health checks**
- Always behaves as **A/AAAA internally**
- Automatically recognise changes in the resource's ip addresses

### Alias can target:
- **Elastic Load Balancers** ✅ (very important)
- **CloudFront**
- **API Gateway**
- **Elastic Beanstalk**
- **S3 Static Websites**
- **VPC Interface Endpoints**
- **Global Accelerator**
- **Other Route 53 records in same zone**

❌ Alias **cannot** target EC2 public DNS names.

---

## 🎯 Routing Policies

### **Simple Routing**
- Route traffic to a single resource
- Can specify multiple values in the same record
- If multiple values are returned, a random one is chosen by the **client**!
- ❌ Cannot use health checks (unless Alias)

---

### **Weighted Routing**
Control percentage of traffic per resource.

- A record weight 80 → 80% traffic
-  record weight 20 → 20% traffic

✅ Used for:
* Gradual deployments
* A/B Testing
* Load-balancing between regions
* Canary releases (**send 5% traffic to new version to test updates**)

* Can be assosicated with health checks
* Weights don't need to sum up to 100. They are relative
* The final percentage is the weight of this record / against the total weight of all records
* Can use **weight = 0** to disable resource without deleting record. If all resources have weight 0, they are all equally returned.

---

### **Latency-Based Routing**
Routes user to the region with **lowest latency**.

* Super helpful when response time is priority
✅ Perfect for global applications  
✅ Can be used **with failover (health checks)**

> When changing my IP with VPN, all TTL are cleared so we can get another response immediately.

---

## ❤️ Route 53 Health Checks

Used for **automated DNS failover**.

### Health Check Types
| Type | Works With | Notes |
|------|------------|-------|
| **Endpoint checks** | Public endpoints | 15 global checkers verify |
| **Calculated Check** | Combines results of others | AND/OR/“N of M healthy” logic |
| **CloudWatch Alarm Check** | Private resources | Works when endpoint is inside VPC |

### Endpoint Health Checks
- Default check interval: **30s** (can reduce to **10s** → higher cost)
- Failure threshold (number of times to be flagged as failed) (default = 3)
- Checks **HTTP, HTTPS, TCP**
- Must allow Route 53 health checkers IP ranges on firewall (they are specific)
- If > 18% report healthy endpoint, Route 53 will consider it healthy.
- Choose which locations Route 53 to use.
- Checks are passed with 2xx and 3xx status codes
- Can be set to check texts to the first 5120 bytes of the response.

---

### Calculated Health Checks
Combine up to **256** health checks.  
- It's like the parent that monitors child health checks
- Specify how many health checks need to pass to make parent pass

- At least X of Y health checks are healthy,
- ALL health checks are healthy,
- One or more health checks are healthy

Useful for:
- Maintenance windows without causing all health checks to fail
- Aggregated service health

### Private Hosted Zone
* Health checks are outside of the VPC so they cannot access private endpoints.
* But we can create CloudWatch Metrics and associate CloudWatch alarms, then the health check can check these alarms.
* Full control in throttles of DynamoDB, alarms on RDS, custom metrics, etc.

---

## 🩺 **Failover Routing**
* Basically they work with health checks,
* One record is primary, the other is secondary
* Based on the health check, the traffic will go to the corresponding record
* You can create only one failover record of each type.

---

### **Geolocation Routing**
Routes based on **user’s location**.

* Specify continent, country, or even a US State
* Use cases: website localization, retrict content distribution, load balancing etc.
* Can be associated with health checks

Requires **Default** record for unmatched traffic.

---

### **Geoproximity Routing**
Routes based on **location + adjustable bias**.
* Ability to shift more traffic to resources based on the defined 'bias' and location

Bias:
- Positive → **Expand region** 1 to 99
- Negative → **Shrink region** -1 to -99

Resources can be:
* AWS Resources(AWS Region)
* Non-AWS Resources (specify latitude and longitude)
* Use feature traffic flow

---

### **IP-Based Routing**
Map **IP ranges (CIDRs)** → specific endpoints.

```
Example:
Route 53 -> CIDR Collection:
203.0.113.0/24 → location 1
200.5.4.0/24 → location 2

Records:
example.com -> 1.2.3.4 -> location 1
example.com -> 5.6.7.8 -> location 2
```

* Routing is based on client's IP address
* Use cases: optimize performance, reduce network costs, route user from particular ISP to a specific endpoint.

---

### **Multivalue Routing**
* Route traffic to multiple resources
- Responds with **multiple IPs**
- Acts like **client-side load balancing**
- Returns **Up to 8** healthy records
- ✅ Supports health checks (return only values for healthy resources)
- ❌ Not a replacement for ELB, but still useful
- It's different from the simple policy, because it only returns healthy resources. Also simple ones, do NOT support health checks.

---

## Domain Registrar vs DNS Service
* The Domain Registars usually provide us with a DNS service to manage our DNS records.
* But we can **buy a domain** in one place and **host DNS** somewhere else.

Example:
- Domain registered in **GoDaddy**
- DNS hosted in **Route 53**

To do this:
1. Create hosted zone in Route 53
2. Replace GoDaddy NS records with Route 53 NS values

---

## 🔗 Hybrid DNS (On-Prem + AWS)

**Route 53 Resolver** handles private & on-prem DNS. Anwsers queries for:
* Local domain names for EC2 instances
* Records in private hosted zones
* Records in public NameServers

Hybrid DNS: Resolving DNS queries beween VPS (Route 53 resolver) and networks (other DNS resolvers)

Networks can be: VPC itself / Peered VPS or on-premises network connected through direct connect or AWS VPN

### Endpoints:
| Type | Purpose |
|------|---------|
| **Inbound Endpoint** | On-prem → Resolve **AWS private** DNS. Listen to queries from outside. |
| **Outbound Endpoint** | AWS → Resolve **on-prem** DNS. Forward queries to DNS Resolvers on the on-premices data center. |

---