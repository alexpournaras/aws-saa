# AWS Elastic Beanstalk

## Key Characteristics
- **Centralized Deployment View:** Provides an easy way to deploy and manage applications.
- **Uses Core AWS Services:** EC2, Auto Scaling Groups (ASG), Elastic Load Balancer (ELB), RDS, etc.
- **Fully Managed Service:** Automatically handles:
  - Capacity provisioning
  - Load balancing
  - Auto scaling
  - Application health monitoring
  - Instance configuration
- **Developer Control:** You retain full control to customize configurations if needed.
- **Cost:** Elastic Beanstalk itself is free. You pay only for the resources your application uses (e.g., EC2 instances, RDS, ELB).

## Core Components
| Component | Description |
|---------|-------------|
| **Application** | A collection of Elastic Beanstalk elements, including environments, versions, and configuration settings. |
| **Environment** | A collection of AWS resources running *one* application version at a time. |
| **Application Version** | A specific deployable build of your application. |

## Environment Tiers
| Tier | Purpose | Typical Services |
|------|---------|----------------|
| **Web Server Tier** | For handling HTTP(S) requests. | Uses ELB, ASG, EC2, etc. |
| **Worker Tier** | Processes background tasks. | Uses SQS to queue work and EC2 worker instances. |

Workers can be fed tasks from the Web Server tier. Worker environments can automatically scale based on the number of SQS messages.

## Multi-Environment Workflow
You can create separate environments, such as:
- **Development**
- **Testing**
- **Production**

This supports safe iterative deployment and testing practices.

### Deployment Flow
Create Application → Upload Application Version → Deploy to Environment → Manage / Scale / Monitor

## Supported Platforms
- Go
- Java
- .NET
- Node.js
- PHP
- Python
- Ruby
- Packr Builder
- **Docker** (single or multi-container)

## Deployment Models

### Single Instance (Typical for Development)
- One EC2 instance (with Elastic IP)
- Optional RDS single instance (master-only)
- Simple, low-cost setup

### High Availability (Production)
- Load balancer (ELB)
- Auto Scaling Group with multiple EC2 instances across multiple AZs
- RDS Multi-AZ for failover (primary + standby replica)

---

Elastic Beanstalk is especially useful for **developers who want fast deployments without managing infrastructure**, while still maintaining the option to customize configurations when needed.