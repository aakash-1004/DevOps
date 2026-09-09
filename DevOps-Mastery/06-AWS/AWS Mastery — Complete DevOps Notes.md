# AWS Mastery — Complete DevOps Notes

**Tags:** #aws #devops #cloud #interview #masterreview **Status:** ✅ Active Reference **Interview Relevance:** 🔴 Maximum **Level:** DevOps Engineer / Cloud Engineer (2-3 YOE target)

> **How to use this note:** Every service follows: Concept → Real Configuration → Production Use Case → Memory Hint → Interview Answer. Interview answers are framed as a senior DevOps engineer would give — grounded in production reality.

---

# TABLE OF CONTENTS

1. [Virtualization & Cloud Computing Foundation](https://claude.ai/chat/b1cae23e-7e28-4e55-be9d-e54f24a46644#1-virtualization--cloud-computing-foundation)
2. [AWS Overview](https://claude.ai/chat/b1cae23e-7e28-4e55-be9d-e54f24a46644#2-aws-overview)
3. [IAM — Identity & Access Management](https://claude.ai/chat/b1cae23e-7e28-4e55-be9d-e54f24a46644#3-iam--identity--access-management)
4. [EC2 — Elastic Compute Cloud](https://claude.ai/chat/b1cae23e-7e28-4e55-be9d-e54f24a46644#4-ec2--elastic-compute-cloud)
5. [EBS — Elastic Block Store](https://claude.ai/chat/b1cae23e-7e28-4e55-be9d-e54f24a46644#5-ebs--elastic-block-store)
6. [AMI — Amazon Machine Image](https://claude.ai/chat/b1cae23e-7e28-4e55-be9d-e54f24a46644#6-ami--amazon-machine-image)
7. [ELB — Elastic Load Balancer](https://claude.ai/chat/b1cae23e-7e28-4e55-be9d-e54f24a46644#7-elb--elastic-load-balancer)
8. [ASG — Auto Scaling Group](https://claude.ai/chat/b1cae23e-7e28-4e55-be9d-e54f24a46644#8-asg--auto-scaling-group)
9. [S3 — Simple Storage Service](https://claude.ai/chat/b1cae23e-7e28-4e55-be9d-e54f24a46644#9-s3--simple-storage-service)
10. [RDS — Relational Database Service](https://claude.ai/chat/b1cae23e-7e28-4e55-be9d-e54f24a46644#10-rds--relational-database-service)
11. [DynamoDB — NoSQL](https://claude.ai/chat/b1cae23e-7e28-4e55-be9d-e54f24a46644#11-dynamodb--nosql)
12. [Lambda — Serverless Compute](https://claude.ai/chat/b1cae23e-7e28-4e55-be9d-e54f24a46644#12-lambda--serverless-compute)
13. [CloudFormation — IaC](https://claude.ai/chat/b1cae23e-7e28-4e55-be9d-e54f24a46644#13-cloudformation--iac)
14. [Route 53 — DNS](https://claude.ai/chat/b1cae23e-7e28-4e55-be9d-e54f24a46644#14-route-53--dns)
15. [CloudFront — CDN](https://claude.ai/chat/b1cae23e-7e28-4e55-be9d-e54f24a46644#15-cloudfront--cdn)
16. [ECS — Elastic Container Service](https://claude.ai/chat/b1cae23e-7e28-4e55-be9d-e54f24a46644#16-ecs--elastic-container-service)
17. [EKS — Elastic Kubernetes Service](https://claude.ai/chat/b1cae23e-7e28-4e55-be9d-e54f24a46644#17-eks--elastic-kubernetes-service)
18. [Amplify — Fullstack Platform](https://claude.ai/chat/b1cae23e-7e28-4e55-be9d-e54f24a46644#18-amplify--fullstack-platform)
19. [Interview Master Q&A](https://claude.ai/chat/b1cae23e-7e28-4e55-be9d-e54f24a46644#19-interview-master-qa)

---

---

# 1. Virtualization & Cloud Computing Foundation

## Virtualization

**Concept:** Creating multiple simulated environments (virtual machines) from a single physical hardware system. One physical server hosts many virtual servers, each isolated with its own OS.

**Why it exists:** Before virtualization — one physical server ran one OS ran one app. Servers utilized only 5-15% of capacity. Waste. With virtualization — one physical server runs 20+ VMs. Utilization jumps to 70-80%. This is the foundation of cloud computing.

**Hypervisor** — the software that creates and manages VMs.

```
Type 1 (Bare-metal Hypervisor)         Type 2 (Hosted Hypervisor)
─────────────────────────────          ─────────────────────────────
Runs directly on hardware               Runs on top of a host OS
No host OS                              Requires host OS
Better performance                      Easier to install
Examples: VMware ESXi, KVM,             Examples: VirtualBox,
Hyper-V, Xen                            VMware Workstation
Used in: production data centers        Used in: dev laptops, learning
                                        (your devops-labs VM = Type 2)
```

**Real world context:** AWS EC2 runs on the **Nitro hypervisor** — a specialized KVM variant (Type 1). Every EC2 instance you spin up is a VM running on Nitro. That's why AWS can offer consistent IOPS and network performance with SLA guarantees.

**Memory hint:**

- Type 1 = "sits on the metal" — no host OS
- Type 2 = "sits on the OS" — needs Windows/Linux underneath

---

## Cloud Computing

**Definition:** On-demand delivery of IT resources over the internet with pay-as-you-go pricing.

**Instead of:** Buying and maintaining physical hardware **You get:** Servers, storage, databases, networking rented from a provider

**Three Service Models:**

```
IaaS (Infrastructure as a Service)
    ├── You manage: OS, runtime, apps, data
    ├── Provider manages: physical hardware, networking, virtualization
    └── Example: AWS EC2, Azure VMs

PaaS (Platform as a Service)
    ├── You manage: apps, data
    ├── Provider manages: OS, runtime, hardware
    └── Example: AWS Elastic Beanstalk, Heroku, App Engine

SaaS (Software as a Service)
    ├── You manage: nothing except your usage
    ├── Provider manages: everything
    └── Example: Gmail, Salesforce, Office 365
```

**Memory hint — "Pizza as a Service":**

- **On-premises** = You make pizza at home (buy everything)
- **IaaS** = Take & bake pizza (they give ingredients, you cook)
- **PaaS** = Pizza delivery (they cook, you set the table)
- **SaaS** = Dine at restaurant (they do everything)

---

## Cloud Deployment Models

```
Public Cloud
    Shared cloud environment
    Multiple users share the underlying infrastructure
    Examples: AWS, Azure, GCP
    Best for: startups, most workloads

Private Cloud
    Dedicated environment for ONE organization
    More control, more privacy, more expensive
    Examples: VMware Private Cloud, OpenStack
    Best for: banks, government, highly regulated industries

Hybrid Cloud
    Mix of public and private
    Sensitive data on private, elastic workloads on public
    Best for: enterprises transitioning to cloud
```

**Real world context:** Most modern companies use hybrid or multi-cloud. Bank keeps customer data on private, uses AWS for the mobile app. Netflix uses AWS but keeps some workloads on their own hardware. Multi-cloud (AWS + GCP + Azure) is common for avoiding vendor lock-in in large enterprises.

---

## Interview Q&A

**Q: What is cloud computing and why has it become dominant?**

> "Cloud computing is on-demand delivery of IT resources over the internet with pay-as-you-go pricing. Instead of buying and maintaining hardware, you rent it from providers like AWS.
> 
> It became dominant for three reasons. First, CapEx to OpEx conversion — no huge upfront hardware investment, you pay only for what you use. Second, elasticity — need 100 servers for Black Friday? Scale up in minutes, scale down after. Third, global reach — deploy to 30+ regions worldwide without building data centers.
> 
> For developers and DevOps engineers, cloud enabled the entire DevOps movement — Infrastructure as Code, immutable infrastructure, and rapid iteration are only possible because you can create and destroy infrastructure programmatically."

**Q: Difference between IaaS, PaaS, and SaaS with real examples?**

> "IaaS gives you raw infrastructure — VMs, storage, networking. You manage the OS and everything above it. AWS EC2 is IaaS — you get a server, you install what you want.
> 
> PaaS gives you a platform to deploy code — you don't touch OS or servers. AWS Elastic Beanstalk is PaaS — push your code, it handles servers, load balancers, scaling. Great for developers who don't want to manage infrastructure.
> 
> SaaS is fully managed software you just use. Gmail, Salesforce — you don't manage anything, just use the app.
> 
> As a DevOps engineer I work primarily with IaaS (EC2, EBS, VPC) and increasingly with PaaS-like services (ECS, Lambda) that reduce operational overhead."

---

---

# 2. AWS Overview

## What is AWS?

**Amazon Web Services** — launched in 2006 as the first public cloud platform. Started with S3 (storage) and EC2 (compute). Now offers 200+ services across compute, storage, networking, databases, AI/ML, IoT, and more.

**Pay-as-you-go model** — you pay only for what you use, billed per second/hour/GB.

## AWS Global Infrastructure

```
Regions (33+ worldwide)
    ├── Physical locations around the world
    ├── Each region is INDEPENDENT
    ├── You choose which region(s) to deploy in
    └── Example: ap-south-1 (Mumbai), us-east-1 (N. Virginia)

Availability Zones (AZs)
    ├── Each region has 3+ AZs
    ├── Each AZ = one or more physical data centers
    ├── AZs in the same region are connected by low-latency links
    └── Isolated from each other — one AZ failure doesn't affect others

Edge Locations (400+)
    ├── Used by CloudFront (CDN) and Route 53
    ├── Closer to end users than regions
    └── Deliver content with low latency
```

**Real world context:** Your AWS setup: `ap-south-1` (Mumbai) region, account `956179206096`, IAM user `devops-admin`. Mumbai has 3 AZs — `ap-south-1a`, `ap-south-1b`, `ap-south-1c`. For high availability, always deploy across at least 2 AZs.

## Why AWS Dominates

1. **Scalability** — scale from 1 to 1 million users
2. **Global reach** — 33+ regions, 400+ edge locations
3. **Reliability** — 99.99% SLA on many services
4. **Security** — compliance with SOC, ISO, PCI DSS, HIPAA
5. **Ecosystem** — huge partner network, tools, expertise available

## Common AWS Career Roles

- Cloud Engineer / Solutions Architect
- **DevOps Engineer** ← your target
- Cloud Developer / Data Engineer
- Site Reliability Engineer (SRE)
- Cloud Security Specialist
- Cloud Support Associate

**Memory hint:** AWS launched with S3 and EC2 — **storage + compute** are the two pillars of cloud. Everything else is built on top.

---

---

# 3. IAM — Identity & Access Management

## Concept

IAM controls **WHO** can access **WHAT** in your AWS account. It's the foundation of AWS security.

**Facts:**

- **Free service** — no additional cost
- **Global service** — not region-specific (unlike most AWS services)
- **Root account** created by default when you sign up — should NEVER be used for daily operations

## Four Core Entities

```
User
    ├── Individual identity for a person or application
    ├── Has permanent credentials (username/password, access keys)
    ├── Attached policies define what they can do
    └── Example: aakash-dev, jenkins-ci

Group
    ├── Collection of users sharing common permissions
    ├── Assign policy to group → all users in group inherit it
    ├── Simplifies management: 100 developers = one Developers group
    └── Example: Developers, Admins, ReadOnly

Role
    ├── Assumable identity — NO permanent credentials
    ├── Temporary access via STS tokens
    ├── Used by AWS services (EC2, Lambda) or cross-account access
    └── Example: EC2-S3-ReadOnly-Role, GitHub-Actions-Deploy-Role

Policy
    ├── JSON document defining permissions (Allow/Deny + Actions + Resources)
    ├── Attached to Users, Groups, or Roles
    ├── Two types: AWS Managed (predefined) and Customer Managed (you create)
    └── Example: AmazonS3ReadOnlyAccess (managed), MyCustomPolicy (customer)
```

## IAM Policy Structure (JSON)

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "s3:GetObject",
        "s3:PutObject"
      ],
      "Resource": "arn:aws:s3:::my-bucket/*"
    },
    {
      "Effect": "Deny",
      "Action": "s3:DeleteObject",
      "Resource": "*"
    }
  ]
}
```

**Breaking it down:**

- **Effect**: `Allow` or `Deny` — Deny always wins over Allow
- **Action**: what operations (e.g., `s3:GetObject`, `ec2:StartInstances`)
- **Resource**: which resources (use ARN — Amazon Resource Name)
- **Condition** (optional): when the rule applies (IP restrictions, time of day, MFA required)

## MFA — Multi-Factor Authentication

Adds a second layer of security beyond password:

1. Something you **know** (password)
2. Something you **have** (phone with authenticator app)

```
Login flow with MFA:
Username + Password → Correct? → Prompt for MFA code → Correct? → Access granted
```

**Options:**

- **Virtual MFA** (Google Authenticator, Duo, Authy) — most common
- **U2F Security Key** (YubiKey) — hardware key
- **Hardware MFA** — physical device from AWS

**Rule:** ALWAYS enable MFA on:

- Root account (mandatory)
- All IAM users
- Especially anyone with admin/write permissions

## Ways to Access AWS

```
1. AWS Console (web UI)
   ├── Human access via browser
   └── Login with username/password + MFA

2. AWS CLI
   ├── Command line access from terminal
   ├── Requires Access Key ID + Secret Access Key
   └── aws configure to set up

3. AWS SDK
   ├── Programmatic access from applications
   ├── Available for Python (boto3), Node.js, Java, Go, etc.
   └── Uses credentials or IAM roles

4. AWS APIs (REST)
   ├── Direct HTTP calls to AWS APIs
   └── SDKs are wrappers around these
```

## Real Production Setup

**Your setup (as per notes):**

```
Root Account:  aakash@example.com (protected, MFA enabled, never used daily)
    ↓
IAM User:  devops-admin
    ├── Access Keys for CLI/programmatic
    ├── MFA enabled
    └── Attached policies: AdministratorAccess (for learning)

For real projects — create separate users:
    ├── developer (limited to dev environment)
    ├── jenkins-ci (limited to specific deployments)
    └── read-only-audit (view-only for audits)
```

## Least Privilege Principle

**Never grant `AdministratorAccess` in production.** Start with zero permissions, grant only what's needed.

**Bad:**

```json
{"Effect": "Allow", "Action": "*", "Resource": "*"}
```

**Good:**

```json
{
  "Effect": "Allow",
  "Action": ["s3:GetObject", "s3:PutObject"],
  "Resource": "arn:aws:s3:::my-app-bucket/*"
}
```

If compromised — attacker can only touch that one bucket. Blast radius contained.

## Memory Hints

- **User** = permanent employee badge
- **Role** = visitor pass (expires)
- **Policy** = the actual permissions written down
- **Group** = department (everyone in it gets the same access)
- **MFA** = double-lock on your door

## Interview Q&A — IAM

**Q: What's the difference between IAM User and IAM Role? When do you use each?**

> "IAM User is a permanent identity with long-lived credentials — access key and secret key. Used for humans logging in to console, or for applications with permanent credentials.
> 
> IAM Role is an assumable identity — no permanent credentials. When assumed, it generates short-lived STS tokens, typically valid for 1 hour. Roles are used for:
> 
> - EC2 instances (attached as instance profile) so apps can call AWS APIs without hardcoded keys
> - Lambda functions
> - Cross-account access
> - CI/CD systems like GitHub Actions via OIDC
> 
> The golden rule: NEVER put access keys on an EC2 instance or in application code. Always use IAM Roles. If you see `aws_access_key_id` in a config file on a server, that's a security finding — flag it immediately."

**Q: Your EC2 app needs read access to S3. What's the correct way to grant it?**

> "Create an IAM Role with an S3 policy — specifically `s3:GetObject` on the target bucket ARN, not `s3:*`. Attach that role to the EC2 instance as an instance profile.
> 
> The application code uses AWS SDK without any explicit credentials. The SDK automatically hits the EC2 instance metadata service at `169.254.169.254/latest/meta-data/iam/security-credentials/` to get temporary credentials from the role. It refreshes them before expiry.
> 
> Never add credentials to code or environment variables on the instance. Roles handle authentication transparently and rotate automatically."

**Q: Explain least privilege and give me a real example.**

> "Least privilege means granting only the minimum permissions needed for the job, nothing more.
> 
> Example: A Lambda function processes uploaded images from S3 and writes results to a specific DynamoDB table. Bad approach — attach `AmazonS3FullAccess` and `AmazonDynamoDBFullAccess`. Good approach — custom policy allowing only `s3:GetObject` on the specific bucket prefix, and `dynamodb:PutItem` on the specific table.
> 
> If that Lambda is ever compromised, the attacker can only read one S3 prefix and write to one table. Blast radius contained. Without least privilege, they'd have full read/write access to every S3 bucket and every DynamoDB table in the account."

---

---

# 4. EC2 — Elastic Compute Cloud

## Concept

EC2 provides **resizable virtual servers** in the cloud, called **instances**. Instead of buying and managing physical servers, you rent VMs on demand.

Under the hood, EC2 runs on the Nitro hypervisor. Each instance you launch is a VM with dedicated CPU, RAM, storage, and network.

## Configuration Options

When launching an EC2 instance, you configure:

```
1. AMI (Amazon Machine Image)
   ├── The OS + pre-installed software
   ├── Ubuntu 22.04, Amazon Linux 2, Windows Server, custom AMI
   └── Determines what's installed at boot

2. Instance Type
   ├── The hardware capacity (CPU, memory, network)
   ├── t3.micro, m5.large, c5.xlarge, etc.
   └── Family + size (t = burstable, m = general, c = compute, r = memory)

3. Storage
   ├── Root EBS volume (default 8GB)
   ├── Additional EBS volumes if needed
   └── Type: gp3, gp2, io1, io2

4. Security Groups
   ├── Firewall rules — controls inbound/outbound traffic
   ├── Only "Allow" rules (no explicit deny)
   └── Stateful — return traffic auto-allowed

5. Key Pair
   ├── SSH access to Linux instances
   ├── RDP access for Windows instances
   └── Public key on instance, private key you download (.pem file)

6. Network Settings
   ├── VPC (Virtual Private Cloud) — your isolated network
   ├── Subnet (public or private)
   ├── Public IP assignment (yes/no)
   └── Elastic IP (static public IP, optional)

7. IAM Role
   ├── Attach role for AWS API access from the instance
   └── Never store credentials — always use role

8. User Data
   ├── Bootstrap script that runs at first boot
   ├── Installs software, configures the instance
   └── Runs as root
```

## Instance Type Families

```
t (t2, t3, t4g)  → Burstable — cheap, good for dev/test, low CPU with occasional bursts
m (m5, m6i)      → General purpose — balanced CPU/memory
c (c5, c6i)      → Compute optimized — CPU-heavy workloads (rendering, HPC)
r (r5, r6i)      → Memory optimized — databases, caching, analytics
i (i3, i4i)      → Storage optimized — NVMe SSD storage
p (p3, p4)       → GPU — machine learning, deep learning
g (g4, g5)       → GPU — graphics workloads
```

**Your context:** Your account requires `t3.micro` (not `t2.micro`) for free tier eligibility.

## User Data Script Example

```bash
#!/bin/bash
sudo yum update -y

# Install Apache web server
sudo yum install -y httpd
sudo systemctl start httpd
sudo systemctl enable httpd

# Create a simple HTML page
echo "<html><h1>Welcome to Apache on Amazon Linux!</h1></html>" > /var/www/html/index.html
```

**Real world context:** In production, user_data typically:

1. Pulls the latest application code from S3 or Git
2. Installs Docker
3. Starts your containerized app
4. Registers with a monitoring system
5. Reports success to CloudWatch

This is how Auto Scaling Groups launch new instances that are immediately ready to serve traffic — no manual setup.

## Security Groups (SG)

**What they are:** Virtual firewalls at the instance level.

**Rules:**

- Region-specific
- Only **Allow** rules (no explicit deny)
- All inbound blocked by default, all outbound allowed
- **Stateful** — if you allow inbound port 80, response traffic is automatically allowed outbound

**Define rules for:**

- Protocol (HTTP, HTTPS, SSH, custom TCP/UDP)
- Port number (80 for HTTP, 22 for SSH, 3306 for MySQL)
- Source (IP address, CIDR range, or another Security Group)

## Common Ports to Know

```
22    → SSH — Linux remote access
3389  → RDP — Windows remote access
80    → HTTP — unencrypted web
443   → HTTPS — encrypted web (SSL/TLS)
21    → FTP — file transfer (unsecured)
25    → SMTP — email sending
3306  → MySQL
5432  → PostgreSQL
1433  → MS SQL Server
27017 → MongoDB
6379  → Redis
```

**Interview trick question:** What port does SFTP use? Answer: **Port 22** — same as SSH (SFTP is SSH-based). Not port 21 (that's FTP).

## Practical: Creating an EC2 Instance

```bash
# Via AWS CLI
aws ec2 run-instances \
  --image-id ami-0abcdef1234567890 \
  --instance-type t3.micro \
  --key-name my-key \
  --security-group-ids sg-xxxxx \
  --subnet-id subnet-xxxxx \
  --user-data file://userdata.sh \
  --tag-specifications 'ResourceType=instance,Tags=[{Key=Name,Value=web-server}]'
```

**Common EC2 commands:**

```bash
aws ec2 describe-instances                    # list all instances
aws ec2 start-instances --instance-ids i-xxx  # start stopped instance
aws ec2 stop-instances --instance-ids i-xxx   # stop running instance
aws ec2 terminate-instances --instance-ids i-xxx  # destroy permanently
```

## Stop vs Terminate

```
Stop
    ├── Instance shuts down but persists
    ├── EBS root volume preserved (data safe)
    ├── Public IP released (unless Elastic IP)
    ├── You still pay for EBS storage
    └── Can restart anytime

Terminate
    ├── Instance destroyed permanently
    ├── EBS root volume DELETED (unless DeleteOnTermination=false)
    ├── All data gone
    └── Stop paying immediately

Hibernate
    ├── RAM contents saved to EBS
    ├── Faster restart than stop
    └── Instance "remembers" its state
```

## Elastic IP

**Problem:** Public IP changes every time you stop/start an instance. **Solution:** Elastic IP = static public IP you own.

- Free while attached to a running instance
- Costs money if allocated but not used (AWS penalizes waste)
- Reassign to a different instance during failover

## Real World Production Setup

```
Web tier:
  m5.large × 3 instances (across 3 AZs)
  Behind ALB
  In Auto Scaling Group
  Public subnets

App tier:
  m5.xlarge × 5 instances
  Behind internal NLB
  Private subnets
  Reach out via NAT Gateway

Database tier:
  RDS Multi-AZ (not raw EC2)
  Private subnets
  Accessible only from app tier SG
```

## Memory Hints

- **Instance** = a VM (rented server)
- **AMI** = OS template (the "operating system in a box")
- **Instance Type** = hardware specs (t3.micro = 1 vCPU, 1GB RAM)
- **Security Group** = firewall for one instance (stateful)
- **Key Pair** = your house key (private key you keep, public key on the instance)
- **Elastic IP** = phone number that doesn't change (static)

## Interview Q&A — EC2

**Q: Walk me through what happens when you launch an EC2 instance.**

> "AWS receives the request via API. First, it validates permissions — your IAM policy must allow `ec2:RunInstances`. Then the scheduler picks a physical host in your specified AZ that has capacity for the instance type. The Nitro hypervisor allocates CPU, RAM, and network resources.
> 
> The AMI is copied to the instance's root EBS volume. Instance boots up, user_data script runs as root. Security Group rules are applied — inbound traffic filtered per your rules. If public IP is enabled, it's assigned. Instance transitions from `pending` to `running` state.
> 
> Total time — usually 30-60 seconds for standard AMIs, longer for large custom AMIs. This is why AMIs matter — a well-built AMI with everything pre-installed launches in seconds vs. minutes for a bare AMI that needs user_data to install everything."

**Q: Stop vs Terminate an EC2 instance — what happens to data?**

> "Stop preserves everything on EBS volumes. The instance shuts down like a laptop hibernating — you can start it again later, and all data is there. You still pay for EBS storage even when stopped. Public IP changes on restart unless you have an Elastic IP attached.
> 
> Terminate destroys the instance permanently. The root EBS volume is deleted by default — data gone. You can override this by setting DeleteOnTermination to false on the volume, but this is unusual.
> 
> In production, we terminate instances all the time — Auto Scaling Groups terminate old instances during rolling updates, on-demand instances get terminated when demand drops. The rule is: never store data on an EC2 root volume that you can't afford to lose. Persistent data goes to EBS with DeleteOnTermination=false, or to S3, or to RDS."

**Q: How does user_data help in Auto Scaling scenarios?**

> "User_data is the bootstrap script that runs when an instance first launches. In Auto Scaling, this is critical because new instances need to be production-ready immediately — no human intervention.
> 
> Typical user_data flow: update OS packages, install Docker, pull the latest application image from ECR, start the container with proper environment variables, and register with monitoring. When ASG scales up during traffic spike, new instances follow this exact flow and start serving traffic in 1-2 minutes.
> 
> The alternative is baking everything into a custom AMI — faster launch (30 seconds vs 2 minutes) but slower iteration since you rebuild the AMI for every app change. The pattern I recommend: bake OS + Docker + monitoring agents into the AMI, use user_data only to pull latest code. Best of both worlds."

---

---

# 5. EBS — Elastic Block Store

## Concept

EBS provides **durable, high-performance block storage** for EC2 instances. Think of it as a virtual hard drive attached to your VM.

**Key characteristics:**

- Persistent — data survives even when EC2 stops/terminates (unless configured otherwise)
- Attached to ONE instance at a time (with exceptions like io2 Multi-Attach)
- Region and AZ specific — can only attach to instances in the same AZ

## EBS vs Instance Store

```
EBS
    ├── Persistent (data survives instance termination)
    ├── Detachable, reattachable
    ├── Snapshots for backup
    ├── Type: gp3, gp2, io1/io2, st1, sc1
    └── Use for: root volumes, databases, application data

Instance Store
    ├── Ephemeral (data GONE on stop/terminate)
    ├── Physically attached to host — fastest possible
    ├── No snapshots
    └── Use for: temporary caches, scratch space
```

**Rule:** If it's data you can't lose, use EBS (or S3, or RDS).

## EBS Volume Types

```
gp3 (Newest general-purpose SSD)
    ├── Default choice for most workloads
    ├── 3,000 IOPS baseline, scalable to 16,000
    ├── Cheaper than gp2
    └── Use for: root volumes, most apps

gp2 (Older general-purpose SSD)
    ├── Legacy — use gp3 instead
    ├── IOPS scales with volume size
    └── Being phased out

io1 / io2 (Provisioned IOPS SSD)
    ├── High-performance, mission-critical workloads
    ├── Up to 64,000 IOPS (io2 Block Express: 256,000 IOPS)
    ├── More expensive
    └── Use for: high-transaction databases, latency-sensitive apps

st1 (Throughput-optimized HDD)
    ├── Big data, log processing, sequential I/O
    ├── Low cost per GB
    └── Use for: data warehouses, log analytics

sc1 (Cold HDD)
    ├── Lowest cost
    ├── Infrequently accessed
    └── Use for: cold archives, backup
```

**Real world context:** Default choice = `gp3`. Only switch to `io2` for high-transaction databases where you can measure the IOPS requirement.

## Key Features

**1. Built-in Redundancy**

- Every EBS volume is automatically replicated within the same AZ
- Protects against hardware failure
- BUT — a single AZ failure = your volume is unavailable
- Solution: cross-AZ replication via snapshots

**2. Encryption**

- Enable encryption at rest with a single checkbox
- Uses AWS KMS keys
- No performance penalty
- Encrypt everything by default in production

**3. Snapshots (Backup)**

- Point-in-time copy of your volume
- Stored in S3 (behind the scenes)
- Incremental — only changed blocks are stored after the first snapshot
- Can copy to another AZ or region
- Can create a new volume from a snapshot

**4. Scalable**

- Resize the volume without stopping the instance
- Increase size, change type, adjust IOPS/throughput — all online
- After resizing, run `growpart` + `resize2fs` on Linux to expand filesystem

## Practical: Attaching EBS to EC2

```bash
# Create a new volume (via CLI)
aws ec2 create-volume \
  --volume-type gp3 \
  --size 20 \
  --availability-zone ap-south-1a

# Attach to an instance
aws ec2 attach-volume \
  --volume-id vol-xxxxx \
  --instance-id i-xxxxx \
  --device /dev/xvdf

# On the EC2 instance (Linux):
sudo lsblk                              # confirm new disk (e.g., /dev/xvdf)
sudo mkfs.ext4 /dev/xvdf                # format it
sudo mkdir /data                        # create mount point
sudo mount /dev/xvdf /data              # mount it
echo '/dev/xvdf /data ext4 defaults 0 0' | sudo tee -a /etc/fstab  # persist across reboots
```

## Snapshot Workflow

```
Create Snapshot
    ├── aws ec2 create-snapshot --volume-id vol-xxx
    ├── First snapshot = full backup
    └── Subsequent snapshots = incremental (only changed blocks)

Copy to Another AZ
    ├── Create snapshot → snapshot lives in region (multi-AZ)
    ├── Create new volume from snapshot in target AZ
    └── Attach new volume to instance in target AZ

Copy to Another Region
    ├── aws ec2 copy-snapshot --source-region ap-south-1 --destination-region us-east-1
    ├── Full copy across regions
    └── Used for disaster recovery
```

## Real Production Use Case

**Database on EBS:**

```
MySQL/PostgreSQL on EC2
    ↓
Root volume:  gp3 20GB (OS)
Data volume:  io2 500GB with 20,000 IOPS (database files)
    ↓
Daily snapshots via AWS Backup
    ↓
Cross-region copy for DR
```

**Storage extension scenario:**

```
Application logs filling /var/log
    ↓
Attach new 100GB EBS volume as /dev/xvdf
    ↓
mkfs.ext4 → mount at /var/log
    ↓
Move existing logs, restart app
    ↓
Root volume freed up, no downtime
```

## Memory Hints

- **EBS** = external hard drive for your VM
- **Snapshot** = backup photograph of your drive at a moment in time
- **gp3** = default choice (most cases)
- **io2** = expensive, only for high-IOPS databases
- **AZ-specific** = same AZ as your instance, always

## Interview Q&A — EBS

**Q: Difference between EBS and Instance Store?**

> "EBS is persistent block storage attached over the network — data survives instance stop and terminate. You can snapshot it, detach it, reattach it to another instance in the same AZ.
> 
> Instance Store is ephemeral — physically attached to the host, fastest possible I/O, but wiped when the instance stops or terminates. No snapshots, no detach.
> 
> Rule of thumb: EBS for anything you can't afford to lose, Instance Store for scratch space, temp files, or high-performance caching that you can rebuild.
> 
> Common pattern: root volume on EBS (persistent), Instance Store for a Redis cache that can be rebuilt from RDS if lost."

**Q: Your database is running out of storage. Walk me through extending EBS.**

> "Zero downtime EBS extension flow. First, in AWS console or CLI, modify the volume — increase the size. AWS handles this online, no instance restart needed. Takes a few minutes for the modification to complete.
> 
> Then on the Linux instance: `lsblk` confirms the new size at the block device level. Run `growpart /dev/xvda 1` to extend the partition to use the new space. Finally, `resize2fs /dev/xvda1` extends the ext4 filesystem online.
> 
> Total time — 5 minutes. Zero downtime, zero data loss. This is why we use LVM in some cases too — same concept, different layer of abstraction.
> 
> Common mistake: extending the volume in AWS but forgetting to resize the filesystem — the disk shows more space but the filesystem still thinks it's the old size. Two-step process, never skip step two."

**Q: How do you back up EBS volumes and restore in another region?**

> "EBS snapshots are the answer. Snapshots are incremental — first one is full backup, subsequent ones only store changed blocks. Stored in S3 behind the scenes, extremely durable.
> 
> Backup workflow: `aws ec2 create-snapshot --volume-id vol-xxx` — or better, use AWS Backup with a scheduled backup plan. AWS Backup handles retention policies, tagging, and cross-region replication automatically.
> 
> Restore in another region: `aws ec2 copy-snapshot --source-region ap-south-1 --destination-region us-east-1 --source-snapshot-id snap-xxx`. Then in the target region, create a new volume from that snapshot, attach to an instance.
> 
> For production DR: daily automated backups via AWS Backup, cross-region copy to a DR region, retention of 30 days for daily backups plus 12 months of monthly backups. RTO under 30 minutes for restore, RPO 24 hours max."

---

---

# 6. AMI — Amazon Machine Image

## Concept

An **AMI (Amazon Machine Image)** is a pre-configured template that provides everything needed to launch an EC2 instance:

- Operating system (Ubuntu, Amazon Linux, Windows)
- Pre-installed applications (Apache, Docker, etc.)
- Configuration files and settings

Think of an AMI as a snapshot of a fully configured machine that you can duplicate.

## Types of AMIs

```
Public AMIs
    ├── Available to all AWS users
    ├── Provided by AWS or community
    ├── Examples: Amazon Linux 2, Ubuntu 22.04, Windows Server 2022
    └── Use for: standard OS deployments

Private AMIs
    ├── Created by you, only in your account
    ├── Can be shared with specific AWS accounts
    ├── Contains your custom configuration
    └── Use for: production golden images

Paid AMIs / Marketplace AMIs
    ├── Third-party provided via AWS Marketplace
    ├── Include software licenses (RHEL, LAMP stack, Bitnami apps)
    ├── Extra hourly cost for the software
    └── Use for: enterprise software, rapid deployment
```

## Real Production Use Case — LAMP Stack Paid AMI

**Scenario:** You need to deploy a PHP web app quickly.

Using a paid LAMP AMI from AWS Marketplace:

- **Rapid Deployment:** Apache, MySQL, PHP pre-configured — skip installation
- **Scalability:** Launch multiple instances behind ALB in minutes
- **Cost:** Pay for the AWS instance + software license (usually small hourly fee)

## Creating a Custom AMI

**Workflow:**

1. Launch an EC2 instance from a base AMI (e.g., Amazon Linux 2)
2. Install your applications, configure everything
3. Run cleanup script (remove SSH keys, credentials, temp files)
4. Create an AMI from the instance
5. Launch new instances from your custom AMI — they come up fully configured

**Cleanup Script Before Creating AMI (CRITICAL):**

```bash
#!/bin/bash

# Remove SSH keys
rm -rf ~/.ssh/authorized_keys

# Clear user credentials and history
rm -rf ~/.aws/credentials
rm -rf ~/.git-credentials
rm -rf ~/.bash_history

# Clean system logs and temp files
rm -rf /var/log/*
rm -rf /tmp/*
rm -rf /var/tmp/*

# Remove temporary user accounts
deluser tempuser --remove-home

# Lock root account
passwd -l root

# Reset app-specific configs if needed
# rm -rf /etc/nginx/nginx.conf
```

**Why cleanup matters:** Without it, everyone who launches from your AMI gets your credentials, your bash history (which might contain passwords), your SSH keys. **This is a major security incident** — happens more often than you'd think in real companies.

## Launch Template vs Custom AMI

```
Launch Template
    ├── AWS resource that stores instance configuration
    ├── Doesn't include the OS/apps — points to an AMI
    ├── Reusable — Auto Scaling uses it to launch instances
    ├── Versioned — you can create v1, v2, v3 and switch between them
    └── Use for: standardizing how instances are launched

Custom AMI
    ├── The actual OS image with apps pre-installed
    ├── Used BY Launch Template
    └── Use for: bundling application code and dependencies

Common pattern:
    Custom AMI (contains OS + Docker + monitoring)
        ↓ referenced by
    Launch Template (instance type, security group, IAM role, user_data)
        ↓ used by
    Auto Scaling Group (launches instances based on template)
```

## EC2 Image Builder

**What it is:** AWS service to automate creation, testing, and deployment of AMIs.

**Features:**

- Runs at scheduled intervals (daily, weekly, monthly)
- Can automatically apply security patches to base AMIs
- Tests the AMI before publishing
- **Free service** — you only pay for the EC2 instances used during the build

**Real world context:** Enterprises use Image Builder to maintain a "golden AMI" — always patched, always up to date. Every month, Image Builder auto-generates a new AMI with latest security patches, tests it, and marks it as the latest. Auto Scaling Groups auto-adopt the new AMI. **Zero manual patching**.

## Immutable Infrastructure Pattern

**Traditional (mutable):**

```
Launch instance → SSH in → install updates → change configs → over time it drifts
```

**Immutable (with AMIs):**

```
Build new AMI with all changes → launch new instances from it → terminate old instances
Never modify running instances — always replace them
```

**Benefits:**

- Consistent state across all instances
- Rollback = launch instances from old AMI
- No configuration drift
- Faster incident response

## Memory Hints

- **AMI** = "OS snapshot with everything baked in"
- **Launch Template** = "recipe for how to bake a cake" (references the AMI)
- **AMI** = the cake, **Launch Template** = the recipe
- **Image Builder** = automated bakery (produces AMIs on schedule)

## Interview Q&A — AMI

**Q: What's the difference between a custom AMI and a Launch Template?**

> "They serve different purposes and work together.
> 
> Custom AMI is the actual OS image — contains the operating system, all your installed applications, and configuration. Think of it as a snapshot of a fully configured server.
> 
> Launch Template is a reusable configuration for how to launch an instance — specifies the AMI ID, instance type, security group, IAM role, user_data, and tags. It doesn't contain the OS itself, just points to which AMI to use.
> 
> Common pattern in production: build a custom AMI with Docker + monitoring agents + security tools baked in. Then create a Launch Template that references that AMI and specifies instance type m5.large, sec group X, IAM role Y. Auto Scaling Group uses the Launch Template to spawn identical instances on demand.
> 
> Update flow: rebuild AMI monthly with latest patches, update Launch Template to point to new AMI, ASG rolls new instances gradually."

**Q: How do you achieve immutable infrastructure with AMIs?**

> "Immutable infrastructure means you never modify running servers — you replace them.
> 
> Build phase: create a custom AMI with your app code, dependencies, and configuration baked in. Version this AMI (e.g., myapp-v1.2.3-2026-07-16). Test it in a staging environment.
> 
> Deploy phase: update the Launch Template to reference the new AMI. Trigger a rolling update in the Auto Scaling Group — ASG launches new instances with the new AMI, terminates old ones. Zero downtime if you have multiple instances behind a load balancer.
> 
> Benefits: consistent state (every instance is identical), easy rollback (point Launch Template back to old AMI, re-roll), no config drift (nobody SSHs in to change things), fast incident response (bad deploy = rollback in minutes).
> 
> This is why tools like Packer exist — automate AMI building as part of CI/CD. Your pipeline: code push → Packer builds AMI → tests → updates Launch Template → ASG rolling update."

---

---

# 7. ELB — Elastic Load Balancer

## Foundational Concepts First

Before ELB, understand three foundational terms:

**Scalability**

```
Ability to grow your system's resources when demand increases.

Vertical Scalability (Scaling Up):
    Add more power (CPU, RAM) to existing server
    Example: t2.micro → m5.large
    Limit: hardware maximum

Horizontal Scalability (Scaling Out):
    Add more instances behind a load balancer
    Example: 1 EC2 → 10 EC2 instances
    Limit: essentially unlimited in cloud
```

**High Availability (HA)**

```
Keeping your service running with minimal downtime.
Achieved by running resources across multiple AZs.
If one AZ fails, others take over.

Example: EC2 instances in AZ-a, AZ-b, and AZ-c behind ALB.
If AZ-a goes down → ALB routes traffic to AZ-b and AZ-c automatically.
```

**Elasticity**

```
Ability to automatically adjust resources based on demand.
Add resources during peak, remove during low usage.

Example: Auto Scaling Group scales from 2 to 20 instances during traffic spike.
Different from scalability — elasticity is automatic.
```

## Load Balancer — Concept

A **load balancer** distributes incoming traffic across multiple servers so no single server gets overloaded.

**Key benefits:**

- **Distributes traffic** — even load across instances
- **Improves availability** — routes around failed instances via health checks
- **Scales resources** — works with ASG for automatic scaling
- **Single point of access** — one DNS name for all your instances
- **HA across AZs** — spans multiple Availability Zones

## Types of AWS Load Balancers

```
1. Application Load Balancer (ALB) — Layer 7 (HTTP/HTTPS)
   ├── Content-based routing (path, host, headers)
   ├── SSL termination
   ├── WebSocket support
   ├── Integration with Cognito for auth
   └── Use for: web apps, microservices, APIs

2. Network Load Balancer (NLB) — Layer 4 (TCP/UDP)
   ├── Ultra-low latency (millions of requests per second)
   ├── Static IP + Elastic IP support
   ├── Preserves client source IP
   ├── TLS termination possible
   └── Use for: gaming, financial trading, IoT

3. Gateway Load Balancer (GWLB) — Layer 3 (IP packets)
   ├── For deploying third-party virtual appliances
   ├── Firewalls, intrusion detection systems
   ├── Traffic inspection and filtering
   └── Use for: enterprise security appliances

4. Classic Load Balancer (CLB) — Legacy, don't use
   ├── Layer 4 and Layer 7
   ├── Deprecated for new deployments
   └── Only for legacy applications
```

**Rule of thumb:** ALB for web apps, NLB for TCP/UDP, GWLB for security appliances.

## ALB Advanced Features

**Path-based routing:**

```
myapp.com/api/*  → API target group (Node.js instances)
myapp.com/admin/* → Admin target group (Django instances)
myapp.com/*      → Frontend target group (React instances)

ONE load balancer, MULTIPLE backend services
```

**Host-based routing:**

```
api.myapp.com     → API target group
admin.myapp.com   → Admin target group
www.myapp.com     → Frontend target group

ONE load balancer, MULTIPLE domains
```

**SSL Termination:**

- Upload SSL certificate to AWS Certificate Manager (ACM)
- ALB handles HTTPS decryption
- Traffic between ALB and instances is HTTP (internal network)
- Your app doesn't need to handle SSL — simpler

## Health Checks

**Configuration:**

- **Path**: `/health` or `/healthz` endpoint on your app
- **Protocol**: HTTP or HTTPS
- **Port**: usually same as app port (e.g., 5000)
- **Interval**: check every 30 seconds
- **Threshold**: fail 3 times before marking unhealthy

**Behavior:**

- Healthy instance → receives traffic
- Unhealthy → removed from rotation
- Becomes healthy again → automatically re-added

**Real world context:** Your app should implement a `/health` endpoint that:

- Returns 200 if the app is functional
- Returns 503 if a critical dependency (DB) is down
- Runs quickly (< 100ms) — the LB pings it frequently

## Target Groups

**Concept:** A target group is a collection of targets (EC2 instances, IP addresses, Lambda functions) that the ALB routes to.

```
ALB
    ├── Listener (port 443, HTTPS)
    │   ├── Rule 1: path=/api/* → api-target-group
    │   ├── Rule 2: path=/admin/* → admin-target-group
    │   └── Default: → frontend-target-group
    │
    ├── api-target-group
    │   ├── i-01234 (EC2 instance)
    │   ├── i-05678 (EC2 instance)
    │   └── i-09abc (EC2 instance)
    │
    └── frontend-target-group
        ├── i-def01
        └── i-def02
```

## Practical: Creating an ALB

```bash
# Steps (via console):
1. Set Up EC2 Instances (2+ instances with a web server, tagged for identification)
2. Configure Security Groups (allow HTTP 80 and SSH 22)
3. Create the Load Balancer (EC2 → Load Balancers → Create → ALB)
4. Register Targets (add instances to the target group, configure health check)
5. Test the Load Balancer (access the DNS name — should see traffic distribution)

# The ALB DNS looks like:
myapp-alb-1234567890.ap-south-1.elb.amazonaws.com
```

## Real World Production Setup

```
Route 53 (DNS)
    ↓  (myapp.com → ALB DNS)
Internet-facing ALB
    ↓  Listener 443 (HTTPS with ACM cert)
    ├── /api/*   → API target group (3 EC2 in ASG)
    ├── /admin/* → Admin target group (2 EC2 in ASG)
    └── /*       → Frontend target group (5 EC2 in ASG)

All in private subnets, only ALB in public subnet.
```

## Memory Hints

- **ALB** = smart traffic cop (understands HTTP routes)
- **NLB** = raw pipe (fast, dumb, transport layer)
- **GWLB** = security checkpoint (traffic inspection)
- **Target Group** = pool of servers waiting for work
- **Health Check** = pulse check ("are you alive?")

## Interview Q&A — ELB

**Q: When would you choose ALB vs NLB?**

> "ALB for HTTP/HTTPS web apps and APIs — anything that speaks the web. ALB operates at Layer 7, so it understands paths, headers, host names. This unlocks powerful features: path-based routing to send /api to one service and /admin to another, host-based routing for multi-tenant apps, WebSocket support for real-time features, and integration with Cognito for authentication.
> 
> NLB for anything that isn't HTTP or needs extreme performance. Layer 4 — just TCP/UDP with minimal overhead. Handles millions of requests per second with sub-millisecond latency. Use it for gaming servers, financial trading platforms, IoT gateways, or anywhere you need static IP addresses that don't change.
> 
> One more distinction: NLB preserves the client's source IP by default, ALB requires X-Forwarded-For header. For apps that need real client IP (rate limiting, geo-blocking), NLB is simpler.
> 
> Production example: web app on ALB with SSL termination and path routing, backend microservices communicating internally via NLB for speed."

**Q: Your ALB is returning 502 errors. How do you diagnose?**

> "502 Bad Gateway from ALB means the load balancer received a request but couldn't get a valid response from any backend instance.
> 
> Systematic diagnosis: First, check target group health — CloudWatch or console. If targets are unhealthy, that's the immediate problem. Check the health check path — is `/health` returning 200? Common cause: app crashed and can't respond, or app is running but the health endpoint has a bug.
> 
> Second, check application logs. `journalctl -u myapp` or CloudWatch Logs. Look for stack traces, connection errors, OOM kills.
> 
> Third, verify Security Groups. ALB's security group must allow inbound 443 from 0.0.0.0/0. Instance security group must allow inbound from the ALB security group on the app's port.
> 
> Fourth, check the app's response time. ALB has a default idle timeout of 60 seconds — if your app takes 90 seconds to respond, ALB gives up and returns 502. Fix: optimize slow queries or increase idle timeout.
> 
> Fifth, if it's intermittent, could be capacity — instances overwhelmed during traffic spike, Auto Scaling not keeping up. Scale up manually and investigate ASG metrics."

**Q: How does path-based routing work in ALB?**

> "You define listener rules that inspect the request path and route to different target groups.
> 
> Example: One ALB fronts a monolithic-looking domain but routes to multiple microservices behind the scenes.
> 
> - `/api/v1/*` → api-v1 target group (3 EC2 instances running Node.js)
> - `/api/v2/*` → api-v2 target group (5 ECS tasks running Python)
> - `/admin/*` → admin target group (2 EC2 instances running Django)
> - `/*` → frontend target group (S3 bucket via CloudFront, or React on EC2)
> 
> Rules are evaluated in priority order. First match wins. Default rule catches anything that doesn't match.
> 
> Real benefit: users see one domain, one SSL certificate, one endpoint. Behind the scenes you have multiple independent services, each scalable independently. This is API gateway pattern without needing API Gateway service.
> 
> Another use case: canary deployments. Route 10% of `/api/*` traffic to a new target group with the new version. Increase gradually. Rollback = update the rule to send 0%."

---

---

# 8. ASG — Auto Scaling Group

## Concept

**ASG (Auto Scaling Group)** is a service that automatically adds or removes EC2 instances based on demand.

**What it does:**

- Scale **up** when demand increases (more instances)
- Scale **down** when demand decreases (fewer instances)
- Maintain a **desired number** of healthy instances at all times
- Replace unhealthy instances automatically

## Key Functions

```
1. Automatic Scaling
   ├── Add/remove instances based on demand
   ├── Triggered by CloudWatch metrics (CPU, network, custom metrics)
   └── Cooldown period prevents flapping

2. Maintain Instance Health
   ├── Health checks (EC2 status or ELB target group)
   ├── Unhealthy instance → terminate → launch replacement
   └── Self-healing without human intervention

3. Scaling Policies
   ├── Target tracking (keep CPU at 60%)
   ├── Step scaling (add 2 when CPU > 70%, add 5 when CPU > 90%)
   ├── Simple scaling (add 1 when metric threshold breached)
   └── Predictive scaling (ML predicts future demand)

4. Ensure Availability
   ├── Minimum size: never fewer than X instances
   ├── Maximum size: never more than Y instances
   ├── Desired capacity: current target
   └── Handles instance failures automatically

5. Scheduled Scaling
   ├── Pre-configure scaling for known patterns
   ├── Example: scale up every weekday at 9AM (office hours traffic)
   └── Example: scale down every weekend

6. Multi-AZ Distribution
   ├── Spread instances across multiple AZs
   ├── AZ failure = only lose 1/N instances
   └── ASG automatically launches replacements in healthy AZs

7. Cost Optimization
   ├── Scale down during low demand = save money
   ├── Mix of On-Demand and Spot Instances
   └── Only pay for what you need
```

## Steps to Create ASG

```
1. Launch Template or Launch Configuration
   ├── Defines HOW instances are launched
   ├── AMI, instance type, security group, IAM role, user_data
   └── Reusable — one template, many ASGs

2. Create Auto Scaling Group
   ├── Select the Launch Template
   ├── Set min/max/desired instances
   └── Name the ASG

3. Select VPC and Subnets
   ├── Choose which subnets to launch in
   ├── Select multiple AZs for HA
   └── Public or private subnets based on architecture

4. Attach Load Balancer (Optional but common)
   ├── ASG registers instances with ALB/NLB automatically
   ├── ASG uses ELB health checks
   └── Enables health-based replacement

5. Configure Scaling Policies
   ├── Target tracking is the easiest
   ├── "Keep average CPU at 60%"
   └── ASG handles the rest

6. Health Checks
   ├── EC2 health checks (instance status)
   ├── ELB health checks (application-level)
   └── Grace period before checks start

7. Add Notifications (Optional)
   ├── SNS topic for scale up/down events
   └── Alerts your team, feeds monitoring

8. Review and Create
```

## Scaling Policies Deep Dive

**Target Tracking (recommended for most cases):**

```
Target: Average CPU utilization = 60%

If CPU rises above 60% → ASG adds instances until average drops to 60%
If CPU falls below 60% → ASG removes instances until average rises to 60%

Simple, self-tuning, no threshold math needed.
```

**Step Scaling:**

```
CPU 50-70%    → Add 1 instance
CPU 70-90%    → Add 2 instances
CPU 90%+      → Add 3 instances

More aggressive scaling for severe spikes.
```

**Scheduled Scaling:**

```
Every Monday 9AM       → Scale to 10 instances (office hours)
Every Monday 6PM       → Scale to 3 instances (evening)
Every Saturday morning → Scale to 2 instances (weekend traffic)

Predictable patterns handled without waiting for metrics to trigger.
```

## Real Production Patterns

**E-commerce site:**

```
Baseline:  5 instances (normal traffic)
Peak:      50 instances (Black Friday)
Overnight: 3 instances (low traffic)

ASG min: 3, max: 60, target CPU: 60%
Result: automatic scaling handles 10x traffic, saves 40% cost overnight
```

**Batch processing:**

```
Baseline:  1 instance (always available)
Job queue length increases:
    5 jobs   → 3 instances
    50 jobs  → 20 instances
    500 jobs → 100 instances (up to max)

Custom CloudWatch metric = queue depth
```

## ASG + ELB Integration

```
Client Request
    ↓
Route 53 (DNS)
    ↓
Application Load Balancer (target group)
    ↓
ASG (min=3, max=20)
    ├── Instance 1 (AZ-a) — healthy
    ├── Instance 2 (AZ-b) — healthy
    └── Instance 3 (AZ-c) — healthy
```

**Flow:**

1. ASG launches instances per Launch Template
2. Instances auto-register with ALB target group
3. ALB starts sending traffic once health check passes
4. If instance becomes unhealthy → ALB stops sending traffic → ASG terminates and replaces
5. New instance boots → registers with ALB → traffic resumes

## Memory Hints

- **ASG** = "smart HR that hires/fires workers based on workload"
- **Launch Template** = "job posting" (how new hires are onboarded)
- **Scaling Policy** = "hiring rules" (when to hire more or fewer)
- **Health Check** = "attendance check" (are workers showing up?)
- **Min/Max/Desired** = "workforce boundaries" (never fewer than X, never more than Y)

## Interview Q&A — ASG

**Q: Explain how ASG maintains availability during an AZ outage.**

> "ASG is designed for multi-AZ deployments. When you create an ASG, you specify subnets across multiple Availability Zones. ASG spreads instances evenly — if you have 6 instances and 3 AZs, it launches 2 per AZ.
> 
> When an AZ fails: ASG's EC2 health checks detect that instances in that AZ are unreachable. ASG terminates the failed instances and immediately launches replacements — but only in HEALTHY AZs. So if AZ-a fails, ASG launches 2 more instances split between AZ-b and AZ-c to maintain the desired count.
> 
> During the outage, you're temporarily running with 2/3 of your normal capacity in 2 AZs. If the ALB was multi-AZ and had cross-zone load balancing enabled, traffic automatically shifted away from AZ-a with no manual intervention.
> 
> Recovery: when AZ-a comes back, ASG rebalances by launching instances there and terminating some from AZ-b and AZ-c to restore even distribution.
> 
> Key takeaway: multi-AZ ASG plus ALB with cross-zone load balancing = automatic failover with zero downtime. This is why 'HA across AZs' is a core cloud principle."

**Q: What's the difference between Target Tracking and Step Scaling?**

> "Both are scaling policies but with different mental models.
> 
> Target Tracking is set-and-forget. You define a target metric like 'keep average CPU at 60%'. ASG continuously monitors and adjusts instance count to maintain that target. Uses two CloudWatch alarms internally (scale up, scale down) but you don't manage them. Self-tuning, handles most workloads well. Best for steady traffic patterns.
> 
> Step Scaling gives you granular control. You define specific thresholds and how many instances to add or remove at each threshold. Example: CPU 50-70% add 1, CPU 70-90% add 2, CPU 90%+ add 3. More responsive to severe spikes because you're explicitly saying 'when things get really bad, add a lot at once'.
> 
> When to use which: Target Tracking for 90% of cases — it's simpler and works well. Step Scaling when you know your traffic patterns are spiky and you want aggressive response, or for batch workloads where scaling behavior needs precise tuning.
> 
> Advanced option: Predictive Scaling uses machine learning on historical CloudWatch data to forecast future demand and pre-scale before the spike hits. Great for predictable patterns like daily/weekly cycles."

**Q: How do you prevent ASG from thrashing (constantly scaling up and down)?**

> "Three mechanisms.
> 
> First, cooldown periods. After a scaling action, ASG waits for the cooldown to expire before evaluating scale actions again. Default is 300 seconds. This gives time for the metric to stabilize after new instances start receiving traffic. Prevents adding 10 instances when 3 would have been enough.
> 
> Second, appropriate target thresholds. If you set CPU target at 90%, you'll be constantly at the edge of scaling. Better to target 60-70% — provides buffer for spikes without triggering unnecessary scale actions.
> 
> Third, instance warmup time. New instances need time to boot, register with LB, and start serving traffic. Configure the warmup period — during warmup, the new instance's metrics don't count toward scaling decisions. Prevents 'we just added an instance but CPU is still high, add another' loop.
> 
> Fourth, health check grace period. When a new instance launches, ASG waits before starting health checks. Prevents premature termination of instances that are still booting.
> 
> Fifth, use target tracking rather than step scaling for most cases — it has built-in stabilization logic.
> 
> If you're still seeing thrashing: your scaling metric might be too volatile. Try scaling on a smoothed metric (5-minute average) rather than instant values."

---

---

# 9. S3 — Simple Storage Service

## Concept

**Amazon S3** is an object storage service — stores files (called "objects") in containers (called "buckets"). Highly durable, highly available, essentially unlimited storage.

**Not a filesystem** — no mount, no folders (though key names can mimic paths).

## Key Facts

```
Data model:       Object storage (not block, not file)
Bucket name:      Globally unique (no two accounts can have same name)
Region:           Buckets are region-specific
Object model:     Key-value pair (key = path, value = content)
Max object size:  5 TB single object
Multipart upload: Required for objects > 100MB, recommended > 5MB
Durability:       99.999999999% (11 nines) — practically indestructible
Availability:     99.99% (Standard class)
```

## Storage Types Comparison

```
Object Storage (S3)
    ├── Access via HTTP/HTTPS API (GET, PUT, DELETE)
    ├── No filesystem, no mount
    ├── Metadata + unique key + content
    └── Best for: unstructured data, backups, media, logs

Block Storage (EBS)
    ├── Raw disk blocks
    ├── Mounted as filesystem
    ├── Attached to one instance
    └── Best for: OS drives, databases

File Storage (EFS)
    ├── Shared filesystem
    ├── Multiple instances mount it simultaneously
    └── Best for: shared config, ML training data
```

## Common Use Cases

**Web Hosting:**

- Host static websites (HTML, CSS, JS)
- Combine with CloudFront for global CDN

**Data Lake:**

- Central repository for structured and unstructured data
- Used with Athena for SQL queries, Glue for ETL

**Backup and Disaster Recovery:**

- Store backups with 11 nines durability
- Combine with lifecycle policies for cost optimization
- Cross-region replication for DR

**Application Data:**

- User uploads (images, videos, documents)
- App logs (structured JSON logs archived to S3)
- ML training data

## S3 Versioning

**What it does:** Keeps every version of every object in the bucket.

```
Without versioning:
    Upload file.txt v1 → stored
    Upload file.txt v2 → v1 GONE, replaced

With versioning:
    Upload file.txt v1 → stored with version ID
    Upload file.txt v2 → BOTH versions exist
    Delete file.txt    → delete marker added, file not truly gone
                         Restore by removing delete marker
```

**Why enable:**

- Accidental deletion recovery
- Rollback to previous version
- Ransomware protection
- Audit trail

**Cost consideration:** Versioning multiplies storage. Use lifecycle policies to expire old versions.

```bash
# Enable versioning
aws s3api put-bucket-versioning \
  --bucket my-bucket \
  --versioning-configuration Status=Enabled

# List versions of an object
aws s3api list-object-versions --bucket my-bucket --prefix file.txt

# Get a specific version
aws s3api get-object --bucket my-bucket --key file.txt --version-id abc123 file.txt
```

## S3 Replication

**Two types:**

```
Same-Region Replication (SRR)
    ├── Copy objects to another bucket in same region
    ├── Use for: compliance, data segregation, log aggregation
    └── Example: replicate production logs to dedicated analytics account

Cross-Region Replication (CRR)
    ├── Copy objects to bucket in different region
    ├── Use for: disaster recovery, geo-redundancy, latency reduction
    └── Example: primary bucket in ap-south-1, replicated to us-east-1
```

**Requirements:**

- Versioning must be enabled on BOTH buckets
- Objects uploaded AFTER enabling replication are copied (existing objects need manual sync)
- IAM role for S3 to perform replication

## Data Encryption

**Two levels:**

**Encryption at Rest:**

```
SSE-S3 (AWS-managed keys)
    ├── S3 manages encryption keys
    ├── Simplest, most common
    └── No extra cost

SSE-KMS (KMS-managed keys)
    ├── Uses AWS KMS
    ├── More control, audit trail
    ├── Cost: KMS API calls
    └── Best for: compliance requirements

SSE-C (Customer-provided keys)
    ├── You provide the key on each request
    ├── AWS doesn't store the key
    └── Rare — most teams use SSE-KMS

Client-Side Encryption
    ├── Encrypt before uploading to S3
    ├── AWS never sees plaintext
    └── Highest security, most complex
```

**Encryption in Transit:**

- Enforce HTTPS via bucket policy
- SSL/TLS for all uploads and downloads

**Best practice:** Enable SSE-KMS by default on all buckets.

## S3 Bucket Policies

**JSON-based access control attached directly to a bucket.**

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "PublicReadOnly",
      "Effect": "Allow",
      "Principal": "*",
      "Action": "s3:GetObject",
      "Resource": "arn:aws:s3:::my-website-bucket/*"
    },
    {
      "Sid": "DenyUnencryptedUploads",
      "Effect": "Deny",
      "Principal": "*",
      "Action": "s3:PutObject",
      "Resource": "arn:aws:s3:::my-website-bucket/*",
      "Condition": {
        "StringNotEquals": {
          "s3:x-amz-server-side-encryption": "AES256"
        }
      }
    }
  ]
}
```

**Common actions:**

- `s3:GetObject` — download/read
- `s3:PutObject` — upload/write
- `s3:DeleteObject` — delete
- `s3:ListBucket` — list contents

## S3 Storage Classes

```
Standard
    ├── Frequently accessed data
    ├── Millisecond retrieval
    ├── 99.99% availability
    └── Highest cost per GB

Intelligent-Tiering
    ├── Auto-moves objects between tiers based on access
    ├── Best for unknown access patterns
    ├── Small monitoring fee per object
    └── No retrieval fees

Standard-IA (Infrequent Access)
    ├── Less frequently accessed but rapid retrieval when needed
    ├── Lower storage cost, retrieval fee
    ├── 99.9% availability
    └── Minimum 30-day storage

One Zone-IA
    ├── Same as IA but single AZ (less resilient)
    ├── 20% cheaper than Standard-IA
    └── OK if you can regenerate the data

Glacier Instant Retrieval
    ├── Archives with millisecond retrieval
    ├── Lower storage cost than IA
    └── Minimum 90-day storage

Glacier Flexible Retrieval (formerly Glacier)
    ├── Retrieval: minutes to hours
    ├── Very low storage cost
    └── Minimum 90-day storage

Glacier Deep Archive
    ├── Retrieval: 12+ hours
    ├── Cheapest — < $1 per TB per month
    └── Minimum 180-day storage
```

**Decision matrix:**

```
Accessed daily      → Standard
Accessed weekly     → Intelligent-Tiering
Accessed monthly    → Standard-IA
Accessed yearly     → Glacier Instant Retrieval
Accessed rarely     → Glacier Flexible
Almost never        → Glacier Deep Archive
```

## S3 Lifecycle Policies

**Automate object management** — move to cheaper storage as data ages, delete when not needed.

**Transition Actions:**

```
Day 0     → Standard (upload)
Day 30    → Standard-IA (transition)
Day 90    → Glacier Flexible (transition)
Day 365   → Glacier Deep Archive (transition)
```

**Expiration Actions:**

```
Day 365       → Delete objects
Day 30 (version) → Delete old versions after 30 days
```

**Example policy:**

```json
{
  "Rules": [{
    "Id": "ArchiveOldLogs",
    "Status": "Enabled",
    "Filter": {"Prefix": "logs/"},
    "Transitions": [
      {"Days": 30, "StorageClass": "STANDARD_IA"},
      {"Days": 90, "StorageClass": "GLACIER"}
    ],
    "Expiration": {"Days": 365}
  }]
}
```

**Real production impact:** Storing 100TB of logs

- All in Standard: $2,300/month
- With lifecycle to Glacier after 90 days: $200/month
- Same data, 92% cost reduction

## Hosting Static Website

**Full workflow:**

```
1. Create S3 bucket (name = domain e.g. www.mysite.com)
2. Upload website files (index.html, styles.css, script.js)
3. Enable Static Website Hosting
   ├── Properties → Static Website Hosting → Enable
   ├── Index document: index.html
   └── Error document: error.html
4. Disable "Block Public Access"
5. Add bucket policy for public read access
6. (Optional) Attach CloudFront for CDN + HTTPS
7. (Optional) Route 53 alias record to point domain to S3
```

**Real world:** Most static sites use S3 + CloudFront + Route 53. S3 alone is HTTP-only — you need CloudFront for HTTPS.

## S3 Snow Family

**Physical devices for moving huge amounts of data to AWS when internet upload is impractical.**

```
AWS Snowcone (small)
    ├── 8 TB storage
    ├── Portable, ruggedized
    └── Use: field data collection, edge computing

AWS Snowball Edge (medium)
    ├── 80 TB storage
    ├── Also does edge compute
    └── Use: moving petabytes to AWS

AWS Snowmobile (massive — truck!)
    ├── Up to 100 PB per truck
    ├── Literal shipping container on a truck
    └── Use: moving entire data centers to cloud
```

**When to use:** If uploading via internet would take weeks or months, order a Snow device. AWS ships it, you fill it, AWS ships it back and uploads to S3.

## S3 Storage Gateway

**Hybrid cloud storage:** Connects on-premises environments to S3 in the cloud.

**Three types:**

```
File Gateway
    ├── NFS/SMB mount from on-prem servers
    ├── Files stored in S3 behind the scenes
    └── Use: seamless cloud backup for existing file servers

Volume Gateway
    ├── iSCSI block storage from on-prem
    ├── Backed by S3, snapshots to EBS
    └── Use: cloud-backed block storage

Tape Gateway
    ├── Virtual tape library, backed by S3 and Glacier
    └── Use: replacing physical tape backup systems
```

## Practical Commands

```bash
# List all buckets
aws s3 ls

# List contents of a bucket
aws s3 ls s3://my-bucket/

# Recursive list
aws s3 ls s3://my-bucket/ --recursive

# Upload file
aws s3 cp file.txt s3://my-bucket/

# Download file
aws s3 cp s3://my-bucket/file.txt ./

# Sync local dir to S3 (like rsync)
aws s3 sync ./dist s3://my-bucket/

# Sync S3 to local
aws s3 sync s3://my-bucket/ ./backup/

# Delete file
aws s3 rm s3://my-bucket/file.txt

# Delete bucket and all contents (dangerous!)
aws s3 rb s3://my-bucket --force

# Set bucket-wide encryption
aws s3api put-bucket-encryption --bucket my-bucket \
  --server-side-encryption-configuration '{
    "Rules": [{
      "ApplyServerSideEncryptionByDefault": {
        "SSEAlgorithm": "AES256"
      }
    }]
  }'
```

## Real Production Setup

**Terraform state bucket:**

```
Bucket: mycompany-terraform-state
    ├── Versioning: ENABLED (rollback state)
    ├── Encryption: SSE-KMS
    ├── Bucket policy: only specific IAM roles allowed
    ├── Access logging: enabled
    └── Lifecycle: expire old versions after 90 days
```

**Application logs:**

```
Bucket: mycompany-app-logs
    ├── App writes JSON logs to S3 every hour
    ├── Prefix: /year/month/day/hour/
    ├── Standard class initially
    ├── Lifecycle: Standard-IA after 30 days, Glacier after 90 days
    └── Delete after 2 years
```

## Memory Hints

- **S3** = infinite HDD in the sky
- **Bucket** = folder at the top level (globally named)
- **Object** = a file (with key + metadata + content)
- **Versioning** = undo button for the whole bucket
- **Lifecycle** = automatic file aging (move to cold storage as it gets old)
- **Storage Classes** = different pricing tiers based on access pattern

## Interview Q&A — S3

**Q: How do you decide which S3 storage class to use?**

> "Two factors: access frequency and cost tolerance for retrieval.
> 
> For frequently accessed data — anything hit multiple times per month per object — use Standard. It's the most expensive per GB but no retrieval fees. Application uploads, active user data, current logs.
> 
> For infrequently accessed data — hit once a month or less — use Standard-IA. About 45% cheaper storage than Standard, but you pay a small retrieval fee per GB downloaded. Good for logs older than 30 days that you occasionally need.
> 
> For archives — data you might need but rarely — use Glacier variants. Glacier Instant Retrieval keeps millisecond access but at cheaper storage. Glacier Flexible has minutes-to-hours retrieval and is much cheaper. Glacier Deep Archive is the cheapest at under $1 per TB per month, but retrieval takes 12+ hours.
> 
> Best practice: don't manually decide. Use lifecycle policies. Objects start in Standard, transition to Standard-IA after 30 days, Glacier after 90, Deep Archive after 365. Delete after retention period. Automated cost optimization.
> 
> One more option: Intelligent-Tiering. AWS monitors access patterns and moves objects automatically. Costs a small monitoring fee per object but eliminates guesswork. Best when access patterns are unpredictable."

**Q: Explain S3 versioning and when you'd enable it.**

> "Versioning keeps every version of every object. Without versioning, uploading a file with the same key overwrites the previous version — the old one is gone. With versioning, both versions exist, each with a unique version ID.
> 
> Even deletion doesn't truly delete — S3 adds a delete marker. The object versions still exist, and you can restore by removing the delete marker.
> 
> When to enable: First, on Terraform state buckets — absolutely critical. Corrupted state file can be rolled back to previous version. Never lose infrastructure state. Second, on application configuration buckets — accidental config push can be reverted instantly. Third, on any bucket with critical data where accidental deletion would be catastrophic.
> 
> Cost consideration: every version consumes storage. A 1GB file overwritten 100 times = 100GB stored. Combine versioning with lifecycle policy: expire old versions after 30-90 days depending on retention needs.
> 
> Ransomware protection is a modern reason. If attacker encrypts your S3 objects, previous versions are still there. MFA Delete adds another layer — even you can't permanently delete versions without MFA."

**Q: How do you host a static website on S3? What are the limitations?**

> "The workflow: create bucket, upload HTML/CSS/JS files, enable static website hosting in bucket properties, disable Block Public Access, apply a bucket policy that grants s3:GetObject to everyone. The bucket exposes a URL like `mybucket.s3-website.ap-south-1.amazonaws.com`.
> 
> Limitations of S3 alone: HTTP only — no HTTPS support directly from S3 website endpoint. This is a dealbreaker for most modern sites. No custom SSL certificates. Regional endpoint — users far from the bucket's region see slower load times. No advanced features like caching headers control, WAF integration, or edge computing.
> 
> Production pattern: S3 + CloudFront + Route 53. S3 hosts the files with public access disabled (only CloudFront can access). CloudFront serves the content globally via CDN with HTTPS. ACM provides free SSL certificate. Route 53 provides the domain and routes users to CloudFront.
> 
> This gives you: HTTPS, global CDN, custom domain, DDoS protection, edge caching, WAF integration. Same static S3 files, production-ready delivery. Cost is minimal — CloudFront's free tier covers a lot, and S3 storage for a static site is often < $1/month."

**Q: How do you protect an S3 bucket from public exposure?**

> "Multiple layers of defense — never rely on just one.
> 
> Layer 1: Block Public Access at the account level. Since 2023, AWS blocks public access by default for new buckets. This is the safety net that catches accidental public settings.
> 
> Layer 2: Bucket policies. Don't include statements with `Principal: '*'` unless intentional for a public website. Use IAM users, roles, or specific AWS accounts as principals.
> 
> Layer 3: ACLs. Legacy access control — most teams disable ACLs entirely now with 'Bucket owner enforced' setting.
> 
> Layer 4: Encryption. Enable default SSE-KMS encryption. Even if data is exposed, it's encrypted at rest.
> 
> Layer 5: Access logging. Enable S3 access logs to another bucket. Every request is logged — you can audit who accessed what.
> 
> Layer 6: AWS Config rules. Set up automated compliance checks that flag any bucket that becomes public or lacks encryption.
> 
> Real world scanning: AWS Trusted Advisor and third-party tools like Prowler scan for misconfigurations. Every quarter, run a full audit — buckets with public access, encryption disabled, cross-account access. Fix findings immediately.
> 
> Most-cited S3 data leaks in the news were misconfigured bucket policies. Always assume attackers are scanning for open buckets — they are."

---

---

# 10. RDS — Relational Database Service

## Concept

**AWS RDS** is a **managed relational database service** that handles administrative tasks: backups, patching, monitoring, scaling.

**What "managed" means:**

- You don't SSH into the DB server
- AWS handles OS updates, DB engine patches
- Automated backups
- Multi-AZ failover
- Read replicas
- Monitoring

You just define: engine, instance size, storage, and connection settings. AWS does the rest.

## Supported Database Engines

```
Amazon Aurora    → AWS-native MySQL/PostgreSQL compatible (fastest)
MySQL            → Community MySQL, managed
PostgreSQL       → Community PostgreSQL, managed
MariaDB          → MySQL fork, managed
Oracle           → Enterprise DB, license required
SQL Server       → Microsoft, license required or BYOL
```

## Amazon Aurora — The Star

Aurora is AWS's proprietary database engine, MySQL-and-PostgreSQL-compatible.

```
Aurora offers:
  ├── Up to 5x throughput of MySQL Community Edition
  ├── Up to 3x throughput of PostgreSQL
  ├── Up to 128 TB of autoscaling SSD storage
  ├── Six-way replication across three AZs (2 copies per AZ)
  ├── Up to 15 read replicas with < 10ms lag
  └── Automatic monitoring with failover
```

**Why so fast:** Aurora separates compute from storage. Storage is a distributed, self-healing service across multiple AZs. Compute instances are stateless — can be replaced instantly without data loss.

## Key Benefits of RDS

```
High Availability
    ├── Multi-AZ deployments with synchronous replication
    └── Automatic failover in ~60 seconds

Scaling
    ├── Vertical: change instance type (t3.medium → r5.large)
    ├── Horizontal: add read replicas for read-heavy workloads
    └── Storage: auto-scaling storage

Automated Backups
    ├── Daily snapshots (default retention 7 days, max 35)
    ├── Continuous transaction logs — point-in-time recovery
    └── Can restore to any second in the retention window

Read Replicas
    ├── Async replication to separate instances
    ├── Offload read traffic from primary
    └── Can be in different regions

Multi-AZ for Disaster Recovery
    ├── Synchronous standby in different AZ
    ├── Automatic failover
    └── No data loss

Cost-Effectiveness
    ├── Cheaper than self-managed DBs on EC2
    ├── No DBA needed for routine ops
    └── Free tier available
```

## Multi-AZ vs Read Replicas

**These solve DIFFERENT problems — critical interview topic.**

```
Multi-AZ (High Availability)
    ├── Synchronous replication to standby in another AZ
    ├── Standby does NOT serve traffic — passive backup
    ├── Automatic failover on primary failure (~60 seconds)
    ├── DNS endpoint updates automatically
    └── PURPOSE: minimize downtime, disaster recovery

Read Replicas (Read Scaling)
    ├── Asynchronous replication to separate instances
    ├── Replicas DO serve read traffic
    ├── Can be in same region or cross-region
    ├── Manual promotion to primary if needed
    └── PURPOSE: scale read performance
```

**Real world:** Use both together.

- Multi-AZ for the primary → automatic failover
- Read replicas for scaling read queries → analytics, reports, dashboards

## Practical Setup

**Scenario:** EC2 running Node.js app, connecting to RDS MySQL.

**Steps:**

```
1. Create RDS MySQL Instance
   ├── Use Free Tier (db.t3.micro)
   ├── Username: admin, set password (no special chars)
   ├── Public access: True (for lab access; NEVER in production)
   ├── Security group: allow port 3306 from EC2 SG
   └── After creation: note the ENDPOINT (hostname)

2. Create EC2 Instance
   ├── Amazon Linux 2
   ├── Install Docker
   └── Pull Node.js app image

3. Run App with DB Credentials as Env Vars
   docker run --rm -p 80:3000 \
     -e DB_HOST="myapp-db.xxxxx.ap-south-1.rds.amazonaws.com" \
     -e DB_USER="admin" \
     -e DB_PASSWORD="secretpassword" \
     -d philippaul/node-mysql-app:02

4. Verify Connection
   docker run -it --rm mysql:8.0 mysql -h myapp-db.xxxxx.rds.amazonaws.com -u admin -p
```

## Production Setup

**Never** make RDS publicly accessible in production. Correct setup:

```
Public Subnets:
    ├── ALB (internet-facing)
    └── NAT Gateway

Private App Subnets:
    ├── EC2 instances (behind ALB)
    └── Connect to RDS via private IP

Private DB Subnets:
    ├── RDS Multi-AZ (primary in AZ-a, standby in AZ-b)
    ├── Security group: only allow 3306 from App SG
    └── NOT reachable from internet
```

## RDS Storage Auto-Scaling

**Setting:** Storage auto-scaling threshold — e.g., 20% free space.

**Behavior:** When free space drops below 20%, RDS automatically increases storage (up to max limit you set). No downtime. Prevents "database full" incidents.

## Common Use Cases

**Web Applications:** Structured data with ACID transactions — user profiles, orders, sessions. **E-commerce:** Inventory, customer data, orders — need strong consistency. **Business Applications:** ERP, CRM, finance apps — need data integrity.

## Memory Hints

- **RDS** = "database as a service" (AWS handles the boring stuff)
- **Aurora** = "the Ferrari version" (AWS-optimized MySQL/PostgreSQL)
- **Multi-AZ** = "hot standby in another AZ" (for HA, doesn't serve traffic)
- **Read Replica** = "clone that answers questions" (for read scaling)
- **Endpoint** = the DNS name that points to your DB (survives failover)

## Interview Q&A — RDS

**Q: What's the difference between Multi-AZ and Read Replicas?**

> "Both involve replication but solve different problems.
> 
> Multi-AZ is for high availability. AWS creates a synchronous standby replica in a different AZ. The standby doesn't serve traffic — it's a hot backup. If the primary fails, RDS automatically fails over to the standby within about 60 seconds. Your app doesn't need to change anything because the RDS endpoint stays the same — it just points to the new primary. Purpose: minimize downtime from AZ failures, hardware failures, or planned maintenance.
> 
> Read Replicas are for read scaling. AWS creates asynchronous replicas that DO serve read queries. Your app routes writes to the primary and reads to the replicas. This offloads the primary and improves overall throughput. Purpose: handle read-heavy workloads like analytics, reports, or high-traffic reads. Can be in same region for latency, or cross-region for geographic distribution.
> 
> In production I use both together. Multi-AZ for the primary provides automatic disaster recovery. Two or three read replicas handle report queries and heavy read traffic without impacting transactional performance on the primary."

**Q: When would you choose Aurora over RDS MySQL?**

> "Aurora when performance, availability, and scale matter more than cost. Aurora offers up to 5x MySQL throughput, storage that auto-scales to 128TB with 6-way replication across 3 AZs, 15 read replicas with sub-10ms lag. It's the gold standard for AWS databases.
> 
> RDS MySQL when you need compatibility with community MySQL features or exact version matching for a legacy app. Also when cost is a major constraint — Aurora is more expensive than RDS MySQL.
> 
> For most modern applications building on AWS, Aurora is the default choice. The performance and availability benefits justify the cost difference. You get built-in features that would take significant engineering effort to replicate on RDS MySQL or self-managed.
> 
> Aurora Serverless is another consideration — Aurora that scales compute automatically based on load, including scaling to zero when idle. Great for dev/test environments or unpredictable workloads."

**Q: How do you handle RDS backups and point-in-time recovery?**

> "RDS provides two backup mechanisms.
> 
> Automated Backups: daily snapshots plus continuous transaction logs. Retention 1-35 days configurable. This enables point-in-time recovery — you can restore to any second within the retention window. If someone runs an `UPDATE` without a `WHERE` clause at 2:47 PM, you restore to 2:46 PM.
> 
> Manual Snapshots: on-demand snapshots you take before major changes. These don't expire — persist until you delete them. Perfect for pre-migration checkpoints or long-term compliance archives.
> 
> Restore process: you don't restore in-place. RDS creates a NEW database from the backup. Your app connects to the new endpoint. This preserves the original for further investigation.
> 
> Cross-region: automated backups don't automatically cross regions. For DR, either configure cross-region snapshot copy, or use Aurora's global database feature which replicates to another region with under 1 second lag.
> 
> Production practice: automated backups with 7-day retention plus weekly manual snapshots for a rolling 90-day archive. Test restore process quarterly — untested backups aren't backups."

---

---

# 11. DynamoDB — NoSQL

## Concept

**DynamoDB** is AWS's managed **NoSQL** database — designed for massive scale, single-digit millisecond latency, and serverless operation.

**NoSQL means:** flexible schema, non-tabular data structure, ideal for unstructured or semi-structured data.

## Data Model

```
Data stored as items in tables
Each item = JSON-like document (key-value pairs)

Example table: users
    ├── Item 1: {name: "Raju", age: 25}
    ├── Item 2: {name: "Sham", age: 28}
    └── Item 3: {name: "Baburao", age: 45, city: "Mumbai"}

Notice: Item 3 has a 'city' field that others don't have.
That's schema flexibility — no fixed columns like SQL.
```

## Key Concepts

```
Table
    ├── Container for items (like a "table" in SQL, but flexible schema)
    └── Requires primary key definition

Primary Key
    ├── Simple: single attribute (e.g., user_id)
    ├── Composite: partition key + sort key (e.g., user_id + timestamp)
    └── Uniquely identifies each item

Item
    ├── One record in the table
    ├── Collection of attributes
    └── Max 400KB per item

Attribute
    ├── A field on an item
    └── Types: String, Number, Binary, Boolean, List, Map, Set
```

## Capacity Modes

```
On-Demand (Pay-per-request)
    ├── No capacity planning needed
    ├── Instantly scales to any load
    ├── Pay for actual reads/writes
    └── Best for: variable/unpredictable workloads

Provisioned Capacity
    ├── Pre-allocate Read/Write Capacity Units (RCU/WCU)
    ├── Auto-scaling available
    ├── Cheaper for predictable workloads
    └── Best for: steady, predictable traffic
```

**Free tier:** 25 GB storage + 25 RCU + 25 WCU (handles ~200 million requests/month).

## Serverless Benefits

```
No Server Management
    ├── No provisioning, no OS, no patching
    └── AWS handles everything

Automatic Scaling
    ├── Scales instantly based on demand
    └── Handles traffic spikes without config

Zero Downtime
    ├── No maintenance windows
    └── Continuous availability

Pay-as-You-Go
    ├── Only pay for actual usage
    └── Scales to zero when idle (no charge)

Idle Cost Savings
    └── Empty tables cost nothing
```

## Use Cases

**Ideal for:**

- Mobile applications (real-time sync)
- Web apps with high traffic
- Gaming (leaderboards, player state)
- IoT (sensor data)
- Ad tech (user profiles, tracking)
- Real-time analytics
- Session storage

**Not ideal for:**

- Complex JOINs (NoSQL doesn't do JOINs well)
- Ad-hoc queries on non-key attributes (need to design keys correctly)
- Strong ACID transactions across many tables

## DynamoDB Accelerator (DAX)

**In-memory caching layer for DynamoDB.**

```
DAX offers:
    ├── Microsecond latency (10x faster than DynamoDB)
    ├── Fully managed cache
    ├── Deployed in multiple AZs for HA
    └── Transparent to your app (same API)

Only for DynamoDB (not other databases like ElastiCache is)
```

**When to use:** Read-heavy workloads where you're hitting DynamoDB's already-fast performance limits.

## DynamoDB Global Tables

**Multi-region replication with eventual consistency.**

```
Region 1 (Mumbai)  ←→  Region 2 (US-East)  ←→  Region 3 (EU-West)

Writes in any region replicate to all others.
Users read/write in nearest region for low latency.
```

**Use case:** Global apps requiring low latency worldwide.

## Practical Setup

**Scenario:** EC2 with Docker running a Node.js app that reads/writes to DynamoDB.

**Steps:**

```
1. Create DynamoDB Table
   Name: 'Contacts'
   Partition key: 'id' (Type: String)
   Capacity: On-Demand or Provisioned

2. Create EC2 Instance
   Install Docker, pull app image

3. Create IAM Access Keys
   Access Key + Secret Key for app authentication
   BETTER: use IAM Role attached to EC2 (no keys!)

4. Run Container with Credentials
   docker run --rm -d -p 80:3000 --name node-dynamo-app \
     -e AWS_REGION=ap-south-1 \
     -e AWS_ACCESS_KEY_ID=xxx \
     -e AWS_SECRET_ACCESS_KEY=yyy \
     philippaul/node-dynamodb-demo
```

**Production version (no access keys):**

```
Attach IAM Role to EC2 with DynamoDB permissions
App uses AWS SDK — auto-detects role credentials
No AWS_ACCESS_KEY_ID or AWS_SECRET_ACCESS_KEY in env
```

## DynamoDB vs RDS — Quick Comparison

```
Feature           RDS                    DynamoDB
─────────────────────────────────────────────────────────
Type              Relational (SQL)       NoSQL (Key-value)
Schema            Fixed, rigid           Flexible
Scaling           Vertical + read replicas Horizontal (auto)
JOINs             Yes                    No (denormalize)
ACID              Full                   Limited (single item)
Complexity        More setup             Fully managed
Best for          Structured data,       Unstructured, high scale
                  complex queries        real-time
Latency           Millisecond            Single-digit ms
                                         (DAX: microsecond)
```

## Memory Hints

- **DynamoDB** = "NoSQL on autopilot"
- **Primary Key** = "unique fingerprint for each item"
- **On-Demand** = "pay only when used" (like Uber pricing)
- **Provisioned** = "reserved parking spot" (pay for allocated capacity)
- **DAX** = "turbo boost for DynamoDB"
- **Global Tables** = "same data everywhere in the world"

## Interview Q&A — DynamoDB

**Q: When would you use DynamoDB over RDS?**

> "DynamoDB when you need massive scale, unpredictable traffic, or don't want to manage anything.
> 
> Perfect use cases: mobile app backends where user count can spike from thousands to millions overnight. Gaming leaderboards with millions of concurrent players. IoT platforms ingesting sensor data at high volume. Session storage for web apps. Real-time analytics dashboards.
> 
> RDS when you have structured, relational data with complex queries — orders, customers, products with foreign key relationships. Financial applications requiring full ACID transactions across multiple tables. Legacy apps designed for SQL.
> 
> Key trade-off: DynamoDB requires careful key design upfront. You can't do ad-hoc queries like 'find all orders over $100 in the last month' unless you designed your keys for that query. RDS is more flexible for evolving query patterns.
> 
> Modern approach: use both. DynamoDB for high-volume operational data (sessions, real-time state), RDS for structured business data (orders, products, users). Some architectures use DynamoDB streams to sync data into RDS or Redshift for analytics."

**Q: Explain DynamoDB's capacity modes.**

> "Two modes: On-Demand and Provisioned.
> 
> On-Demand: pay per read and write request. No capacity planning. DynamoDB scales instantly to any load. Best for new applications, unpredictable traffic, or dev/test environments. Downside: more expensive per request than provisioned.
> 
> Provisioned: you pre-allocate Read Capacity Units and Write Capacity Units. DynamoDB reserves that capacity. Cheaper if your traffic is predictable and steady. Comes with Auto Scaling — DynamoDB automatically adjusts provisioned capacity based on utilization. Good for production workloads with known baselines.
> 
> Decision framework: start with On-Demand for new apps to avoid over/under-provisioning. Monitor for 2-4 weeks to understand patterns. Switch to Provisioned with Auto Scaling once you know your baseline — typically 40-60% cheaper for steady workloads.
> 
> You can switch modes every 24 hours if patterns change."

---

---

# 12. Lambda — Serverless Compute

## Concept

**AWS Lambda** is a serverless computing service — run code in response to events without managing servers.

**You upload code. AWS handles:**

- Server provisioning
- Scaling (from 0 to thousands of concurrent executions)
- Availability
- OS patching
- Monitoring

**You pay only for:**

- Number of executions
- Duration of each execution (in milliseconds)

**No charge when idle.** Empty Lambda = $0/month.

## When to Use Lambda

**Perfect for event-driven, short-lived tasks:**

```
1. Image Processing
   User uploads image → Lambda triggered → resizes/compresses → saves back to S3

2. Data Transformation
   Data arrives in S3 → Lambda cleans it → writes to DynamoDB/RDS

3. Real-Time Notifications
   User signs up → Lambda sends welcome email via SES

4. Scheduled Tasks (Cron-like)
   Every hour → CloudWatch Events triggers Lambda → runs cleanup script

5. API Backend
   API Gateway → Lambda → returns response
   (Serverless REST APIs)

6. Stream Processing
   Kinesis stream → Lambda processes each record → stores results

7. Chatbots
   Slack/Teams webhook → Lambda handles command → posts response
```

## Event-Driven Execution

**Lambda runs in response to triggers.** Common event sources:

```
S3               → File uploads, deletions
DynamoDB Streams → Table changes (inserts, updates, deletes)
API Gateway      → HTTP requests
CloudWatch       → Scheduled events (cron), alarms
SNS              → Message published to topic
SQS              → Message in queue
Kinesis          → Stream records
EventBridge      → Custom events, third-party integrations
CodeCommit       → Git push
Cognito          → User signup, login
```

## Automatic Scaling

- **1 request:** 1 Lambda instance runs
- **1,000 concurrent requests:** 1,000 Lambda instances run in parallel
- **No configuration needed** — AWS handles it

**Concurrency limit:** 1,000 concurrent executions per region by default (can request increase).

## Pay-as-You-Go Pricing

**Billed on:**

- **Requests:** $0.20 per 1 million requests (first 1M free per month)
- **Duration:** $0.0000166667 per GB-second

**Example:** Lambda with 512MB memory, runs for 200ms, called 1 million times/month:

- Requests: $0.20
- Duration: 1M × 0.5GB × 0.2s × $0.0000167 = ~$1.67
- **Total: ~$1.87/month for 1 million executions**

**Comparison:** Same workload on EC2 (t3.small running 24/7) = ~$15/month + you manage the server.

## Supported Languages

```
Node.js (JavaScript)
Python
Java
C# (.NET Core)
Go
Ruby
Custom Runtime API (bring any language via container image)
```

**Most popular:** Python and Node.js for their fast cold start and rich ecosystem.

## Lambda Limits

```
Memory:           128 MB to 10,240 MB (10 GB)
CPU:              Scales with memory (proportionally)
Timeout:          15 minutes maximum
Deployment size:  50 MB zipped, 250 MB unzipped
Container image:  10 GB
Concurrent execs: 1,000 default (soft limit)
Temp storage:     512 MB (/tmp) — expandable to 10 GB
```

## Cold Start vs Warm Start

**Cold Start:** First invocation of a Lambda (or after idle period). AWS spins up a new container. Takes 100ms - 2 seconds depending on runtime.

**Warm Start:** Reusing an existing container. Sub-millisecond overhead.

**Mitigation for critical low-latency apps:**

- **Provisioned Concurrency**: keep N containers always warm (extra cost)
- **Language choice**: Python and Node.js have faster cold starts than Java or .NET

## Real Production Use Cases

**Image Processing Pipeline:**

```
User uploads image to S3 bucket "raw-images"
    ↓
S3 event triggers Lambda "image-processor"
    ↓
Lambda:
    ├── Reads image from S3
    ├── Resizes to multiple sizes (thumbnail, medium, large)
    ├── Compresses
    ├── Applies watermark
    └── Uploads results to S3 bucket "processed-images"
    ↓
DynamoDB record updated with URLs
    ↓
SNS notification sent to user
```

**Serverless API:**

```
Client HTTP request
    ↓
API Gateway (REST or HTTP API)
    ↓
Lambda function (handles route)
    ↓
DynamoDB (data storage)
    ↓
Response back to client

Zero servers to manage. Scales from 1 to millions of requests.
```

**Scheduled Cleanup:**

```
CloudWatch Events (every day at 2AM)
    ↓
Lambda "cleanup-old-data"
    ↓
Scans DynamoDB for records older than 90 days
    ↓
Archives to S3 Glacier
    ↓
Deletes from DynamoDB
```

## Memory Hints

- **Lambda** = "code that runs when something happens"
- **Serverless** = "no server for YOU to manage" (AWS still uses servers)
- **Cold Start** = "first-time wake up" (small delay)
- **Concurrency** = "how many can run at the same time"

## Interview Q&A — Lambda

**Q: When would you choose Lambda over EC2 or containers?**

> "Lambda is ideal for event-driven, short-lived, sporadic workloads. Three main scenarios:
> 
> First, event processing: S3 uploads, DynamoDB stream changes, API calls, scheduled tasks. Lambda's event integration is native — no polling, no infrastructure setup.
> 
> Second, unpredictable traffic: workloads that go from 0 to 1000s of requests unpredictably. Lambda scales instantly with zero management. On EC2 you'd over-provision to handle spikes, wasting money during idle.
> 
> Third, cost optimization for low-traffic APIs: if your API gets 100 requests per day, running EC2 24/7 is wasteful. Lambda costs pennies for the same workload.
> 
> When NOT to use Lambda: long-running processes (>15 min timeout), stateful workloads, applications requiring persistent connections (WebSockets — though Lambda + API Gateway can do this), or workloads with strict latency requirements where cold starts matter.
> 
> Modern architecture: use Lambda for event handlers and light APIs, ECS/EKS for stateful services and heavy compute, EC2 for legacy or specialized needs. Lambda often glues these together."

**Q: How do you handle Lambda cold starts?**

> "Cold starts happen when Lambda has to spin up a new container to handle a request. Duration varies by runtime — Python and Node.js are around 100-300ms, Java can be 1-2 seconds because JVM initialization.
> 
> Strategies: First, optimize package size. Smaller deployment = faster init. Remove unnecessary dependencies, use Lambda Layers for shared libraries.
> 
> Second, choose runtime carefully. If cold start matters, use Python or Node.js over Java/.NET.
> 
> Third, use Provisioned Concurrency for critical low-latency workloads. Keeps N containers pre-warmed at all times. Costs extra but eliminates cold starts. Good for user-facing APIs where 2-second cold start is unacceptable.
> 
> Fourth, warm-up strategy: schedule CloudWatch Events to invoke Lambda every 5 minutes to keep containers warm. Cheaper than Provisioned Concurrency but less reliable.
> 
> Fifth, architectural pattern: use SQS or SNS in front of Lambda for async workloads where cold start doesn't matter. Queue absorbs the burst, Lambda processes at its natural pace.
> 
> Real-world: for most workloads, cold start doesn't matter — the first user might see 300ms extra, subsequent users are instant. Optimize only when you measure it as a problem."

**Q: How does Lambda pricing work?**

> "Two components: requests and duration.
> 
> Requests: $0.20 per million requests. First 1 million free per month.
> 
> Duration: measured in GB-seconds. Memory allocated × execution time. Priced at $0.0000167 per GB-second. First 400,000 GB-seconds free per month.
> 
> Example calculation: Lambda with 1GB memory runs for 500ms, called 1M times per month: Requests: 1M × $0.20/M = $0.20 Duration: 1M × 1GB × 0.5s = 500K GB-seconds. First 400K free, pay for 100K. 100K × $0.0000167 = $1.67 Total: about $1.87/month.
> 
> Compare to always-on EC2 running the same workload — $15-20/month plus operational overhead. For event-driven workloads with idle time, Lambda is often 10x cheaper.
> 
> Cost optimization tips: allocate the RIGHT amount of memory. More memory = more CPU = faster execution = potentially cheaper. Use AWS Lambda Power Tuning tool to find optimal memory setting. Also, minimize package size to reduce cold start duration, and cache values across invocations (module-level variables persist between warm invocations)."

---

---

# 13. CloudFormation — IaC

## Concept

**AWS CloudFormation** is Infrastructure as Code (IaC) — define AWS resources in YAML or JSON templates, and CloudFormation creates, updates, and manages them for you.

**Key characteristics:**

- **Declarative language** — you define the desired end state
- **Template-based** — YAML or JSON files
- **AWS Native** — supports all AWS resources
- **Cost-free tool** — you only pay for resources you create
- **State management** — CloudFormation tracks state internally (no external state file like Terraform)

## Why IaC?

```
Consistency:     Same template = same infrastructure every time
Automation:      No manual clicking in console
Repeatable:      Deploy to dev, staging, prod with same code
Version control: Templates in Git = full change history
Rollback:        Failed deployment auto-rolls back
Auditability:    Every change is recorded
```

## CloudFormation Concepts

```
Template
    ├── YAML or JSON file defining resources
    └── Reusable across environments

Stack
    ├── A collection of resources created from a template
    └── Manage all resources as one unit

Stack Set
    ├── Deploy same template across multiple accounts/regions
    └── Great for enterprise multi-account setups

Change Set
    ├── Preview changes before applying (like Terraform plan)
    └── See what will be created/updated/deleted

Drift Detection
    ├── Detects resources changed outside CloudFormation
    └── Reports differences from template
```

## Simple CloudFormation Template Example

```yaml
---
AWSTemplateFormatVersion: '2010-09-09'
Description: 'Simple EC2 instance'

Resources:
  SimpleEC2Instance:
    Type: "AWS::EC2::Instance"
    Properties:
      InstanceType: t3.micro
      ImageId: ami-02db68a01488594c5
      Tags:
        - Key: Name
          Value: MySimpleInstance
```

## Common Use Cases

```
1. Create EC2 Instances with Security Groups + Elastic IPs
2. Provision S3 Buckets with configurations
3. Set up VPCs with Subnets and Route Tables
4. Create RDS databases with automated backups
5. Deploy Lambda functions with API Gateway
6. Set up ELB and ASG for web apps
7. Deploy IAM roles, policies, users
```

## CLI Commands

```bash
# Create a stack
aws cloudformation create-stack \
  --stack-name MyStack \
  --template-body file://template.yaml

# Update stack
aws cloudformation update-stack \
  --stack-name MyStack \
  --template-body file://template.yaml

# Delete stack (removes ALL resources)
aws cloudformation delete-stack --stack-name MyStack

# See what will change (preview)
aws cloudformation create-change-set \
  --stack-name MyStack \
  --change-set-name my-changes \
  --template-body file://template.yaml
```

## CloudFormation vs Terraform

```
CloudFormation                     Terraform
────────────────────────────────   ────────────────────────────────
AWS only                           Multi-cloud (AWS, Azure, GCP)
YAML/JSON                          HCL (HashiCorp)
State managed by AWS               State file (you manage)
Free                               Free (Terraform Cloud has pricing)
Better AWS integration             More flexible, better community
Native drift detection             terraform plan
Change Sets                        terraform plan
```

**Real world:** Most modern teams use Terraform because of multi-cloud support and better tooling. CloudFormation is common in AWS-only shops or enterprises with existing AWS investments.

## Memory Hint

- **CloudFormation** = "AWS's built-in Terraform"
- **Template** = "recipe for infrastructure"
- **Stack** = "the actual cooked meal" (created from template)
- **Change Set** = "preview of what will change"

## Interview Q&A — CloudFormation

**Q: CloudFormation vs Terraform — which do you prefer?**

> "Depends on the context.
> 
> CloudFormation: AWS-native, no external state to manage, integrates deeply with AWS services (Change Sets, drift detection built-in). Free service. Better for AWS-only shops or when you want minimal external dependencies.
> 
> Terraform: multi-cloud (AWS, Azure, GCP, Kubernetes, and thousands of providers via community). Better developer experience with HCL. Larger ecosystem — pre-built modules for everything. State management is more explicit which is powerful but requires care.
> 
> My preference: Terraform for most modern projects. Reasons: multi-cloud portability, better modularity, larger community, and it's the industry standard for infrastructure code in 2026. Also, learning Terraform gives you skills that transfer across all clouds.
> 
> I'd choose CloudFormation when: the team is AWS-exclusive and has existing CloudFormation investment, or when the workload uses AWS-specific features that Terraform lags on."

---

---

# 14. Route 53 — DNS

## Concept

**AWS Route 53** is a scalable DNS service. Named after DNS port 53. Handles:

- Domain registration
- DNS routing
- Health checking

## DNS Refresher

**DNS (Domain Name System)** translates human-friendly domain names to IP addresses.

```
User types: www.example.com
    ↓
DNS lookup: www.example.com → 192.168.0.2
    ↓
Browser connects to 192.168.0.2
```

Without DNS, we'd have to remember IPs. DNS is the internet's phone book.

**Default port:** 53 (both UDP and TCP)

## Route 53 Features

```
Domain Registration
    ├── Register new domains directly
    └── Manage renewal, transfers, WHOIS privacy

Hosted Zones
    ├── Container for DNS records for a domain
    ├── Public hosted zone: internet-facing
    └── Private hosted zone: internal VPC

DNS Records
    ├── A record: domain → IPv4
    ├── AAAA record: domain → IPv6
    ├── CNAME: domain → another domain
    ├── MX: mail server records
    ├── TXT: verification, SPF, DKIM
    ├── NS: nameserver records
    └── SRV: service location records

Routing Policies
    ├── Simple: one record, one target
    ├── Weighted: split traffic (10% to new version)
    ├── Latency-based: route to closest region
    ├── Failover: primary/secondary for DR
    ├── Geolocation: route based on user's country
    └── Multi-value: return multiple IPs (basic load balancing)

Health Checks
    ├── Monitor endpoints
    ├── Trigger DNS failover
    └── Alert via SNS
```

## DNS Record Types Deep Dive

```
A Record (IPv4)
    Example: www.google.com → 12.34.56.78
    Most common — direct IP mapping

AAAA Record (IPv6)
    Example: www.example.com → 2001:db8::1
    IPv6 equivalent of A record

CNAME Record (Canonical Name)
    Example: blog.example.com → www.example.com
    Alias for another domain
    CANNOT be used at zone apex (root domain)

MX Record (Mail Exchange)
    Example: example.com → mail.example.com (Priority 10)
    Directs email to your mail server

TXT Record (Text)
    Example: example.com → "v=spf1 include:_spf.example.com ~all"
    Used for verification, SPF (email spam prevention), DKIM

NS Record (Name Server)
    Example: example.com → ns-123.awsdns-45.org
    Authoritative name servers for the domain

SRV Record (Service)
    Example: _sip._tcp.example.com → 10 60 5060 sipserver.example.com
    Service location — used for VoIP, LDAP, etc.
```

## How Route 53 Works

**Workflow:**

```
1. Domain Registration
   Register a domain (myexample.com) via Route 53 or transfer existing

2. Hosted Zone Creation
   Route 53 creates a hosted zone with 4 nameservers (NS records)

3. DNS Records
   Add A, CNAME, MX, TXT records as needed
   Point them to AWS resources (EC2 IP, ALB, CloudFront, S3)

4. Routing Policies
   Configure how DNS resolves queries (simple, weighted, latency, etc.)

5. Health Checks (optional)
   Monitor endpoints
   Route away from unhealthy endpoints
```

## Routing Policies — Use Cases

**Simple Routing:**

```
myexample.com → 54.123.45.67

One record, one destination. Basic.
```

**Weighted Routing (Canary Deployments):**

```
myexample.com:
    ├── 90% traffic → ALB v1 (old version)
    └── 10% traffic → ALB v2 (new version)

Gradually shift traffic to new version. If issues, revert.
```

**Latency-Based Routing (Global Apps):**

```
User in Europe    → EU-West ALB (lowest latency)
User in Asia      → Mumbai ALB (lowest latency)
User in America   → US-East ALB (lowest latency)

Automatically route to region with best latency.
```

**Failover Routing (Disaster Recovery):**

```
Primary:   Mumbai ALB (healthy) → 100% traffic
Standby:   US-East ALB (backup)

If primary health check fails:
    Route 53 automatically switches to US-East

Automated DR at DNS level.
```

**Geolocation Routing (Compliance/Content):**

```
User in EU    → EU servers (GDPR compliance)
User in China → China-specific content
User elsewhere → Default servers
```

## Health Checks

**Monitor endpoints:**

- HTTP/HTTPS endpoints
- TCP endpoints
- Metric-based (CloudWatch alarms)

**Actions on failure:**

- Route DNS to failover target
- Send SNS notification
- Update DNS records

## Multi-Region Deployment Pattern

```
Primary Region: Mumbai (ap-south-1)
    ├── ALB + EC2 in ASG (across 3 AZs)
    └── RDS Multi-AZ

DR Region: US-East (us-east-1)
    ├── ALB + EC2 in ASG (pilot light mode - minimal instances)
    └── RDS Read Replica

Route 53:
    ├── Latency-based routing normally
    ├── Failover to US-East if Mumbai unhealthy
    └── Health checks monitoring both regions
```

## Use Cases Summary

```
Hosting Websites
    ├── Point domain to EC2, ALB, S3, or CloudFront
    └── Manage DNS records for the site

Load Balancing
    ├── Distribute traffic across endpoints
    └── Basic multi-value or weighted routing

Disaster Recovery
    ├── Health checks + failover routing
    └── Automatic failover on region outage

Multi-Region Deployments
    ├── Latency-based or geolocation routing
    └── Route users to nearest region

Email Configuration
    ├── MX records for mail servers
    └── TXT records for SPF/DKIM/DMARC
```

## Pricing

**Not free — billed for:**

- Hosted zones: $0.50 per zone per month
- DNS queries: $0.40 per million queries
- Health checks: $0.50-$0.75 per health check per month
- Domain registration: varies by TLD ($10-40/year typical)

**No free tier** — charges start immediately.

## Memory Hints

- **Route 53** = "AWS's DNS service" (53 = DNS port)
- **A Record** = "domain → IP"
- **CNAME** = "domain → another domain"
- **Hosted Zone** = "container for DNS records"
- **Routing Policy** = "how to answer DNS queries"

## Interview Q&A — Route 53

**Q: Explain a real production DNS setup for a global web app.**

> "Global e-commerce app with 3 regions: Mumbai, US-East, EU-West.
> 
> Domain registered via Route 53 (myapp.com). Hosted zone contains multiple record sets.
> 
> Primary user-facing records use latency-based routing. Route 53 measures latency from user's DNS resolver to each region and returns the fastest one. User in Delhi gets Mumbai IP, user in California gets US-East, user in Paris gets EU-West.
> 
> Each regional endpoint is an ALB. Health checks monitor each ALB every 30 seconds. If Mumbai ALB fails, Route 53 stops returning it — traffic auto-routes to the healthy regions.
> 
> For disaster recovery, we also configure failover routing on the primary region. If Mumbai's ALB fails, traffic fails over to US-East as backup.
> 
> Additional records: MX for email via Google Workspace, TXT for SPF/DKIM/DMARC verification, CNAME for www subdomain aliasing to main domain, another CNAME pointing to CloudFront distribution for static assets like images.
> 
> Result: users always hit the nearest healthy region. Zero downtime during regional outages. Sub-100ms latency globally."

**Q: What's the difference between A record and CNAME?**

> "A record maps a domain directly to an IPv4 address. `www.example.com → 54.123.45.67`. Route 53 returns the IP, browser connects.
> 
> CNAME (Canonical Name) is an alias — maps a domain to ANOTHER domain. `blog.example.com → www.example.com`. Route 53 returns another hostname, which then gets resolved to an IP via another DNS lookup.
> 
> Two important CNAME limitations: First, CNAME cannot be used at the zone apex (root domain). You can't have `example.com` as CNAME — must be A record. This is a DNS standard, not AWS specific. Second, CNAME adds an extra DNS lookup, slightly slower than A record.
> 
> AWS's solution: Route 53 Alias records. Alias behaves like CNAME but at the zone apex. `example.com → ALB DNS name`. Also faster because Route 53 resolves the alias internally.
> 
> Use A record for direct IP mapping. Use Alias when pointing to AWS resources (ALB, CloudFront, S3, other Route 53 records). Use CNAME only for pointing to non-AWS domains from non-apex subdomains."

---

---

# 15. CloudFront — CDN

## Concept

**AWS CloudFront** is a Content Delivery Network (CDN) — speeds up delivery of web content by caching it at servers close to users (edge locations).

**Why CDN:** Without CDN, all users hit your origin server. User in Europe accessing a server in Mumbai = 250ms+ latency.

With CDN:

```
User in Europe → CloudFront Edge in London (10ms latency)
    ├── If cached: instant response from edge
    └── If not cached: edge fetches from origin, caches for next user
```

## How CloudFront Works

```
1. User requests myapp.com/logo.png
2. DNS routes to nearest CloudFront edge location
3. Edge checks cache:
   ├── HIT: return cached content (fast)
   └── MISS: fetch from origin, cache locally, return
4. Subsequent users at that edge get cached response
5. TTL expires → edge refetches from origin
```

**Result:** Static content served from 400+ edge locations worldwide with millisecond latency.

## What Gets Cached

**By default:**

- Static content (images, CSS, JS, videos)
- Configurable via cache policies

**Can also cache:**

- Dynamic content (HTML, API responses)
- Configure via cache behaviors and TTLs

**Never cached:**

- Sensitive/user-specific data
- Backend logic
- Configurable via cache policies

## Cache Behavior Controls

```
TTL (Time to Live)
    ├── How long CloudFront caches content before refetching
    ├── Min TTL, Default TTL, Max TTL
    └── Example: cache images for 24 hours

Cache Behaviors
    ├── Different rules for different URL patterns
    ├── /images/* → long cache (1 day)
    ├── /api/* → no cache (dynamic API)
    └── /* → default (1 hour)

Origin Headers
    ├── Cache-Control from origin can override CloudFront
    └── Vary header controls per-user caching
```

## Browser Cache vs CloudFront

```
Browser Cache
    ├── Files cached on USER'S device
    ├── Only helps that individual user
    ├── First visit slow, repeat visits fast
    └── Local optimization

CloudFront (CDN)
    ├── Files cached at edge locations
    ├── Helps ALL users in that region
    ├── First user slow, subsequent users fast
    └── Global optimization

Both work together — browser cache + CDN = best performance
```

## Origin Types

**Where CloudFront fetches content from:**

```
S3 Bucket
    ├── Most common origin
    ├── Static websites
    └── Media files

EC2 Instance
    ├── Direct to EC2 with public IP
    └── Not recommended (no HA)

ALB (Application Load Balancer)
    ├── Preferred for dynamic content
    ├── Load balanced across multiple instances
    └── Multi-AZ HA

Custom Origin
    ├── Any HTTP/HTTPS endpoint
    ├── On-premises servers, other clouds
    └── Any accessible URL
```

## Practical Setup — Static Website with CloudFront

**Full flow:**

```
1. S3 bucket with static website files (index.html, css/, js/, images/)
2. Configure S3 as CloudFront origin
3. Create CloudFront distribution
4. Attach SSL certificate from ACM (free)
5. Configure cache behaviors:
   ├── /images/* → long cache (1 week)
   ├── /css/*, /js/* → medium cache (1 day)
   └── /* → default (1 hour)
6. Point Route 53 alias to CloudFront distribution
7. Done — global CDN, HTTPS, custom domain
```

## Real Production Use Cases

**E-commerce Site:**

```
Static assets (images, CSS, JS)  → CloudFront (cached long)
Product pages (HTML)              → CloudFront (cached briefly)
API calls                         → CloudFront (cache disabled or short TTL)
User dashboard                    → CloudFront (bypass cache — user-specific)
```

**Video Streaming:**

```
Video segments (HLS)  → CloudFront (aggressive caching)
Video manifests       → CloudFront (short TTL)
Live streams          → CloudFront with low-latency streaming
```

**Global App:**

```
Users worldwide → CloudFront (nearest edge)
                → Origin: ALB in one primary region

Effect: users in Europe/Asia see near-local response times
        without needing to deploy in every region
```

## CloudFront Security Features

```
HTTPS Enforcement
    ├── Redirect HTTP to HTTPS
    ├── Modern TLS versions
    └── Free SSL cert via ACM

Signed URLs / Signed Cookies
    ├── Restrict access to paid content
    └── Time-limited access tokens

WAF Integration
    ├── AWS Web Application Firewall
    ├── Block malicious traffic at edge
    └── SQL injection, XSS protection

Geo-Restriction
    ├── Block/allow specific countries
    └── Compliance and content licensing

DDoS Protection
    ├── AWS Shield Standard (free with CloudFront)
    └── AWS Shield Advanced (paid, more protection)

Origin Access Identity (OAI) / Origin Access Control (OAC)
    ├── Restrict S3 access — only CloudFront can access
    └── Users MUST go through CloudFront
```

## Pricing

**Based on:**

- Data transfer OUT to internet
- HTTPS requests
- Origin fetches

**Free tier:** 1TB data transfer + 10M requests per month for first 12 months.

**Real cost example:** Small static website (50GB/month traffic) = ~$5-10/month with CloudFront.

## Memory Hints

- **CloudFront** = "AWS's CDN" (delivery network)
- **Edge Location** = "mini data center close to users"
- **Origin** = "source of truth" (S3, ALB, EC2)
- **TTL** = "how long to cache before refresh"
- **Cache Hit** = "found at edge" (fast)
- **Cache Miss** = "not at edge, fetch from origin" (slower first time)

## Interview Q&A — CloudFront

**Q: How does CloudFront improve performance for a global user base?**

> "Three mechanisms.
> 
> First, geographic proximity. CloudFront has 400+ edge locations worldwide. A user in London requesting content from an origin in Mumbai would normally see 200-300ms latency. With CloudFront, they hit a London edge in 5-10ms. Massive latency reduction.
> 
> Second, caching. Once content is cached at an edge, subsequent users at that edge get instant responses. The origin isn't hit again until TTL expires. This offloads origin traffic and reduces load.
> 
> Third, network optimization. AWS uses its private network backbone between edges and origins — often faster than the public internet. TCP connections are reused. HTTPS handshakes are optimized.
> 
> Combined effect: 10x latency reduction and 90% origin load reduction is typical for static content.
> 
> For dynamic content, CloudFront can still help with connection pooling and geographic proximity even without caching. And with the new 'Origin Shield' feature, CloudFront can consolidate multiple edge requests to reduce origin load."

**Q: Design a static website deployment on AWS with global availability, HTTPS, and low latency.**

> "The reference architecture: Route 53 + CloudFront + S3 + ACM.
> 
> S3 bucket hosts the static files (HTML, CSS, JS, images). Block Public Access enabled — only CloudFront can access it via Origin Access Control (OAC).
> 
> CloudFront distribution sits in front of S3. It caches content at 400+ edge locations globally. Cache behaviors: /images/* long TTL, /css/_/js/_ medium TTL, /* default 1 hour.
> 
> SSL certificate from AWS Certificate Manager (ACM). Free, auto-renews, attached to CloudFront. Users get HTTPS with a proper certificate.
> 
> Route 53 hosted zone for the domain. Alias record for `www.myapp.com` points to CloudFront distribution. Optionally, apex record (`myapp.com`) uses Alias to CloudFront.
> 
> Result: global static site, HTTPS, low latency worldwide, DDoS protection via CloudFront + AWS Shield Standard. Total cost for a small site: under $5/month.
> 
> If we needed dynamic content later: add API Gateway + Lambda behind CloudFront using the /api/* cache behavior with cache disabled. Backend Node.js on Lambda scales automatically."

---

---

# 16. ECS — Elastic Container Service

## Concept

**AWS ECS (Elastic Container Service)** is a cloud-based container management service that runs and manages Docker containers on a cluster of virtual servers.

**What it handles for you:**

- Container **creation** (spins up containers based on your task definition)
- Container **management** (health monitoring, restart on failure)
- Container **updating** (rolling deployments)
- **Scaling** (horizontal scaling based on demand)

## Core ECS Terms

```
Cluster
    ├── Logical grouping of tasks and services
    ├── Hosts all resources and infrastructure
    └── One cluster can have many services

Task Definition
    ├── Blueprint for how a container runs
    ├── Similar to a Kubernetes Deployment spec
    ├── Specifies: image, CPU, memory, ports, env vars, IAM role
    └── Versioned (task-def:1, task-def:2, etc.)

Task
    ├── Running instance of a task definition
    ├── Represents actual running containers
    └── Each task = 1+ containers running together (like a K8s pod)

Service
    ├── Long-running task manager
    ├── Handles scaling, load balancing, health
    ├── Ensures N tasks are always running
    └── Similar to a K8s Deployment
```

## ECS Launch Types

**Two ways to run containers on ECS:**

```
1. EC2 Launch Type
   ├── You manage the underlying EC2 instances
   ├── More control, more responsibility
   ├── Cheaper for high steady load
   └── Best when you need custom AMIs, GPUs, or specific instance types

2. Fargate Launch Type (Serverless)
   ├── AWS manages the underlying infrastructure
   ├── You just define containers — no servers to manage
   ├── Pay per task (vCPU + memory + time)
   ├── More expensive per unit but zero ops
   └── Best for variable workloads, microservices
```

**Decision:**

- **High steady workload** (containers running 24/7 fully utilized): EC2 launch type
- **Variable, sporadic, or microservices**: Fargate

## ECS Architecture

```
ECS Cluster
    ├── EC2 Instances (or Fargate — no instances visible)
    │   └── ECS agent running on each instance
    │
    ├── Service: web-app
    │   ├── Task Definition: web-app:v3
    │   ├── Desired count: 5 tasks
    │   └── Load balancer: attached to ALB target group
    │       ├── Task 1 (Container: nginx + node.js)
    │       ├── Task 2 (Container: nginx + node.js)
    │       ├── Task 3 (Container: nginx + node.js)
    │       ├── Task 4 (Container: nginx + node.js)
    │       └── Task 5 (Container: nginx + node.js)
    │
    └── Service: worker
        ├── Task Definition: worker:v2
        ├── Desired count: 3 tasks
        └── No load balancer (background worker)
```

## Task Definition Example (JSON)

```json
{
  "family": "flask-app",
  "networkMode": "awsvpc",
  "requiresCompatibilities": ["FARGATE"],
  "cpu": "512",
  "memory": "1024",
  "executionRoleArn": "arn:aws:iam::xxx:role/ecsTaskExecutionRole",
  "containerDefinitions": [
    {
      "name": "flask",
      "image": "aakash0908/flask-app:v2",
      "portMappings": [
        {
          "containerPort": 5000,
          "protocol": "tcp"
        }
      ],
      "environment": [
        {"name": "FLASK_ENV", "value": "production"}
      ],
      "secrets": [
        {
          "name": "MONGO_URI",
          "valueFrom": "arn:aws:secretsmanager:xxx:secret:mongo-uri"
        }
      ],
      "logConfiguration": {
        "logDriver": "awslogs",
        "options": {
          "awslogs-group": "/ecs/flask-app",
          "awslogs-region": "ap-south-1",
          "awslogs-stream-prefix": "flask"
        }
      }
    }
  ]
}
```

## ECS vs Kubernetes (EKS)

```
ECS                                    EKS (Kubernetes)
────────────────────────────────       ────────────────────────────────
AWS proprietary                        Open-source, portable
Simpler to learn                       Steeper learning curve
Deep AWS integration                   Cloud-agnostic
No K8s features (Ingress, etc.)        Full K8s ecosystem
Free (pay for underlying compute)      $0.10/hour for control plane
Best for AWS-only shops                Best for multi-cloud, K8s expertise
```

**Rule of thumb:**

- **ECS**: startups on AWS, teams new to containers, simple use cases
- **EKS**: enterprises, teams with K8s expertise, multi-cloud requirements

## Real Production Setup

**Microservices on ECS Fargate:**

```
Application Load Balancer
    ↓
ECS Cluster (Fargate)
    ├── Service: auth-service (3 tasks)
    ├── Service: user-service (5 tasks)
    ├── Service: payment-service (3 tasks)
    └── Service: notification-worker (2 tasks — no LB)
    ↓
RDS (managed database)
DynamoDB (session storage)
S3 (file storage)
```

**Deployment flow:**

```
1. Developer pushes code
2. CI (GitHub Actions) builds Docker image
3. Push image to ECR (AWS's Docker registry)
4. Update task definition to new image tag
5. ECS rolling deployment:
   ├── Start new tasks with v2
   ├── Wait for health checks
   ├── Deregister old v1 tasks from ALB
   └── Stop v1 tasks
6. Zero downtime deployment complete
```

## Memory Hints

- **ECS** = "AWS's container platform" (simpler than K8s)
- **Cluster** = "the party venue" (where everything happens)
- **Task Definition** = "recipe for a task"
- **Task** = "one running instance" (like a K8s pod)
- **Service** = "keeps N tasks running always"
- **Fargate** = "serverless containers" (no VMs to manage)

## Interview Q&A — ECS

**Q: When would you choose ECS Fargate over ECS EC2 launch type?**

> "Depends on workload characteristics and operational preferences.
> 
> Fargate when: you want zero infrastructure management, workload has variable or unpredictable traffic, you're running microservices with independent scaling needs, or you're a small team without capacity to manage EC2. Pay only for what you use, no server patching.
> 
> EC2 launch type when: you have steady high load (24/7 fully utilized containers) — Fargate becomes expensive at high sustained utilization. Need specific instance types (GPU, high-memory). Need custom AMIs or specific OS-level configurations. Already have EC2 fleet you want to leverage.
> 
> Cost comparison: at high utilization (>80% for months), EC2 with Reserved Instances can be 50% cheaper than Fargate. But Fargate eliminates operational overhead — worth the premium for many teams.
> 
> Modern preference: Fargate for most microservices. EC2 for specialized workloads. This is what most teams I've worked with settle on."

**Q: ECS vs EKS — how do you decide?**

> "Depends on team skills, ecosystem needs, and portability requirements.
> 
> ECS when: AWS-only strategy, team new to containers, want simplest possible container orchestration, deep AWS service integration matters (IAM per task, native ALB integration, CloudWatch out of the box).
> 
> EKS when: multi-cloud portability needed, existing K8s expertise on team, need Kubernetes ecosystem tools (Helm, ArgoCD, Istio), running complex workloads that benefit from K8s features (StatefulSets, CronJobs, Ingress with path-based routing, service mesh).
> 
> Cost consideration: EKS control plane is $0.10/hour ($73/month) regardless of usage. ECS control plane is free. For small workloads, ECS is cheaper.
> 
> My default recommendation for a team starting fresh: ECS Fargate if they're AWS-only and want simplicity. EKS if they have K8s expertise or expect to grow into complex workloads. The knowledge from either transfers reasonably well to the other if requirements change later."

---

---

# 17. EKS — Elastic Kubernetes Service

## Concept

**AWS EKS (Elastic Kubernetes Service)** is a managed Kubernetes service. AWS runs the control plane, you manage worker nodes (or use Fargate for serverless nodes).

**Why managed K8s:**

- Running your own K8s control plane is complex (etcd, API server, scheduler, controllers)
- Requires HA setup across AZs
- Requires patching, upgrades, security hardening
- EKS handles all of this — you focus on your workloads

## Kubernetes Refresher

Kubernetes is an open-source platform for automating deployment, scaling, and management of containerized applications.

**Cluster components:**

```
Control Plane (managed by AWS in EKS)
    ├── kube-apiserver (brain, all comms)
    ├── etcd (state storage)
    ├── kube-scheduler (places pods on nodes)
    └── kube-controller-manager (runs controllers)

Worker Nodes (you manage — or Fargate)
    ├── kubelet (node agent)
    ├── kube-proxy (networking)
    ├── containerd (runtime)
    └── Your pods running here
```

## EKS Node Options

```
1. Self-Managed EC2 Nodes
   ├── You launch and manage EC2 instances
   ├── Add to cluster via user_data or eksctl
   └── Full control, more work

2. Managed Node Groups
   ├── AWS manages EC2 lifecycle (launch, terminate, upgrade)
   ├── Easier to operate
   ├── Auto Scaling built-in
   └── Recommended for most use cases

3. Fargate
   ├── Serverless nodes — no EC2 management
   ├── Pay per pod (vCPU + memory + time)
   ├── Not all K8s features supported (DaemonSets, some networking)
   └── Best for stateless, variable workloads
```

## Prerequisites — CLI Tools

**AWS CLI:**

```bash
# Verify installation
aws --version

# Configure credentials
aws configure

# Verify identity
aws sts get-caller-identity
```

**eksctl** (EKS-specific CLI, easiest way to create clusters):

```bash
# Windows (Chocolatey)
choco install eksctl

# Mac (Homebrew)
brew tap weaveworks/tap
brew install weaveworks/tap/eksctl

# Verify
eksctl version
```

**kubectl** (Kubernetes CLI):

```bash
# Mac
brew install kubectl

# Windows
choco install kubernetes-cli

# Verify
kubectl version --client
```

## Creating an EKS Cluster

**Simple cluster with AutoMode:**

```bash
eksctl create cluster --name=my-cluster --enable-auto-mode
```

**With more options:**

```bash
eksctl create cluster \
  --name=production-cluster \
  --region=ap-south-1 \
  --node-type=t3.medium \
  --nodes=3 \
  --nodes-min=2 \
  --nodes-max=5 \
  --managed
```

**Options explained:**

- `--nodes=3` — start with 3 worker nodes
- `--nodes-min=2, --nodes-max=5` — Auto Scaling range
- `--node-type=t3.medium` — EC2 instance type
- `--managed` — managed node group (recommended)
- `--fargate` — use Fargate instead of EC2

**Takes 15-20 minutes** to fully provision. AWS creates VPC, subnets, IAM roles, control plane, node group.

## After Cluster Creation

```bash
# Update kubectl config to point to EKS cluster
aws eks update-kubeconfig --name my-cluster --region ap-south-1

# Verify cluster access
kubectl get nodes

# Deploy your app
kubectl apply -f deployment.yaml
```

## EKS Architecture

```
AWS Managed Control Plane
    ├── Multi-AZ HA (across 3 AZs)
    ├── Auto-patched, auto-upgraded
    ├── Costs $0.10/hour ($73/month) per cluster
    └── Accessed via kubectl

Worker Nodes (in your VPC)
    ├── EC2 or Fargate
    ├── Deployed across multiple AZs
    ├── Can use Cluster Autoscaler for automatic scaling
    └── You pay for these instances

Networking
    ├── AWS VPC CNI plugin (default)
    ├── Each pod gets a real VPC IP
    ├── Direct integration with Security Groups
    └── High performance networking

Storage
    ├── EBS CSI driver for persistent volumes
    ├── EFS CSI driver for shared storage
    └── S3 for object storage
```

## Real Production Setup

```
EKS Cluster (production)
    ├── Managed Node Group 1: m5.large × 3 (production workloads)
    ├── Managed Node Group 2: c5.xlarge × 2 (CPU-intensive services)
    ├── Fargate Profile: system pods (kube-system namespace)
    │
    ├── Deployed apps:
    │   ├── Namespace: production
    │   │   ├── Deployment: api-service (5 replicas)
    │   │   ├── Deployment: worker (3 replicas)
    │   │   └── HPA scaling on CPU
    │   ├── Namespace: monitoring
    │   │   ├── Prometheus
    │   │   └── Grafana
    │   └── Namespace: ingress-nginx
    │       └── Ingress controller (routes external traffic)
    │
    ├── Cluster Autoscaler (scales node groups based on pod demand)
    ├── Metrics Server (for HPA)
    ├── AWS Load Balancer Controller (for Ingress → ALB)
    └── External DNS (auto-creates Route 53 records)
```

## Concepts to Master for EKS

Deep Kubernetes knowledge (revisit K8s notes):

- **Pods** — smallest unit
- **Deployments** — manages replicas, rolling updates
- **Services** — internal networking, load balancing
- **Ingress** — external HTTP/HTTPS routing
- **ConfigMaps & Secrets** — configuration and credentials
- **RBAC** — access control
- **HPA** — horizontal pod autoscaling
- **StatefulSets** — for databases, stateful apps
- **PersistentVolumes** — storage

## Memory Hints

- **EKS** = "AWS-managed Kubernetes"
- **eksctl** = "easy EKS cluster creator"
- **Managed Node Group** = "AWS-managed worker nodes"
- **Fargate on EKS** = "serverless K8s nodes"

## Interview Q&A — EKS

**Q: What are the advantages of managed EKS over self-managed K8s on EC2?**

> "Three main advantages: reliability, security, and operational efficiency.
> 
> Reliability: EKS runs the control plane across multiple AZs automatically. You get 99.95% SLA. Self-managed means you build and maintain HA control plane yourself — hard to get right, requires ongoing expertise.
> 
> Security: AWS patches the control plane automatically. Handles CVE responses for etcd, kube-apiserver, and other components. You focus on your workloads.
> 
> Operational efficiency: control plane upgrades are much easier — you initiate an upgrade, AWS handles the multi-step process. Self-managed upgrades are notoriously complex.
> 
> Cost is the trade-off — $73/month per cluster. But for anything production, that's negligible compared to the engineering time saved. And AWS handles disaster recovery, backup of etcd, and other complex operations.
> 
> My recommendation: unless you have a compliance requirement forcing on-prem or specific control plane customization, use EKS. The team's time is better spent on application-level work than K8s control plane maintenance."

**Q: What's the difference between EKS with EC2 and EKS with Fargate?**

> "Different node models.
> 
> EKS with EC2 (Managed Node Groups): AWS provisions and manages EC2 instances that run your pods. You choose instance types, size the node group. You have visibility and control over the nodes. Can SSH in (though shouldn't need to). Pay for EC2 instances 24/7.
> 
> EKS with Fargate: Fully serverless — no EC2 to manage. AWS runs pods on hidden infrastructure. You define Fargate profiles specifying which pods run on Fargate (by namespace, labels). Pay per pod (vCPU + memory + time). No node management, no capacity planning.
> 
> Fargate limitations: no DaemonSets, no privileged containers, no HostNetwork, no HostPath volumes. Not suitable for monitoring agents that need node-level access.
> 
> Common pattern: mix both. Fargate for stateless apps that scale up/down frequently. EC2 nodes for system pods (monitoring, ingress controllers, DaemonSets) that need node access. Best of both worlds."

---

---

# 18. Amplify — Fullstack Platform

## Concept

**AWS Amplify** is a platform that simplifies building, deploying, and hosting full-stack web and mobile apps.

**Combines:**

- Frontend hosting (React, Vue, Angular, static sites)
- Backend services (auth, API, database, storage)
- CI/CD (auto-deploy from Git)
- Custom domains and SSL

## What Amplify Handles

**Frontend:**

- HTML, CSS, JavaScript
- Frameworks: React, Angular, Vue.js, Next.js
- Static site hosting with global CDN

**Backend:**

- Python, Node.js, Java, PHP
- Frameworks: Express, Django, Spring Boot
- Databases: MySQL, MongoDB, PostgreSQL

**Key features:**

- Connect to backend services (Cognito for auth, AppSync for GraphQL API, S3 for storage)
- Automatic deployments from GitHub/GitLab/CodeCommit
- Custom domains with free SSL via ACM
- Preview environments (deploy PR to test URL)

## What You Can Build

```
1. Static Websites
   ├── Marketing sites, blogs, landing pages
   └── Deploy from GitHub → auto-deploys on every push

2. Single Page Apps (SPAs)
   ├── React, Vue, Angular apps
   ├── With client-side routing
   └── Global CDN, fast load times

3. Full-Stack Apps
   ├── Frontend + backend + database
   ├── User authentication via Cognito
   ├── GraphQL API via AppSync
   └── File storage via S3

4. Mobile App Backends
   ├── iOS/Android SDKs
   └── Same backend serves web + mobile
```

## Practical Workflow

**Deploy static site from GitHub:**

```
1. Push site code to GitHub repo
2. Amplify Console → Connect to GitHub
3. Select repo and branch (e.g., main)
4. Amplify auto-detects build settings (React, Vue, etc.)
5. Deploy → live at https://main.xxxxxxx.amplifyapp.com
6. Custom domain: point Route 53 to Amplify
7. SSL: automatic via ACM
8. Auto-deploy: every push to main → auto-deploy
```

## Amplify vs Other Options

```
Amplify vs S3 + CloudFront (static site)
    ├── Amplify: easier — GitHub integration, auto-deploy, one dashboard
    ├── S3+CF: more granular control, cheaper at scale
    └── Amplify: better for developer productivity, S3+CF for cost-optimized

Amplify vs Vercel/Netlify
    ├── Amplify: better AWS ecosystem integration
    ├── Vercel: better Next.js support, faster deploys
    └── Netlify: mature JAMstack platform

Amplify vs EKS/ECS (full-stack)
    ├── Amplify: PaaS — simpler, less control
    ├── EKS/ECS: IaaS-style — full control, more complexity
    └── Amplify: for developer-focused teams, EKS/ECS for production scale
```

## Memory Hints

- **Amplify** = "AWS's Vercel/Netlify" (fullstack PaaS)
- **Fullstack platform** = "one dashboard for frontend + backend"
- **Auto-deploy** = "git push → live in minutes"

---

---

# 19. Interview Master Q&A

## Foundation Questions

**Q: What is a Region and Availability Zone in AWS? Why do they matter?**

> "A Region is a physical geographic location (like Mumbai, N. Virginia, London). Each Region contains multiple Availability Zones — physically isolated data centers with independent power, cooling, and networking. AZs in the same region are connected via low-latency links.
> 
> They matter for high availability. Deploying across multiple AZs protects against single-datacenter failures. If AZ-a in Mumbai has a power outage, your workloads in AZ-b and AZ-c continue serving traffic. This is the foundation of cloud reliability — always design for multi-AZ.
> 
> For truly critical apps, deploy multi-region — protects against entire region outages. But most workloads are fine with multi-AZ, which handles the far more common AZ-level failures. Multi-region adds significant complexity and cost, so justify it with actual availability requirements."

**Q: What is the difference between horizontal and vertical scaling?**

> "Vertical scaling — scaling up. Add more power (CPU, RAM, disk) to an existing server. Example: upgrade EC2 from t3.medium to m5.xlarge. Simple but has limits — you can only get so big, and it usually requires downtime.
> 
> Horizontal scaling — scaling out. Add more servers behind a load balancer. Example: EC2 count from 3 to 30 instances behind ALB. No hardware limits, no downtime, can scale essentially infinitely.
> 
> Cloud makes horizontal scaling trivial — Auto Scaling Groups handle it automatically. That's why cloud native apps are designed to be stateless — any instance can serve any request, so horizontal scaling just works."

## IAM Deep Dive

**Q: Walk me through best practices for an IAM setup in production.**

> "First, protect the root account — enable MFA, never use it for daily operations, don't create access keys for it. Store the root password in a secrets manager. Use it only for account-level actions like billing.
> 
> Second, principle of least privilege everywhere. Never grant AdministratorAccess unless truly needed. Start with zero permissions, add specific policies as needed.
> 
> Third, use IAM roles for AWS services, never access keys. EC2 instances use instance profiles. Lambda functions use execution roles. GitHub Actions uses OIDC to assume roles temporarily. No long-lived credentials on servers.
> 
> Fourth, use Groups for humans. Create a Developers group with dev-appropriate policies. Add developers to the group. Removes user offboarding — remove from group, they lose access. Cleaner than managing per-user policies.
> 
> Fifth, enable MFA for all human users, especially admins. Consider hardware tokens (YubiKey) for admin accounts.
> 
> Sixth, audit regularly. Use IAM Access Analyzer to find overly permissive policies. AWS Config to detect drift. CloudTrail to audit all API calls.
> 
> Seventh, credential rotation — rotate access keys quarterly for programmatic access that must use keys. Better yet, replace with roles where possible."

## Networking

**Q: Explain a production VPC design for a 3-tier web application.**

> "Standard reference architecture, multi-AZ for high availability.
> 
> VPC CIDR: 10.0.0.0/16 — 65,536 addresses, plenty of room.
> 
> Six subnets across two AZs (minimum, three AZs for higher availability): Public subnets: 10.0.1.0/24 (AZ-a), 10.0.2.0/24 (AZ-b) — for ALB, NAT Gateway Private App subnets: 10.0.10.0/24 (AZ-a), 10.0.11.0/24 (AZ-b) — for EC2/ECS Private DB subnets: 10.0.20.0/24 (AZ-a), 10.0.21.0/24 (AZ-b) — for RDS
> 
> Public subnets have route to Internet Gateway. Private subnets route to NAT Gateway (in respective public subnet) for outbound internet — needed for package updates, external API calls.
> 
> Security groups layered: ALB SG allows 443 from 0.0.0.0/0. App SG allows traffic only from ALB SG. DB SG allows traffic only from App SG. Traffic path enforced by security groups — no direct public access to app or DB tier.
> 
> NACLs at subnet level for additional defense in depth if compliance requires. Otherwise, security groups alone are sufficient for most workloads.
> 
> This design gives you: high availability across 2 AZs, defense in depth with layered security groups, protected database tier, controlled outbound internet access via NAT Gateway, and clean separation of concerns."

## Storage

**Q: Compare EBS, S3, and EFS. When would you use each?**

> "Different storage types for different use cases.
> 
> EBS: block storage, attached to one EC2 instance at a time. Persistent, high performance. Use for OS root volumes, databases, application data on EC2. Not shareable — each EBS volume is exclusively attached to one instance (with rare exceptions).
> 
> S3: object storage, accessed via HTTP API. Infinitely scalable, 11 nines durability, region-wide access. Use for: application file storage, backups, static websites, data lakes, logs. Not a filesystem — no mount, no directories in the traditional sense.
> 
> EFS: file storage, NFS-based shared filesystem. Multiple EC2 instances can mount it simultaneously. Use for: shared configuration, ML training data, WordPress on multiple instances, application logs that multiple services write to.
> 
> Cost hierarchy: S3 cheapest (especially Glacier), EBS middle, EFS most expensive.
> 
> Modern architecture rarely uses EFS — instead use S3 for shared storage via API. EFS is legacy pattern for lift-and-shift of NFS-based apps."

## Cost Optimization

**Q: An engineering VP asks how to reduce AWS costs. What's your approach?**

> "Systematic approach starting from highest-impact levers.
> 
> First, right-size compute. AWS Compute Optimizer analyzes CloudWatch data and recommends smaller instance types. Typical savings 20-40% just by matching instance size to actual usage. Auto Scaling Groups handle variable load — don't over-provision baseline.
> 
> Second, use Reserved Instances or Savings Plans for predictable workloads. If EC2 instances run 24/7, RI gives 40-60% savings for 1-year commitment. Savings Plans are even more flexible — commit to spend rather than specific instances.
> 
> Third, Spot Instances for fault-tolerant workloads — up to 90% discount. Batch jobs, ML training, dev/test environments. Auto Scaling supports mixing On-Demand and Spot.
> 
> Fourth, storage optimization. S3 lifecycle policies move old data to Glacier — massive savings. Delete unattached EBS volumes (they still cost money). Right-size EBS — over-provisioned volumes are common.
> 
> Fifth, delete unused resources. Unused Elastic IPs cost money. Idle load balancers. Old snapshots. AWS Cost Explorer identifies these.
> 
> Sixth, use Serverless where appropriate. Lambda for sporadic workloads instead of always-on EC2. Fargate for containers with variable load. DynamoDB on-demand for unpredictable traffic. Pay only for actual use.
> 
> Seventh, network egress costs. Data transfer OUT to internet is expensive ($0.09/GB). CloudFront caching reduces origin egress. VPC endpoints eliminate charges for S3/DynamoDB traffic within region.
> 
> Eighth, cross-account visibility. AWS Cost Explorer, tagging strategy, budgets and alerts. Nobody spends what they don't measure.
> 
> Real result: I've seen teams reduce AWS costs 30-50% with 3-6 months of systematic optimization without any performance impact."

## Availability & Reliability

**Q: How would you design a highly available web application on AWS?**

> "Reference architecture achieving 99.99% availability.
> 
> DNS layer: Route 53 with health checks and failover routing. Optionally multi-region with latency-based routing for global users.
> 
> Edge layer: CloudFront CDN in front of everything. Caches static content globally, provides DDoS protection via AWS Shield, HTTPS via ACM certificate.
> 
> Load balancer layer: Application Load Balancer, multi-AZ, spread across 3 AZs. Cross-zone load balancing enabled.
> 
> Compute layer: Auto Scaling Group with EC2 instances across 3 AZs. Min 3 instances, max scales with demand. Instances behind ALB. Rolling deployments via Launch Template versions. Or ECS/EKS for container workloads.
> 
> Database layer: RDS Multi-AZ for production database. Automatic failover in ~60 seconds. Read replicas for scale. Cross-region read replica for DR.
> 
> Storage: S3 for object storage with cross-region replication for critical data. EBS with snapshots for EC2 data volumes.
> 
> Monitoring: CloudWatch metrics and alarms. SNS notifications to on-call. X-Ray for distributed tracing.
> 
> DR: Route 53 failover to a warm standby in another region. Terraform templates to spin up full replica if needed. RTO under 15 minutes, RPO under 5 minutes for critical data.
> 
> This architecture handles single instance failures, AZ failures, and even entire region failures. 99.99% is achievable, and if you eliminate every SPOF and add multi-region active-active, 99.999% becomes possible."

## Disaster Recovery

**Q: Explain the four DR strategies and their trade-offs.**

> "Four strategies with progressively lower RTO/RPO and higher cost.
> 
> Backup and Restore: cheapest, slowest. Regular backups to S3, restore in another region during disaster. RTO hours-days, RPO hours. Good for non-critical workloads.
> 
> Pilot Light: minimal infrastructure always running in DR region. Core components like databases replicating. During disaster, scale up other components. RTO 10-30 min, RPO minutes. Moderate cost.
> 
> Warm Standby: scaled-down copy of production always running in DR region. Ready to scale up quickly. RTO 5-15 min, RPO seconds-minutes. Higher cost.
> 
> Multi-Region Active-Active: full production in multiple regions, all serving traffic. Route 53 latency-based routing. Instant failover. RTO seconds, RPO near-zero. Highest cost — essentially double infrastructure.
> 
> How to choose: match strategy to business needs. E-commerce during holiday season → active-active worth the cost. Internal HR system → backup and restore is fine.
> 
> Most companies use warm standby for critical services and backup/restore for non-critical. Multi-region active-active is reserved for the most demanding workloads."

## Security

**Q: How do you secure a web application on AWS?**

> "Defense in depth across multiple layers.
> 
> Layer 1: Network isolation. VPC with private subnets for app and DB tiers. Public subnets only for load balancers and NAT Gateway. Security groups enforce least-privilege network access — DB SG only accepts traffic from App SG.
> 
> Layer 2: Edge security. AWS Shield Standard (free with CloudFront) blocks common DDoS. AWS WAF filters malicious HTTP traffic — SQL injection, XSS, bot traffic. Rate limiting on API endpoints.
> 
> Layer 3: Identity and access. IAM roles for service-to-service auth — no hardcoded credentials. Cognito for user authentication. MFA required for admin accounts.
> 
> Layer 4: Encryption. HTTPS everywhere with ACM certificates. EBS volumes encrypted at rest with KMS. S3 buckets encrypted. RDS encrypted. Secrets in Secrets Manager, not env vars.
> 
> Layer 5: Monitoring and audit. CloudTrail logs all API calls. GuardDuty detects threats via ML. AWS Config tracks configuration changes. Access logs from ALB and CloudFront to S3 for forensics.
> 
> Layer 6: Vulnerability management. ECR image scanning for containers. Amazon Inspector for EC2 vulnerabilities. Regular patching of AMIs.
> 
> Layer 7: Data protection. VPC endpoints for S3/DynamoDB — traffic never leaves AWS. S3 bucket policies block public access by default.
> 
> Ongoing: security audits quarterly. Incident response plan tested. Team trained on security. Compliance as needed (SOC 2, PCI DSS, HIPAA)."

---

# Session Wrap-up

**What we've covered:**

- 18 AWS services in depth with production context
- Real configuration examples for every service
- 30+ interview questions with senior-level answers
- Memory hints for retention
- Cross-service architecture patterns

**Your target:** MNC-level DevOps role. This document covers 90% of what will be asked in AWS-focused interviews at that level.

**Next steps in your DevOps mastery:**

1. Practice these patterns in your AWS account (ap-south-1, `t3.micro` free tier)
2. Build the projects mentioned — VPC, ALB+ASG, ECS, EKS
3. Move to Terraform notes (IaC on top of these services)
4. Move to CI/CD notes (deploying to these services)

**Security reminder:** Never store AWS access keys, RDS passwords, or any credentials in plaintext notes. Always use AWS Secrets Manager or environment variables from encrypted sources.