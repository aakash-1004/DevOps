
# GCP Mastery — Complete DevOps Notes

**Tags:** #gcp #devops #cloud #multicloud #interview #masterreview
**Status:** ✅ Active Reference
**Interview Relevance:** 🔴 Maximum
**Level:** DevOps Engineer / Cloud Engineer (2-3 YOE, cross-cloud)
**Prerequisites:** [[AWS-Mastery-Notes]] and [[Azure-Mastery-Notes]] — this doc assumes fluency in both and maps forward

> **How to use this note:**
> Every service follows: Concept → AWS Equivalent → Azure Equivalent → Key Differences → Real Config → Production Use Case → Console Path → Memory Hint → Interview Q&A.
> By mapping to BOTH AWS and Azure, you build trilateral fluency — the goal is that when an interviewer asks "have you worked across clouds," you can answer with concrete architectural comparisons across all three.

---

# GCP Basics — Quick Orientation

Before diving into services, three things GCP does differently that shape everything downstream:

**1. Project is the atomic unit — not Account (AWS) or Subscription (Azure)**
Every GCP resource belongs to a Project. Billing, quotas, API enablement, IAM — all scoped to Project. Projects are cheap and free to create (unlike Azure Subscriptions which require billing setup). Multi-project architecture is the norm, not the exception.

**2. APIs must be explicitly enabled per project**
Unlike AWS/Azure where services are just available, GCP requires you to enable each API before use. `gcloud services enable compute.googleapis.com` before you can create a VM. Frustrating at first, security win in reality — reduces attack surface.

**3. Service Accounts are first-class identities**
In AWS you have Users and Roles. In Azure you have Users and Managed Identities. In GCP the default identity for anything programmatic is a Service Account. Every project gets a default compute SA automatically. This shapes IAM design fundamentally.

## Global Infrastructure

- **Regions**: `us-central1`, `europe-west1`, `asia-south1` (Mumbai), etc.
- **Zones**: `us-central1-a`, `us-central1-b`, `us-central1-c` (like AZs)
- **Multi-region locations**: `us`, `eu`, `asia` — used for Cloud Storage and BigQuery
- **Global services**: VPCs, Load Balancers, Cloud DNS — these span regions natively

## Product Naming Convention

GCP names services descriptively — "Cloud SQL", "Cloud Storage", "Cloud Functions", "Compute Engine". Once you internalize the pattern, service names tell you what they do. Compare to AWS's mix of literal (S3, EC2) and codenames (Kinesis, Athena, Redshift).

---

# TABLE OF CONTENTS

1. [GCP Fundamentals — Organization, Folders, Projects](#1-gcp-fundamentals)
2. [IAM — Identity, Service Accounts, Roles](#2-iam)
3. [Compute Engine — Virtual Machines](#3-compute-engine)
4. [Persistent Disk — Block Storage](#4-persistent-disk)
5. [Cloud Storage — Object Storage](#5-cloud-storage)
6. [Cloud Load Balancing — The One-Product-Many-Modes Model](#6-cloud-load-balancing)
7. [Managed Instance Groups (MIG) — Autoscaling](#7-mig)
8. [Cloud SQL — Managed Relational](#8-cloud-sql)
9. [Firestore, Bigtable, Spanner — Three Distinct NoSQL/NewSQL Products](#9-firestore-bigtable-spanner)
10. [Cloud Functions & Cloud Run — Serverless](#10-cloud-functions--cloud-run)
11. [IaC — Deployment Manager, Config Connector, Terraform](#11-iac)
12. [Cloud DNS](#12-cloud-dns)
13. [Cloud CDN](#13-cloud-cdn)
14. [GKE — Google Kubernetes Engine](#14-gke)
15. [VPC Networking — Global VPCs and Firewall Rules](#15-vpc-networking)
16. [BigQuery — Analytics Warehouse](#16-bigquery)
17. [Three-Way Cross-Cloud Interview Master Q&A](#17-three-way-master-qa)

---
---

# 1. GCP Fundamentals

Before touching any service, understand GCP's resource hierarchy. This is the single biggest mental-model shift — and unlike Azure where Resource Groups are the surprise concept, in GCP it's how deeply **Project** sits at the center of everything.

## The Hierarchy

```
Organization                    (root — one per company, tied to Google Workspace/Cloud Identity)
    ↓
Folder                          (optional grouping — like Azure Management Groups)
    ↓
Project                         (billing + quota + API + IAM boundary — the workhorse)
    ↓
Resource                        (VM, bucket, DB — everything lives in a project)
```

## Key Concepts

**Organization**
- One per company, tied to your domain (`aakashrao.dev`)
- Free to create if you have Google Workspace or Cloud Identity
- The IAM root — organization-level policies cascade down
- Analogous to AWS Organizations root OR Azure Entra ID Tenant

**Folder**
- Optional intermediate layer for grouping projects
- Common patterns: folder per department (`engineering`, `finance`) or per environment (`prod`, `dev`, `sandbox`)
- IAM policies at folder level inherit to all child projects
- Analogous to Azure Management Groups (AWS has OUs)

**Project**
- **The load-bearing concept in GCP.** More like Azure Subscription than AWS Account in terms of power, but with a critical difference: **projects are cheap and instantly created**
- Billing account attached (multiple projects can share one billing account)
- Quotas are per-project (VMs, IPs, API calls)
- APIs enabled per-project (`gcloud services enable ...`)
- IAM policies attach to project (or higher — inherited down)
- Every resource has a project ID as part of its identity
- Project ID must be globally unique across all of GCP (like an S3 bucket name)

**Resource**
- The actual thing — VM, bucket, database
- Belongs to exactly one project
- Full resource name: `projects/{project-id}/zones/{zone}/instances/{vm-name}`

## Three-Way Hierarchy Comparison

| Concept | AWS | Azure | GCP |
|---|---|---|---|
| Company root | Organization | Entra ID Tenant | Organization |
| Grouping layer | Organizational Units (OUs) | Management Groups | Folders |
| Billing boundary | Account | Subscription | Project |
| Logical container | (tags + IAM) | Resource Group | Project (again — it's both) |
| Resource | Resource | Resource | Resource |

**The key GCP quirk:** Project plays TWO roles that are split in Azure. It's both the billing boundary (like Subscription) AND the logical grouping container (like Resource Group). GCP doesn't have a separate "resource group" concept — you create more projects instead.

## Why "Just Create More Projects" Is the GCP Way

In AWS, you tag resources and use IAM for grouping.
In Azure, you create Resource Groups within Subscriptions.
In GCP, you create separate projects: `myapp-prod`, `myapp-dev`, `myapp-sandbox`.

Because projects are:
- Free to create
- Delete cleanly (nuking a project nukes all resources — the atomic-cleanup superpower)
- Isolated by default (no cross-project access without explicit IAM grants)

...multi-project architectures are the default pattern, not a governance overhead.

## Real Configuration

```bash
# Login
gcloud auth login

# List existing projects
gcloud projects list

# Create a new project
gcloud projects create myapp-dev-aakash \
  --name="MyApp Dev" \
  --folder=123456789012

# Set default project for gcloud
gcloud config set project myapp-dev-aakash

# Link billing account to project (required before creating billable resources)
gcloud beta billing projects link myapp-dev-aakash \
  --billing-account=01ABCD-234567-EFGH89

# Enable APIs (each service you'll use needs its API enabled)
gcloud services enable \
  compute.googleapis.com \
  storage.googleapis.com \
  container.googleapis.com \
  sqladmin.googleapis.com

# List enabled APIs
gcloud services list --enabled

# Delete a project (30-day grace period, then permanent deletion)
gcloud projects delete myapp-dev-aakash
```

## Regions and Zones

- Regions: `us-central1` (Iowa), `us-east1` (South Carolina), `europe-west1` (Belgium), `asia-south1` (Mumbai), `asia-southeast1` (Singapore)
- Zones: `us-central1-a`, `us-central1-b`, `us-central1-c` — always at least 3 zones per region
- Location for storage/BigQuery: `us` (multi-region), `us-central1` (single region), `nam4` (dual-region North America)

Different from Azure: **all GCP regions have zones**. No exceptions where a region only has one location. Simpler HA design.

## Console Path

- Portal: `console.cloud.google.com`
- Top navbar: **Project selector** (this is where you switch projects — always check what project you're in)
- Left navigation: hamburger menu → categorized services (Compute, Storage, Databases, Networking, etc.)
- **Cloud Shell** (top right icon) — free Debian VM with gcloud/kubectl/terraform pre-installed, browser-based terminal
- **APIs & Services** → enable APIs, view usage
- **Billing** → cost breakdown by project/service/label

## Memory Hint

- **Organization** = the company root (like AWS Org, Azure Tenant)
- **Folder** = optional grouping (like OU or Management Group)
- **Project** = billing + quota + API + IAM — GCP's "load-bearing" concept
- **Projects are cheap** — the GCP idiom is "create more projects," not "use one big project"

"AWS uses one account with tags. Azure uses one subscription with resource groups. GCP just creates a new project."

## Interview Q&A — Fundamentals

**Q: You've worked with both AWS and Azure. Walk me through the equivalent hierarchy in GCP and where it's genuinely different.**

> "GCP has Organization → Folder → Project → Resource. Organization is the company root, tied to your Google Workspace domain. Folders are optional grouping — think of them like OUs in AWS Organizations or Management Groups in Azure. Both AWS and Azure engineers should recognize this three-way similarity so far.
>
> The interesting difference is Project. In GCP, Project plays two roles that Azure splits between Subscription and Resource Group. It's the billing boundary AND the logical container. GCP doesn't have a separate 'resource group' concept — the answer is 'just create more projects.'
>
> Why this matters: projects are free to create and instantly provisioned. You don't provision a new Azure Subscription for every environment — that requires billing setup, admin approval, and paperwork. But in GCP, `gcloud projects create myapp-dev` and you have a fresh, isolated environment in 10 seconds.
>
> Practical implication: multi-project architectures are the norm. A typical setup might have `myapp-prod`, `myapp-staging`, `myapp-dev`, `myapp-sandbox`, plus shared projects like `myapp-shared-network` and `myapp-shared-security`. Each project cleanly isolated with its own IAM policies. Deleting a project nukes everything cleanly.
>
> Interview trap: candidates from AWS backgrounds sometimes try to run everything in one project because 'that's what we did in AWS.' Senior signal is knowing GCP encourages splitting into more projects for isolation."

**Q: Explain the mental shift from 'AWS account' to 'GCP project' for a team migrating.**

> "AWS accounts are heavyweight — creating one involves AWS Organizations, root user setup, billing consolidation, Control Tower guardrails if you're doing it right. Teams keep the number small and use IAM + tags within each account.
>
> GCP projects are lightweight. You can have hundreds. A team migrating from AWS should NOT try to map one AWS account to one GCP project — that misses the model entirely.
>
> The right mapping: one AWS account often becomes multiple GCP projects. If in AWS you separated prod and dev via naming conventions and IAM within a single account, in GCP you'd separate them into two projects with independent IAM policies. If in AWS you used AWS Organizations with a prod account and dev account, that's a closer 1:1 to GCP projects.
>
> Practical migration pattern: start by categorizing AWS resources by lifecycle and blast radius. Anything sharing the same lifecycle (staging web tier + staging DB tier + staging cache) can live in one GCP project. Different environments = different projects.
>
> Cost implication: since projects are free, over-splitting has no financial cost. Under-splitting has isolation cost — a bad IAM binding can affect more resources. Err on the side of more projects.
>
> Bonus: use Folders to group related projects and apply org-wide policies. `engineering` folder holds all engineering projects; a folder-level IAM policy grants the engineering team default access to everything under it."

---
---

# 2. IAM — Identity, Service Accounts, Roles

## Concept

Cloud IAM (Identity and Access Management) is GCP's authorization system. It determines who (identity) can do what (role) on which resource.

## AWS Equivalent

**GCP IAM ≈ AWS IAM** (but with different structure — no policies-as-JSON in the same way)

## Azure Equivalent

**GCP IAM ≈ Azure RBAC** (structurally more similar to Azure than AWS)

Like Azure, GCP is role-based, not policy-based. You bind roles to identities at a scope. No hand-writing JSON policies unless you're doing custom roles.

## Three-Way Comparison

| Concept | AWS IAM | Azure RBAC | GCP IAM |
|---|---|---|---|
| Identity types | User, Role | User, Service Principal, Managed Identity | User, Service Account, Group |
| Permission bundle | Policy (JSON) | Role (predefined + custom) | Role (predefined + custom) |
| Attachment model | Policy → User/Role | Role Assignment → Identity + Scope | Role Binding → Identity + Resource |
| Scope hierarchy | Per-resource ARN | Sub → RG → Resource | Org → Folder → Project → Resource |
| Compute identity | Instance Profile (IAM Role) | Managed Identity | Service Account |

## Key Differences from AWS AND Azure

**1. Everything Programmatic Uses Service Accounts**

In AWS, an EC2 instance uses an Instance Profile (a wrapper around IAM Role). In Azure, a VM uses a Managed Identity. In GCP, a VM uses a **Service Account** — but Service Accounts are their own top-level identity type, not derived from user accounts.

Every project gets a **default Compute Engine service account** automatically. VMs, functions, GKE pods — they all authenticate as some Service Account. This makes SAs first-class citizens in a way IAM Roles and Managed Identities aren't quite.

**2. Three Tiers of Roles**

| Tier | Purpose | Example |
|---|---|---|
| **Basic roles** | Very broad, project-wide | `roles/owner`, `roles/editor`, `roles/viewer` |
| **Predefined roles** | Service-scoped, granular | `roles/compute.admin`, `roles/storage.objectViewer` |
| **Custom roles** | You define exact permissions | `roles/myorg.customRole` |

**Never use Basic roles in production.** They're too broad. `roles/editor` is essentially "admin on everything." Interview red flag if a candidate defaults to basic roles.

**3. Bindings Instead of Assignments**

GCP calls the "grant this role to this identity on this resource" operation a **binding**, not an assignment (Azure) or an attachment (AWS). Semantically identical to Azure — attach a role to an identity at a scope. The scope is any resource in the hierarchy (Org, Folder, Project, or specific resource).

**4. Service Account Impersonation**

A distinctive GCP pattern: users can **impersonate** Service Accounts. Instead of downloading a service account key file (bad practice — long-lived credential), you grant your user the `roles/iam.serviceAccountTokenCreator` role on the SA, then run:

```bash
gcloud auth application-default login --impersonate-service-account=my-sa@project.iam.gserviceaccount.com
```

Your commands execute as the SA without you holding its key. Same idea as AWS `sts:AssumeRole` but cleaner day-to-day.

**5. IAM Conditions**

GCP supports conditional role bindings using CEL (Common Expression Language) — grant access only when specific conditions are met (time of day, IP range, resource tag). Similar to AWS IAM policy conditions but structured differently.

## Real Configuration

```bash
# List current identity
gcloud auth list

# Create a service account
gcloud iam service-accounts create ci-deployer \
  --display-name="CI Deployment SA" \
  --description="Used by GitHub Actions to deploy"

# Grant SA a role at project scope
gcloud projects add-iam-policy-binding myapp-prod \
  --member="serviceAccount:ci-deployer@myapp-prod.iam.gserviceaccount.com" \
  --role="roles/run.admin"

# Grant a user impersonation rights on the SA (preferred over key files)
gcloud iam service-accounts add-iam-policy-binding \
  ci-deployer@myapp-prod.iam.gserviceaccount.com \
  --member="user:aakash@aakashrao.dev" \
  --role="roles/iam.serviceAccountTokenCreator"

# Impersonate the SA for CLI commands
gcloud auth application-default login --impersonate-service-account=ci-deployer@myapp-prod.iam.gserviceaccount.com

# Grant a group at folder scope (cascades to all projects under folder)
gcloud resource-manager folders add-iam-policy-binding 123456789 \
  --member="group:sre-team@aakashrao.dev" \
  --role="roles/compute.viewer"

# Create a custom role (rare — usually predefined is enough)
gcloud iam roles create backupOperator \
  --project=myapp-prod \
  --title="Backup Operator" \
  --description="Can trigger backups and read logs" \
  --permissions="compute.snapshots.create,logging.logEntries.list" \
  --stage=GA
```

## Production Use Case

**Team of 10 needs multi-project access, CI/CD deploys, audit trail:**

```
Organization: aakashrao.dev
├── Folder: engineering
│   ├── Project: myapp-prod
│   ├── Project: myapp-staging
│   └── Project: myapp-dev
│
├── Google Group: developers@aakashrao.dev
│   └── Binding: roles/editor on projects myapp-dev + myapp-staging directly
│
├── Google Group: sres@aakashrao.dev
│   └── Binding: roles/owner on myapp-prod (with break-glass audit alerting)
│   └── Binding: roles/viewer on myapp-dev, myapp-staging
│
├── Service Account: ci-deployer@myapp-prod.iam.gserviceaccount.com
│   ├── Binding: roles/run.admin, roles/cloudsql.client on myapp-prod
│   └── Federated identity to GitHub Actions via Workload Identity Federation
│       (no key files, no long-lived credentials)
│
└── Break-glass: 2 named users with roles/owner directly on Organization
    (audited quarterly, alerts on any use)
```

Same principles as AWS multi-account IAM or Azure RBAC — use groups not individuals, scope narrowly, prefer short-lived credentials.

## Console Path

- Console → **IAM & Admin** (from hamburger menu)
  - **IAM** — all bindings on the current project
  - **Service Accounts** — create/manage SAs
  - **Roles** — see all roles, create custom
  - **Audit Logs** — every IAM change is logged
  - **Workload Identity Federation** — federate external identities (GitHub, AWS, Azure) to GCP SAs

## Memory Hint

- **GCP IAM** = "Azure RBAC but with Service Accounts everywhere"
- **Basic roles** = never use (too broad)
- **Predefined roles** = default choice (`roles/compute.admin`, `roles/storage.objectViewer`)
- **Service Account** = the identity for anything programmatic
- **Impersonation** = the modern pattern (no key files)
- "Every project has a default compute SA. Every VM authenticates as some SA. GCP is Service-Account-first."

## Interview Q&A — IAM

**Q: Compare AWS IAM, Azure RBAC, and GCP IAM. Which model does GCP most resemble?**

> "GCP IAM is structurally closer to Azure RBAC than to AWS IAM. Both use role-based, not policy-based, models. You bind pre-built roles to identities at a scope. Custom roles exist but are the exception.
>
> The key differences from Azure:
>
> First, the scope hierarchy is different. Azure: Subscription → Resource Group → Resource. GCP: Organization → Folder → Project → Resource. The extra layers give more granularity for large orgs.
>
> Second, Service Accounts are more first-class in GCP. Azure has Managed Identities, which are lifecycle-tied to specific resources. GCP Service Accounts are standalone identities that any resource can use — a VM, a GKE pod, a Cloud Function. You create them explicitly and grant them roles just like you would a user.
>
> Third, three tiers of roles: basic, predefined, custom. Basic (owner/editor/viewer) is too broad for production. Predefined roles are the sweet spot — `roles/compute.instanceAdmin` for VM management, `roles/storage.objectViewer` for read access to buckets. Custom is rare.
>
> Compared to AWS IAM: GCP is much less about writing JSON policies. AWS forces you to think in terms of Actions and Resources at the JSON level. GCP mostly abstracts that away — you pick roles from a catalog and bind them.
>
> Practical migration mindset: an AWS engineer moving to GCP should stop reaching for JSON immediately. Search the predefined role catalog first. 90% of what you need already exists."

**Q: How would you securely wire up a GitHub Actions pipeline to deploy to GCP?**

> "Workload Identity Federation, no service account keys. This is the modern pattern and it's genuinely elegant.
>
> Traditional approach (bad): create a service account, download its key file as JSON, store in GitHub Secrets. Problems: long-lived credential, key rotation is manual, leaked key is a nightmare, key file works from anywhere.
>
> Modern approach: federate GitHub's OIDC tokens to a GCP Service Account.
>
> Setup:
> 1. Create a Workload Identity Pool in GCP
> 2. Add a GitHub OIDC provider to the pool: configures GCP to trust tokens issued by GitHub for a specific repo
> 3. Create a GCP Service Account for the deploy pipeline
> 4. Grant the Service Account the specific IAM roles it needs on the target project
> 5. Bind the SA to allow impersonation from GitHub Actions running in the specific repo
>
> Runtime: GitHub Action requests an OIDC token from GitHub, GCP validates it via the federation, exchanges it for a short-lived Service Account access token. Every workflow run uses a fresh 1-hour token. No secrets in GitHub except a public identifier.
>
> AWS equivalent: OIDC federation from GitHub to an IAM Role. Azure equivalent: OIDC federation from GitHub to a Service Principal via Entra ID. Same pattern in all three clouds — GCP arguably has the smoothest developer experience for setting it up."

---
---

# 3. Compute Engine

## Concept

Compute Engine is GCP's IaaS VM service. Rent a VM by the second, install what you want.

## AWS Equivalent

**Compute Engine = EC2**

## Azure Equivalent

**Compute Engine = Azure Virtual Machines**

Same category. Same use case. Different pricing model (sustained-use discounts, per-second billing), different machine type naming, different HA primitives.

## Key Differences from BOTH

**1. Per-Second Billing with 1-Minute Minimum**
AWS: per-second billing (with 1-min minimum for Linux, 1-hour for Windows historically).
Azure: per-second billing.
GCP: per-second billing, 1-minute minimum, applies to everyone.

Small difference — but GCP was first to per-second billing and it's aggressive about not rounding up.

**2. Sustained Use Discounts — AUTOMATIC**

GCP's killer pricing feature. Run a VM for more than 25% of the month, discounts kick in automatically. Up to 30% off if you run 24/7. **No commitment required.**

AWS and Azure require you to commit (Reserved Instances / Savings Plans) for equivalent discounts. GCP just gives it to you.

Committed Use Discounts (CUDs) exist too for deeper savings on 1-year or 3-year commits — those are the AWS RI / Azure Reserved Instance equivalents.

**3. Machine Type Naming**

| Family | Purpose | AWS Equivalent | Azure Equivalent |
|---|---|---|---|
| **E2** | General purpose, cost-optimized | AWS t3 | Azure B-series |
| **N2, N2D** | General purpose, balanced | AWS m5, m5a | Azure D-series |
| **C2, C2D** | Compute-optimized | AWS c5 | Azure F-series |
| **M2, M3** | Memory-optimized | AWS r5, x1 | Azure E-series |
| **A2, A3** | GPU (NVIDIA A100, H100) | AWS p4, p5 | Azure N-series |
| **T2A, T2D** | ARM (Ampere Altra, AMD Milan) | AWS Graviton | Azure Ampere |

Number roughly indicates generation, letter suffix indicates CPU vendor (D=AMD, A=ARM).

**4. Spot VMs and Preemptible VMs**

Older: **Preemptible VMs** — 24-hour max lifetime, evictable, ~80% off. Still supported for backward compatibility.
Newer: **Spot VMs** — no 24-hour limit, evictable, up to 91% off. **Use this for new workloads.**

Both AWS (Spot) and Azure (Spot VMs) have equivalents; GCP has both an older and newer version, which occasionally confuses documentation.

**5. Custom Machine Types**

Unique GCP feature: create VMs with any CPU/RAM combination you want. Need 3 vCPU with 15GB RAM? Just define it. AWS and Azure force you to pick from a catalog.

For most workloads, predefined types are fine and cheaper. Custom types are useful when you have precise memory-to-CPU ratios that don't match any predefined shape.

**6. Live Migration (No Reboots for Host Maintenance)**

GCP live-migrates VMs during host maintenance. Your VM keeps running, no reboot, no downtime. AWS and Azure use scheduled reboots for host maintenance.

## Real Configuration

```bash
# Enable API (first time in a project)
gcloud services enable compute.googleapis.com

# Create a VM
gcloud compute instances create vm-web-01 \
  --project=myapp-prod \
  --zone=us-central1-a \
  --machine-type=e2-medium \
  --image-family=debian-12 \
  --image-project=debian-cloud \
  --tags=web-server \
  --service-account=web-sa@myapp-prod.iam.gserviceaccount.com \
  --scopes=https://www.googleapis.com/auth/cloud-platform \
  --metadata=startup-script='#!/bin/bash
    apt-get update && apt-get install -y nginx
    systemctl start nginx'

# List VMs
gcloud compute instances list

# SSH (gcloud handles keys automatically via OS Login or metadata-based keys)
gcloud compute ssh vm-web-01 --zone=us-central1-a

# Stop (billing continues for disk only — no compute charge; similar to Azure deallocate, DIFFERENT from AWS stop)
gcloud compute instances stop vm-web-01 --zone=us-central1-a

# Suspend (like laptop sleep — RAM saved to disk, faster resume, small charge for RAM disk)
gcloud compute instances suspend vm-web-01 --zone=us-central1-a

# Start
gcloud compute instances start vm-web-01 --zone=us-central1-a

# Delete (deletes attached disk by default unless --keep-disks specified)
gcloud compute instances delete vm-web-01 --zone=us-central1-a
```

## The Startup Script → user_data Equivalent

GCP metadata `startup-script` runs at boot — same idea as AWS user_data and Azure cloud-init:

```bash
gcloud compute instances create vm-web-01 \
  --metadata-from-file=startup-script=install.sh
```

## Production Use Case

**Standard web tier on Compute Engine:**

```
Project: myapp-prod-web
├── VPC: prod-vpc (custom-mode, subnet us-central1: 10.0.1.0/24)
├── Instance Template: web-tier-v1
│   ├── Machine: n2-standard-2 (2 vCPU, 8GB RAM)
│   ├── Image: cos-stable (Container-Optimized OS)
│   ├── Boot disk: pd-balanced 50GB
│   ├── Metadata: startup-script pulls app container and runs
│   ├── Service Account: web-sa (with cloud-platform scope)
│   └── Network tag: web-server (matches firewall rules)
├── Managed Instance Group: web-mig
│   ├── Regional MIG (spans 3 zones for HA)
│   ├── Autoscaler: 3-20 based on CPU > 60% for 3 min
│   └── Health check: HTTP /health on port 8080
└── HTTPS Load Balancer → web-mig backend
```

Deploy via Terraform. Instance Template is immutable — updates create a new template version and MIG rolls it out.

## Console Path

- Console → **Compute Engine** → **VM instances**
- Create: **CREATE INSTANCE** → wizard (machine type, boot disk, networking, service account)
- **Instance templates** — reusable VM specs
- **Instance groups** — MIGs (managed) and unmanaged groups
- Individual VM: **Details**, **Monitoring**, **Serial console** (like AWS EC2 serial console — helpful for debugging boot issues)

## Memory Hint

- **E2** = economical (cheapest, general-purpose)
- **N2** = normal (balanced default)
- **C2** = compute (CPU-heavy)
- **M2/M3** = memory (RAM-heavy)
- **Stop = free** in GCP (unlike Azure where you must deallocate)
- **Sustained Use Discounts are automatic** — this is the pricing feature to remember for interviews

## Interview Q&A — Compute Engine

**Q: Compare Compute Engine pricing to AWS EC2 and Azure VMs. What's the practical difference?**

> "Same on-demand rates roughly, but GCP has two pricing wins that matter in real workloads.
>
> First, Sustained Use Discounts. If you run a VM for a significant portion of the month, GCP automatically discounts it — up to 30% off for a 24/7 workload. Zero commitment. AWS and Azure require you to explicitly commit via Reserved Instances or Savings Plans for equivalent savings. This is a real financial edge for teams that don't want to make long-term commits but still run steady workloads.
>
> Second, per-second billing with 1-minute minimum applies across the board. AWS and Azure also have per-second billing now but with some edge cases. GCP is simpler.
>
> Third — the stop behavior. In AWS, stopping an EC2 instance immediately stops compute billing but you still pay for EBS. In Azure, you must explicitly deallocate to stop billing — a common gotcha. In GCP, stopping a VM stops compute billing immediately (disk still charged) — same behavior as AWS, cleaner than Azure.
>
> For interviews: the sustained use discount story is the easy win to mention. It shows you understand cloud economics beyond 'just pick the cheapest instance type.'"

**Q: When would you use Custom Machine Types instead of predefined?**

> "Custom Machine Types are worth reaching for when your workload has a specific CPU-to-memory ratio that doesn't fit predefined shapes well.
>
> Example: an in-memory cache that needs 4 vCPUs and 40GB of RAM. Predefined options:
> - `n2-standard-4` gives you 4 vCPU, 16GB RAM — too little memory
> - `n2-highmem-4` gives you 4 vCPU, 32GB RAM — still too little
> - `n2-highmem-8` gives you 8 vCPU, 64GB RAM — plenty of both but you're paying for CPUs you don't need
>
> With a custom machine type: 4 vCPU, 40GB RAM exactly. You pay for what you use, no waste.
>
> Trade-offs:
> - Custom types cost slightly more per unit than predefined (small premium)
> - You lose some pricing conveniences — Sustained Use Discounts still apply, but they're capped differently
> - Not all machine families support custom types (E2 does, but T2A/T2D are predefined-only)
>
> Practical rule: benchmark your workload first. If a predefined type wastes more than 20% of a resource type, consider custom. Otherwise, predefined is simpler.
>
> AWS and Azure don't have direct equivalents — you're locked to their catalogs. Some workloads move to GCP specifically for this flexibility (memory-heavy databases especially)."

**Q: An engineer complains their VM 'randomly reboots.' How do you respond?**

> "First question: is it actually a reboot, or is it a live migration? On GCP, live migration is the default for host maintenance — the VM keeps running, no downtime, but you might see a brief network blip.
>
> Check the operation history: `gcloud compute operations list --filter='targetLink:vm-web-01'`. Look for `compute.instances.migrateOnHostMaintenance` operations. If those match the timing, it's not a reboot — it's live migration doing its job.
>
> If it's an actual reboot, check the maintenance policy:
> - `TERMINATE` — VM will be shut down for maintenance (this is the setting for certain instance types like GPUs or Confidential VMs which can't be live-migrated)
> - `MIGRATE` — default, live migration
>
> If the policy is TERMINATE (usually not by choice — some instance families require it), maintenance events cause actual reboots. Solution: either accept it and design for stateless workloads, or migrate to an instance family that supports live migration.
>
> Other causes to check: kernel panic in serial console output, OOM killer, health check failures triggering auto-heal on a Managed Instance Group.
>
> Compare to AWS: no live migration. Every maintenance event = reboot with 2 weeks notice. Compare to Azure: also no live migration (host maintenance = reboot). Live migration is a real GCP differentiator for legacy workloads that hate downtime."

---
---

# 4. Persistent Disk

## Concept

Persistent Disk is GCP's network-attached block storage — durable, resizable, snapshotable, attaches to Compute Engine VMs.

## AWS Equivalent

**Persistent Disk = AWS EBS**

## Azure Equivalent

**Persistent Disk = Azure Managed Disk**

Same category. Same lifecycle model. Different tier names, different regional/zonal design.

## Three-Way Comparison

| GCP | AWS | Azure | Use Case |
|---|---|---|---|
| `pd-standard` | `st1` / `sc1` | Standard HDD | Cheap, sequential, backup |
| `pd-balanced` | `gp3` (baseline) | Standard SSD | Modern default for general |
| `pd-ssd` | `gp3` (higher IOPS) / `io2` | Premium SSD | High-performance |
| `pd-extreme` | `io2 Block Express` | Ultra Disk | Extreme IOPS |
| `hyperdisk-*` | `io2 Block Express` | Ultra Disk | Newest tier, decoupled IOPS/GB |

## Key Differences from BOTH

**1. Regional vs Zonal Persistent Disks**

Unique GCP feature: **regional persistent disks** replicate synchronously across two zones. AWS EBS is zonal-only (you can snapshot cross-AZ but not have live sync). Azure has ZRS disks (three-AZ sync).

Regional PDs give you HA at the disk level — if a zone goes down, you can attach the disk to a VM in the other zone. Great for databases needing fast failover.

**2. Snapshots Work Cross-Region Natively**

`gcloud compute snapshots create` creates snapshots that are automatically globally accessible. Restore in any region.

AWS: EBS snapshots are per-region unless you explicitly copy across regions.
Azure: similar — snapshots are regional unless explicitly copied.

GCP snapshot model is more geo-flexible out of the box.

**3. Hyperdisk — Decouple IOPS from Size**

Newest tier (2023+): with Hyperdisk, you provision size and IOPS/throughput separately. Need a 100GB disk with 20,000 IOPS? Configure independently.

AWS `gp3` has similar decoupling. Azure Ultra Disk does too. Hyperdisk is GCP's answer.

**4. Automatic Snapshot Schedules**

GCP lets you attach a snapshot schedule policy directly to a disk. Daily/hourly snapshots run automatically without external scheduling. AWS uses AWS Backup or Lifecycle Manager. Azure uses Azure Backup. GCP's is built in and simpler for basic needs.

## Real Configuration

```bash
# Create a 100GB balanced PD
gcloud compute disks create data-disk-01 \
  --project=myapp-prod \
  --zone=us-central1-a \
  --size=100GB \
  --type=pd-balanced

# Attach to VM
gcloud compute instances attach-disk vm-web-01 \
  --disk=data-disk-01 \
  --zone=us-central1-a

# On the VM: format and mount
sudo mkfs.ext4 -F /dev/sdb
sudo mkdir /data
sudo mount /dev/sdb /data
echo '/dev/sdb /data ext4 defaults,nofail 0 0' | sudo tee -a /etc/fstab

# Resize a disk online (no downtime for supported OS)
gcloud compute disks resize data-disk-01 \
  --size=200GB \
  --zone=us-central1-a
# Then on VM: sudo resize2fs /dev/sdb

# Create a snapshot
gcloud compute snapshots create data-disk-01-snap-$(date +%Y%m%d) \
  --source-disk=data-disk-01 \
  --source-disk-zone=us-central1-a

# Create a snapshot schedule policy
gcloud compute resource-policies create snapshot-schedule daily-backup-policy \
  --region=us-central1 \
  --max-retention-days=30 \
  --daily-schedule \
  --start-time=03:00

# Attach policy to disk
gcloud compute disks add-resource-policies data-disk-01 \
  --zone=us-central1-a \
  --resource-policies=daily-backup-policy

# Create a REGIONAL PD (sync-replicated across 2 zones)
gcloud compute disks create data-disk-regional \
  --replica-zones=us-central1-a,us-central1-b \
  --size=200GB \
  --type=pd-balanced \
  --region=us-central1
```

## Production Use Case

**Database VM with regional PD for HA:**

```
VM: db-primary (us-central1-a)
├── Boot disk: pd-balanced 50GB
├── Data disk: pd-ssd 500GB REGIONAL (replicated to us-central1-b)
├── Log disk: pd-balanced 200GB (zonal, less critical)
├── Snapshot schedule: hourly for data disk, daily for logs
└── Failover: standby VM in us-central1-b can attach regional disk within 1 min if primary zone fails
```

Same failover time as Azure SQL DB HA. Cheaper than running two full DB instances with async replication.

## Memory Hint

- **pd-standard** = HDD (cheap, sequential)
- **pd-balanced** = default production choice (like AWS gp3)
- **pd-ssd** = high performance
- **Regional PD** = 2-zone sync replication (uniquely GCP)
- **Snapshots are global** by default — no cross-region copying needed
- **Attach snapshot schedule policy** to disks — set-and-forget backups

## Interview Q&A — Persistent Disk

**Q: Compare Persistent Disk types across the three clouds. When would you pick GCP for storage-heavy workloads?**

> "Baseline mapping: pd-standard ≈ st1 ≈ Standard HDD (cheap sequential). pd-balanced ≈ gp3 ≈ Standard SSD (modern default). pd-ssd ≈ io2 ≈ Premium SSD (high-performance). pd-extreme and Hyperdisk ≈ io2 Block Express ≈ Ultra Disk (extreme IOPS).
>
> GCP-specific wins for storage-heavy workloads:
>
> First, regional PDs — synchronous replication across two zones at the disk layer. If you need sub-minute failover for a stateful workload without running a full replicated database, GCP gives you this natively. AWS engineers usually solve it with DRBD or database-level replication; Azure has ZRS disks but the failover story is different. Regional PDs give you the simplest 'attach disk to VM in surviving zone' failover story of the three.
>
> Second, cross-region snapshots by default. GCP snapshots are stored in a Google-managed multi-region location and can be restored anywhere. AWS and Azure both require explicit cross-region copies which are extra cost and effort.
>
> Third, decoupled IOPS via Hyperdisk. Similar to gp3 and Ultra Disk, but GCP tends to price aggressively for these workloads.
>
> When to choose GCP for storage: databases that need multi-zone HA without complex replication setup, workloads with backup/DR requirements across regions, and analytics workloads with unpredictable IOPS needs where Hyperdisk shines."

---
---

# 5. Cloud Storage

## Concept

Cloud Storage is GCP's object storage — store files as objects, access via HTTP API, essentially unlimited scale.

## AWS Equivalent

**Cloud Storage = S3**

## Azure Equivalent

**Cloud Storage = Azure Blob Storage** (but simpler — no Storage Account wrapper)

## Three-Way Comparison

| Concept | AWS S3 | Azure Blob | GCP Cloud Storage |
|---|---|---|---|
| Container name | Bucket | Container (in Storage Account) | Bucket |
| Namespace | Global | Per-storage-account | **Global** |
| Wrapper resource | None | Storage Account | None |
| Location model | Region | Region | Region OR multi-region OR dual-region |
| Signed URLs | Pre-signed URL | SAS token | Signed URL |
| Storage classes | 6 (Standard, IA, One Zone-IA, Glacier IR, Glacier FR, Glacier DA) | 4 (Hot, Cool, Cold, Archive) | 4 (Standard, Nearline, Coldline, Archive) |

## Key Differences from BOTH

**1. Buckets Are NOT Wrapped in an Account**

Azure requires a Storage Account, then containers inside it. GCP is like S3 — buckets are top-level objects in a project. Simpler.

Bucket names are globally unique across all of GCP (like S3 bucket names). Reserve early.

**2. Location Types: Region / Dual-Region / Multi-Region**

Unique GCP flexibility:
- **Region** (`us-central1`) — cheapest, single-region durability
- **Dual-region** (`nam4` = us-central1 + us-east1) — synced across two regions, moderate cost, good DR
- **Multi-region** (`us`, `eu`, `asia`) — spread across all regions in the geography, highest durability, most expensive

AWS S3 has only region + Cross-Region Replication (async). Azure Blob has ZRS/GRS/GZRS as tiers within a storage account. GCP's location model is a distinct axis independent of storage class.

**3. Storage Classes**

| GCP | AWS | Azure | Retrieval time |
|---|---|---|---|
| Standard | S3 Standard | Hot | Immediate |
| Nearline | S3 Standard-IA | Cool | Immediate (30-day min) |
| Coldline | S3 Glacier Instant Retrieval | Cold | Immediate (90-day min) |
| Archive | S3 Glacier Deep Archive | Archive | Hours (365-day min) |

All GCP classes support instant retrieval except Archive. AWS Glacier tiers historically had multi-hour retrieval; GCP has always emphasized instant-access even for cold tiers.

**4. Object Lifecycle Rules**

Same concept as S3 lifecycle policies. Transition objects between classes based on age, delete after N days, etc.

**5. Object Versioning + Retention Policies**

Same as S3 versioning. Additionally, GCP supports **Bucket Lock** — an immutable retention policy where objects cannot be deleted before a retention period expires, even by admins. Good for compliance (WORM — Write Once Read Many).

## Real Configuration

```bash
# Create a bucket (name globally unique, choose location and class)
gsutil mb -p myapp-prod -l us-central1 -c standard gs://myapp-prod-user-uploads

# Or with gcloud
gcloud storage buckets create gs://myapp-prod-user-uploads \
  --project=myapp-prod \
  --location=us-central1 \
  --default-storage-class=standard \
  --uniform-bucket-level-access

# Upload a file
gcloud storage cp local-file.txt gs://myapp-prod-user-uploads/

# Or via gsutil (older CLI, still widely used)
gsutil cp local-file.txt gs://myapp-prod-user-uploads/

# List objects
gcloud storage ls gs://myapp-prod-user-uploads/

# Download
gcloud storage cp gs://myapp-prod-user-uploads/local-file.txt .

# Generate a signed URL (5 min TTL)
gcloud storage sign-url gs://myapp-prod-user-uploads/local-file.txt \
  --duration=5m \
  --impersonate-service-account=web-sa@myapp-prod.iam.gserviceaccount.com

# Set lifecycle policy via JSON
cat > lifecycle.json <<EOF
{
  "lifecycle": {
    "rule": [
      { "action": {"type": "SetStorageClass", "storageClass": "NEARLINE"},
        "condition": {"age": 30} },
      { "action": {"type": "SetStorageClass", "storageClass": "COLDLINE"},
        "condition": {"age": 90} },
      { "action": {"type": "Delete"},
        "condition": {"age": 2555} }
    ]
  }
}
EOF

gsutil lifecycle set lifecycle.json gs://myapp-prod-user-uploads/

# Enable versioning
gsutil versioning set on gs://myapp-prod-user-uploads/

# Set retention policy (compliance WORM)
gsutil retention set 7y gs://myapp-prod-user-uploads/
gsutil retention lock gs://myapp-prod-user-uploads/   # Cannot be undone
```

## Uniform vs Fine-Grained Access Control

Buckets have two ACL modes:
- **Uniform (recommended)** — all objects inherit bucket-level IAM. No per-object ACLs. Simpler, cleaner. Default for new buckets.
- **Fine-grained** — legacy per-object ACLs on top of IAM. More flexible but complex; hard to audit.

Always create new buckets with `--uniform-bucket-level-access`. AWS/Azure engineers should reach for this to match the "bucket policy only" mental model.

## Production Use Case

**User upload pipeline with least-privilege access:**

```
Project: myapp-prod-data
├── Bucket: myapp-prod-user-uploads (us-central1, Standard, uniform ACL)
│   ├── Lifecycle: to Nearline after 30 days, Coldline after 90, delete after 7 years
│   ├── Versioning: on
│   ├── Encryption: CMEK via KMS key in same region
│   └── Access: web-sa@myapp-prod (roles/storage.objectAdmin scoped to this bucket)
│
├── Bucket: myapp-prod-public-assets (multi-region us, Standard)
│   ├── Public read via allUsers with roles/storage.objectViewer
│   └── Cloud CDN in front for caching
│
├── Bucket: myapp-prod-backups (dual-region nam4)
│   ├── Bucket Lock: 7-year retention (immutable)
│   └── Access: backup-sa only, no user access
│
Web tier (Cloud Run):
├── Uses SA web-sa
├── Upload flow: backend generates signed URL, client uploads to bucket directly
└── Download flow: backend generates signed URL for read, client downloads directly
```

Same architecture as an S3-based flow. No account keys, all signed URLs, immutable audit trail.

## Console Path

- Console → **Cloud Storage** → **Buckets**
- Bucket detail:
  - **Objects** — browse and upload
  - **Configuration** — location, storage class, versioning
  - **Permissions** — IAM bindings
  - **Protection** — versioning, retention, lifecycle
  - **Lifecycle** — rules for transitions and deletions

## Memory Hint

- **Cloud Storage** = "S3 without the annoying region-name-in-URL, with more location flexibility"
- **Bucket names global** — reserve like S3
- **Multi-region location** = maximum durability, HIGHER cost
- **Dual-region** = the sweet spot for regional DR
- **Uniform access** = default for new buckets, use it

## Interview Q&A — Cloud Storage

**Q: A team wants to migrate from S3 to Cloud Storage. What are the operational surprises?**

> "Migration itself is straightforward — both are HTTP object stores with similar APIs. `gsutil` even has an S3-compatible mode. The interesting operational surprises are elsewhere:
>
> First, location model. S3 is per-region. Cloud Storage adds dual-region and multi-region options that don't exist in S3 (S3 replication is separate and async). For workloads that need cross-region durability, GCP simplifies this — pick a dual-region location and you get sync replication built in. AWS engineers usually configure CRR separately.
>
> Second, storage classes look similar but have different minimum storage durations. AWS S3 Intelligent-Tiering automatically transitions based on access patterns. GCP has Autoclass which is similar but requires opt-in per bucket. Similar outcomes, different opt-in models.
>
> Third, retrieval behavior for cold tiers is fundamentally different. AWS S3 Glacier historically had multi-hour retrieval — retrieving an archived object was an async operation. GCP Coldline and Archive support instant retrieval (Archive charges more for it, but no wait). This changes application design — you can query archived data in Cloud Storage without a two-step retrieval pattern.
>
> Fourth, uniform access control. GCP encourages bucket-level IAM only, no per-object ACLs. AWS S3 has bucket policies AND ACLs, and teams often end up with both which creates auditing nightmares. GCP's uniform mode forces cleaner design.
>
> Migration mechanics: `gsutil rsync` from S3 to Cloud Storage handles most workloads. Storage Transfer Service (a GCP-managed service) handles large migrations with parallelism and resumability. For petabyte-scale, use Transfer Appliance for physical shipment."

**Q: Design a WORM-compliant archive on Cloud Storage.**

> "Bucket Lock with retention policy is the answer. This is GCP's WORM (Write Once Read Many) primitive.
>
> Setup:
> 1. Create a bucket, dual-region or multi-region for durability
> 2. Storage class Archive (cheapest for long-term retention)
> 3. Enable versioning (so overwrites don't destroy data)
> 4. Set retention policy: `gsutil retention set 7y gs://audit-archive/` (7 years, adjust for regulatory requirement)
> 5. LOCK the retention policy: `gsutil retention lock gs://audit-archive/` — this is irreversible, no admin can shorten it after locking
> 6. Enable object holds for specific critical objects that need even more protection
>
> Result: objects cannot be deleted or modified before 7 years pass. Even project owners cannot bypass. This meets SEC 17a-4, FINRA, and most WORM compliance standards.
>
> Comparison:
> - AWS S3 Object Lock with Compliance mode is the direct equivalent
> - Azure Blob Immutable storage with Legal Hold is the Azure equivalent
> - All three achieve similar compliance. GCP's Bucket Lock is arguably the simplest to configure.
>
> Cost optimization: Archive class + multi-region location + versioning. Total cost per TB per month is a few dollars — cheap for the compliance value.
>
> Watch out for: locked retention policies cannot be undone. Test in a non-production bucket first. And separate the auditors' write access from the readers' read access — no single identity should be able to both write and read compliance data."

---
---

# 6. Cloud Load Balancing

**This is where GCP diverges most from AWS/Azure.** In AWS you learn ALB, NLB, GWLB. In Azure you learn Load Balancer, App Gateway, Front Door, Traffic Manager. In GCP, there's **one service — Cloud Load Balancing — with multiple modes**.

## Concept

Cloud Load Balancing is a single product spanning L4 and L7, regional and global, external and internal — you configure the mode at deploy time.

## AWS Equivalent

Depends on mode:
- **Global external HTTPS LB** ≈ CloudFront + ALB combined
- **Regional external HTTPS LB** ≈ ALB
- **Network LB** (passthrough) ≈ NLB
- **Internal HTTPS LB** ≈ Internal ALB

## Azure Equivalent

Depends on mode:
- **Global external HTTPS LB** ≈ Front Door + Application Gateway
- **Regional external HTTPS LB** ≈ Application Gateway
- **Network LB** ≈ Azure Load Balancer (Standard)
- **Internal HTTPS LB** ≈ Internal Application Gateway

## Cloud Load Balancer Modes — The Matrix

| Mode | Layer | Scope | Traffic Type |
|---|---|---|---|
| **Global External Application LB** | L7 | Global | HTTP/HTTPS from internet |
| **Regional External Application LB** | L7 | Regional | HTTP/HTTPS from internet |
| **Internal Application LB** | L7 | Regional | HTTP/HTTPS internal |
| **External Passthrough Network LB** | L4 | Regional | TCP/UDP from internet |
| **Internal Passthrough Network LB** | L4 | Regional | TCP/UDP internal |
| **External Proxy Network LB** | L4 | Global or Regional | TCP with proxy |

**Rule of thumb:**
- Global web app → **Global External Application LB** (this is the standout — one product with anycast IP)
- Regional web app → **Regional External Application LB**
- TCP/UDP internal → **Internal Passthrough Network LB**
- TCP internet-facing → **External Passthrough Network LB**

## Key Differences from BOTH AWS and Azure

**1. Global Load Balancer with a Single Anycast IP**

The Global External Application LB is unique to GCP at the depth GCP does it. One IP address, announced from every Google edge POP worldwide via anycast. Users hit the closest POP automatically, traffic then routes over Google's private backbone to the healthiest backend region.

AWS: you'd combine CloudFront (edge) + ALB (regional). Two services, two configurations, no single anycast IP.

Azure: you'd combine Front Door (edge) + Application Gateway (regional). Same pattern as AWS.

GCP: one Global External Application LB does all of it. Single IP, single config, edge caching + global routing + regional load balancing.

**2. Backend Types Are Extremely Flexible**

Backends can be: MIGs (like ASG), zonal NEGs, serverless NEGs (Cloud Run/Functions/App Engine), internet NEGs (on-prem or another cloud), buckets (static content). You can mix backend types in one LB.

Real example: a single Global External Application LB can route `/api/*` to a MIG running Compute Engine VMs, `/functions/*` to Cloud Functions, and `/static/*` to a Cloud Storage bucket — all with one URL map.

AWS and Azure require more service composition for equivalent architectures.

**3. Cloud Armor for WAF**

Cloud Armor is the WAF layer, integrated with the Application LB. OWASP Top 10 pre-configured rules, custom rules with CEL expressions, rate limiting per IP or per user identity, geo-blocking, DDoS protection.

Similar to AWS WAF or Azure Front Door WAF. Same category, tightly integrated with the LB.

**4. Cloud CDN Integration**

Cloud CDN is enabled with a checkbox on the LB — same LB serves as origin, edge caching is added on top. Fewer services to configure than CloudFront-in-front-of-ALB.

## Real Configuration — Global External HTTPS LB

Building a global HTTPS LB is 6+ resources. Here's the minimum:

```bash
# Assume you have a MIG (web-mig) with an instance template already

# 1. Reserve a global static IP
gcloud compute addresses create web-lb-ip --global

# 2. Managed SSL certificate (Google manages renewal)
gcloud compute ssl-certificates create web-ssl-cert \
  --domains=www.aakashrao.dev \
  --global

# 3. Health check
gcloud compute health-checks create http web-health-check \
  --port=8080 \
  --request-path=/health

# 4. Backend service (defines how to send traffic to backends)
gcloud compute backend-services create web-backend \
  --protocol=HTTP \
  --port-name=http \
  --health-checks=web-health-check \
  --global \
  --enable-cdn

# 5. Add MIG as a backend
gcloud compute backend-services add-backend web-backend \
  --instance-group=web-mig \
  --instance-group-region=us-central1 \
  --balancing-mode=UTILIZATION \
  --max-utilization=0.8 \
  --global

# 6. URL map (routing rules)
gcloud compute url-maps create web-url-map \
  --default-service=web-backend

# 7. HTTPS proxy (attaches URL map to SSL cert)
gcloud compute target-https-proxies create web-https-proxy \
  --url-map=web-url-map \
  --ssl-certificates=web-ssl-cert

# 8. Forwarding rule (attaches proxy to IP)
gcloud compute forwarding-rules create web-forwarding-rule \
  --address=web-lb-ip \
  --global \
  --target-https-proxy=web-https-proxy \
  --ports=443
```

Yes, it's more moving parts than an AWS ALB or Azure App Gateway. The benefit is composability — you can build almost any load balancing topology from these primitives.

## Cloud Armor Attachment

```bash
# Create WAF policy
gcloud compute security-policies create web-armor-policy \
  --description="Web WAF policy"

# Add OWASP rule
gcloud compute security-policies rules create 1000 \
  --security-policy=web-armor-policy \
  --expression="evaluatePreconfiguredExpr('sqli-v33-stable')" \
  --action="deny-403"

# Rate limit: 100 req/min per IP
gcloud compute security-policies rules create 2000 \
  --security-policy=web-armor-policy \
  --expression="true" \
  --action="rate-based-ban" \
  --rate-limit-threshold-count=100 \
  --rate-limit-threshold-interval-sec=60 \
  --ban-duration-sec=600 \
  --conform-action=allow \
  --exceed-action=deny-429 \
  --enforce-on-key=IP

# Attach to backend service
gcloud compute backend-services update web-backend \
  --security-policy=web-armor-policy \
  --global
```

## Production Use Case

**Global three-tier app on one Global External Application LB:**

```
Global External Application LB (anycast IP: 34.120.x.x)
├── SSL cert: managed, auto-renewing, covers www + apex
├── Cloud Armor: OWASP rules + rate limit + geo-block sanctioned countries
├── Cloud CDN: enabled with 5-min TTL for /static/*
│
├── URL Map:
│   ├── /api/* → api-backend (MIGs across us-central1 + europe-west1)
│   ├── /functions/webhook → functions-neg-backend (Cloud Functions in us-central1)
│   ├── /static/* → static-bucket-backend (Cloud Storage bucket, CDN cached)
│   └── /* → web-backend (MIG in us-central1 with cross-region failover to us-east1)
│
└── Backend services:
    ├── web-backend, api-backend: cross-region MIGs, latency-based routing
    ├── functions-neg-backend: serverless NEG
    └── static-bucket-backend: Cloud Storage bucket
```

One IP, one config, one certificate for everything. This is genuinely simpler than the AWS equivalent (CloudFront + ALB + Route 53 + WAF + Lambda@Edge separately).

## Console Path

- Console → **Network Services** → **Load balancing**
- **CREATE LOAD BALANCER** wizard walks through: type (App/Network), scope (Global/Regional), external/internal, backends, frontend
- Existing LB detail: **Frontend**, **Host and path rules** (URL map), **Backend configuration**, **Security** (Armor + SSL)

## Memory Hint

- **One product, many modes** — GCP unifies what AWS/Azure split
- **Global External Application LB** = anycast + edge + WAF + CDN in one thing (AWS: CloudFront+ALB, Azure: Front Door+App Gateway)
- **Backend types** = MIG + serverless NEG + bucket + internet NEG (mix in one LB)
- **URL map** = the routing logic (like ALB rules or App Gateway path routing)
- **Cloud Armor** = WAF layer

## Interview Q&A — Cloud Load Balancing

**Q: You're designing a globally distributed web app. Compare how you'd architect the load balancing layer in AWS, Azure, and GCP.**

> "The architectural pattern is the same — anycast edge, regional origins, cross-region failover — but the number of services differs.
>
> AWS: CloudFront in front for edge caching and anycast, ALB in each region as origin, Route 53 with health-checked latency-based routing for regional failover, AWS WAF attached to CloudFront. Four services composed. Well-understood pattern, but four service configurations to keep in sync.
>
> Azure: Front Door in front for edge and global routing, Application Gateway in each region as regional LB, Azure DNS for the domain, WAF policies on Front Door. Three-to-four services. Slightly cleaner than AWS because Front Door bundles edge + WAF + global routing.
>
> GCP: Global External Application Load Balancer with cross-region backends. One service. Anycast IP is announced from every Google edge POP globally, users hit closest POP automatically, traffic routes over Google's private backbone to the healthiest backend region. Cloud CDN adds edge caching to the same LB. Cloud Armor adds WAF. All configured on one LB resource.
>
> The GCP architecture is genuinely simpler for global apps. Fewer moving parts, one certificate to manage, one URL map to edit, one place to look for logs and metrics.
>
> AWS wins on maturity and ecosystem — more third-party integrations, more documentation, larger community. Azure wins on WAF sophistication and rules engine at edge. GCP wins on architectural simplicity for global apps.
>
> For a genuinely global workload where you don't need AWS-specific integrations, GCP's LB is the cleanest choice."

**Q: A team complains that CloudFront + ALB feels 'over-engineered' for their needs. How does GCP simplify this?**

> "That team is the perfect candidate for GCP's Global External Application LB.
>
> The mental model shift: instead of thinking 'CDN in front of load balancer,' think 'load balancer with edge and CDN built in.'
>
> On GCP, you create one Global External Application LB. Toggle Cloud CDN on the backend service — done, edge caching added. Attach a managed SSL certificate — done, HTTPS with auto-renewal. Attach a Cloud Armor policy — done, WAF. Add backends from any region — done, cross-region failover. Point your DNS at the LB's anycast IP — done, deployed.
>
> Fewer resources to reason about. Fewer places for misconfiguration.
>
> Trade-offs:
> - Vendor lock-in is real. This design is GCP-specific and doesn't port cleanly to AWS/Azure.
> - Less granular pricing. You can't independently optimize edge vs origin cost like you can with CloudFront + ALB separately.
> - Some advanced features exist only on the CloudFront side (Lambda@Edge equivalents like Cloud Functions on the edge exist but are less mature).
>
> Practical recommendation: for teams starting fresh on GCP with a global web app, use this LB. It's cleaner and faster to build. For teams porting from AWS with heavy CloudFront customization, plan the architecture more carefully — some patterns don't translate 1:1."

---
---

# 7. MIG — Managed Instance Groups

## Concept

Managed Instance Group (MIG) is GCP's autoscaling group primitive — a fleet of identical VMs from a template, with autoscaling, autohealing, and rolling updates.

## AWS Equivalent

**MIG = AWS Auto Scaling Group + Launch Template combined**

## Azure Equivalent

**MIG = Azure Virtual Machine Scale Set (VMSS)**

## Three-Way Comparison

| Feature | AWS ASG | Azure VMSS | GCP MIG |
|---|---|---|---|
| Template resource | Launch Template | (embedded in VMSS) | Instance Template |
| Scaling policies | Target Tracking + Step + Scheduled | Autoscale profiles + Application Insights metrics | Autoscaler + custom metrics |
| Health check | ELB or EC2 | Application Health Extension | HTTP/TCP/SSL health check |
| Zone spread | Multi-AZ via subnet mix | Zones flag | Zonal MIG or Regional MIG |
| Rolling update | Instance Refresh | Rolling upgrade | Rolling update (built-in) |
| Spot integration | Mixed instance policy | Spot priority in pool | Spot VMs in MIG |

## Key Differences from BOTH

**1. Regional vs Zonal MIGs**

Two flavors:
- **Zonal MIG** — all instances in one zone. Simpler, no HA.
- **Regional MIG** — instances spread across 3 zones automatically. HA by default.

**Always use Regional MIGs for production.** The overhead is minimal.

AWS: ASG can span multiple AZs via subnet configuration.
Azure: VMSS with `--zones 1 2 3` spans zones.
GCP: it's a first-class Regional MIG option — cleaner.

**2. Autohealing**

If a VM fails a health check, MIG kills and replaces it. Configurable initial delay to allow startup time.

Same feature exists in AWS ASG and Azure VMSS. GCP's is slightly cleaner to configure — one health check spec used for both LB health and autohealing.

**3. Instance Templates Are Versioned**

You create a new Instance Template for changes, then set the MIG to use the new version. Rolling updates handle the transition automatically. Old template stays around for rollback.

AWS Launch Templates are versioned similarly. Azure VMSS uses "model" versioning.

## Real Configuration

```bash
# 1. Create an instance template
gcloud compute instance-templates create web-template-v1 \
  --machine-type=n2-standard-2 \
  --image-family=debian-12 \
  --image-project=debian-cloud \
  --tags=web-server \
  --service-account=web-sa@myapp-prod.iam.gserviceaccount.com \
  --scopes=cloud-platform \
  --metadata=startup-script='#!/bin/bash
    apt-get update && apt-get install -y nginx
    systemctl start nginx'

# 2. Create a health check
gcloud compute health-checks create http web-health-check \
  --port=80 \
  --request-path=/ \
  --check-interval=10s \
  --healthy-threshold=2 \
  --unhealthy-threshold=3

# 3. Create Regional MIG
gcloud compute instance-groups managed create web-mig \
  --template=web-template-v1 \
  --size=3 \
  --region=us-central1 \
  --health-check=web-health-check \
  --initial-delay=180

# 4. Set autoscaling
gcloud compute instance-groups managed set-autoscaling web-mig \
  --region=us-central1 \
  --min-num-replicas=3 \
  --max-num-replicas=20 \
  --target-cpu-utilization=0.6 \
  --cool-down-period=90

# 5. Rolling update to new template
gcloud compute instance-templates create web-template-v2 \
  --machine-type=n2-standard-4 \
  --image-family=debian-12 \
  --image-project=debian-cloud \
  --service-account=web-sa@myapp-prod.iam.gserviceaccount.com \
  --scopes=cloud-platform

gcloud compute instance-groups managed rolling-action start-update web-mig \
  --version=template=web-template-v2 \
  --region=us-central1 \
  --max-surge=3 \
  --max-unavailable=0
```

## Production Use Case

**Auto-scaling web tier with mixed on-demand + Spot:**

```
Regional MIG: web-mig (us-central1, 3 zones)
├── Instance template v3 (n2-standard-4, cos-stable image, container-based app)
├── Autoscaling:
│   ├── Target CPU: 60%
│   ├── Min: 3, Max: 30
│   ├── Cool-down: 90s
│   └── Custom metric: HTTP requests/sec > 200 → scale out
├── Health check: HTTP /health on 8080, 10s interval
├── Autoheal: if 3 consecutive failures, replace VM
├── Update policy: rolling, max-surge 3, max-unavailable 0 (no downtime)
└── Frontend: attached to Global External Application LB as backend
```

## Console Path

- Console → **Compute Engine** → **Instance groups**
- Create MIG wizard → template, region, size, autoscaling policy, health check
- Existing MIG: **Details** → instance list, autoscaling, autohealing, updates history

## Memory Hint

- **MIG = ASG + Launch Template combined** (like VMSS)
- **Regional MIG** = HA default (3 zones), always use for prod
- **Zonal MIG** = single zone (dev/test only)
- **Instance Template** = versioned VM spec
- **Rolling update** built-in — no separate deploy tool needed

## Interview Q&A — MIG

**Q: Compare AWS ASG, Azure VMSS, and GCP MIG. Which is easiest to operate?**

> "Same category, similar features, subtle usability differences.
>
> Configuration overhead: MIG and VMSS both bundle template + scaling into one resource. AWS ASG separates Launch Template from ASG which is more composable but more moving parts to manage.
>
> Rolling updates: GCP MIG has rolling updates as a first-class operation with `rolling-action start-update`. Very clean. AWS ASG has Instance Refresh — also good but slightly more configuration. Azure VMSS has rolling upgrades but the mental model is a bit different.
>
> Multi-zone spread: Regional MIG is a checkbox — 'spread across zones.' AWS ASG requires you to specify subnets across AZs. Azure VMSS uses `--zones 1 2 3`. All achievable, GCP is slightly cleaner.
>
> Mixed on-demand + Spot: AWS ASG has mixed instance policy with instance overrides. Azure VMSS Flexible has similar. GCP MIG supports Spot VMs alongside regular VMs. Feature parity.
>
> Autohealing: MIG and VMSS both have health-check-driven replacement. AWS ASG relies on EC2 or ELB health checks. All three converge on similar behavior.
>
> My honest take: for a greenfield project, MIG is marginally simpler to configure. For an inherited fleet, whichever the team already uses is fine — they're comparable in production."

---
---

# 8. Cloud SQL

## Concept

Cloud SQL is GCP's managed relational database service for MySQL, PostgreSQL, and SQL Server.

## AWS Equivalent

**Cloud SQL = AWS RDS**

## Azure Equivalent

**Cloud SQL ≈ Azure Database for MySQL/PostgreSQL** (for those engines) OR **Azure SQL Database** (for SQL Server)

## Three-Way Comparison

| Feature | AWS RDS | Azure Database | Cloud SQL |
|---|---|---|---|
| Engines | MySQL, Postgres, MariaDB, SQL Server, Oracle | MySQL, Postgres, SQL Server (via Azure SQL) | MySQL, Postgres, SQL Server |
| HA model | Multi-AZ (sync standby) | Zone-redundant HA | Regional HA (sync replica in another zone) |
| Read replicas | Cross-region supported | Read replicas supported | Same-region or cross-region |
| Backup | Automated + manual snapshots | Automated + LTR | Automated + on-demand |
| Aurora equivalent | Aurora (custom) | Hyperscale/Business Critical | AlloyDB (Postgres-compatible, distinct product) |

## Key Differences from BOTH

**1. AlloyDB — the Postgres "Aurora Killer"**

GCP has a separate service: **AlloyDB for PostgreSQL**. It's a Postgres-compatible service with columnar accelerator, 4x faster than standard Postgres for transactional workloads, 100x faster for analytical queries, ~2x faster than AWS Aurora Postgres in Google's benchmarks.

Not covered under Cloud SQL — it's a distinct product. If you're doing Postgres at scale on GCP and RDS-style tuning isn't enough, AlloyDB is the answer. Similar architecture to Aurora (compute/storage separation).

**2. Regional Availability**

Cloud SQL HA uses a sync standby in a different zone within the same region. Like AWS RDS Multi-AZ. Failover is automatic and fast (~1 min).

**3. Public IP by Default (Change This)**

New Cloud SQL instances default to having a public IP. **Turn this off** and use Private IP + VPC Peering for internal access. Same principle as never exposing an RDS instance to the internet.

**4. Cloud SQL Auth Proxy**

Unique tool: `cloud-sql-proxy` runs locally or on a VM, authenticates to GCP via a Service Account, then presents a local socket that your app connects to. The proxy handles SSL and IAM auth transparently.

Great for: dev machines connecting to prod DBs securely, apps on Kubernetes that don't want to hardcode DB IPs, avoiding SSL certificate management.

AWS RDS Proxy is somewhat similar but positioned differently (mainly for connection pooling).

**5. IAM Authentication for Databases**

You can grant IAM users/service accounts direct database access — no separate DB password. `gcloud sql users create ... --type=cloud_iam_service_account` creates a DB user tied to a GCP identity.

Similar to AWS RDS IAM authentication and Azure Entra ID authentication for Azure SQL. GCP's setup is arguably simplest.

## Real Configuration

```bash
# Create a Postgres instance (development tier)
gcloud sql instances create prod-postgres \
  --project=myapp-prod \
  --database-version=POSTGRES_15 \
  --tier=db-custom-2-8192 \
  --region=us-central1 \
  --root-password='StrongPassword2026!' \
  --storage-size=100GB \
  --storage-type=SSD \
  --availability-type=REGIONAL \
  --no-assign-ip \
  --network=projects/myapp-prod/global/networks/prod-vpc

# Create a database
gcloud sql databases create appdb --instance=prod-postgres

# Create a user
gcloud sql users create appuser \
  --instance=prod-postgres \
  --password='UserPassword2026!'

# Create an IAM-based user (Service Account can auth directly)
gcloud sql users create web-sa@myapp-prod.iam \
  --instance=prod-postgres \
  --type=cloud_iam_service_account

# Configure automatic backups + point-in-time recovery
gcloud sql instances patch prod-postgres \
  --backup-start-time=03:00 \
  --enable-point-in-time-recovery \
  --retained-backups-count=30 \
  --retained-transaction-log-days=7

# Connect via Cloud SQL Auth Proxy (from any machine with gcloud auth)
cloud-sql-proxy --auto-iam-authn myapp-prod:us-central1:prod-postgres &
psql -h 127.0.0.1 -U web-sa@myapp-prod.iam -d appdb
```

## Production Use Case

**Production Postgres with HA + read replica for reporting:**

```
Project: myapp-prod-data
├── Cloud SQL: prod-postgres
│   ├── Version: Postgres 15
│   ├── Tier: db-custom-4-16384 (4 vCPU, 16GB RAM)
│   ├── Storage: 500GB SSD, auto-increase enabled
│   ├── HA: Regional (sync replica in us-central1-b)
│   ├── Backups: daily 03:00, 30-day retention, PITR enabled
│   ├── Network: Private IP only, VPC peering to prod-vpc
│   ├── Encryption: Google-managed keys (or CMEK for compliance)
│   └── IAM auth: web-sa@myapp-prod (least-privilege DB roles)
│
├── Read Replica: prod-postgres-reporting
│   ├── Region: us-central1 (same region, async replication)
│   └── Used by BI/analytics tools only
│
└── App tier connects via:
    ├── Cloud SQL Auth Proxy sidecar on Cloud Run
    ├── Or Private IP directly for VM-based apps
    └── Reporting queries use ApplicationName hint to route to replica
```

## Console Path

- Console → **SQL** → **Instances**
- Instance detail: Overview, Connections, Users, Databases, Backups, Replicas, Operations
- **Cloud SQL Studio** — in-browser SQL editor (recent addition, similar to Azure SQL Query Editor)

## Memory Hint

- **Cloud SQL** = "RDS but with the Auth Proxy for identity-based access"
- **AlloyDB** = Aurora Postgres equivalent (separate product, mention if asked)
- **Regional HA** = sync replica in another zone (like RDS Multi-AZ)
- **Auth Proxy** = unique GCP tool, worth knowing by name
- **Turn off public IP** on new instances immediately

## Interview Q&A — Cloud SQL

**Q: Design a highly available Postgres deployment on GCP. Compare to what you'd do on AWS and Azure.**

> "Cloud SQL Postgres with Regional (HA) availability. Setup:
>
> Instance tier: db-custom-4-16384 or higher based on load — Cloud SQL supports custom vCPU/RAM shapes like Compute Engine.
> Availability type: Regional — this provisions a synchronous standby in a different zone. Automatic failover in ~60 seconds if the primary zone fails.
> Storage: SSD with auto-increase enabled (up to configured max), 30-day PITR retention.
> Networking: Private IP only, exposed via VPC peering to the app VPC. No public endpoint.
> Auth: IAM database authentication with dedicated Service Accounts per app tier. No shared passwords.
> Backup: automated daily backups, PITR enabled for granular restore.
>
> Cross-cloud comparison:
>
> AWS RDS Postgres Multi-AZ is the direct analogue. Same architecture (sync standby, automatic failover), similar cost profile. RDS has been production-battle-tested longer, ecosystem is deeper. Cloud SQL is comparable in feature set for standard Postgres workloads.
>
> Azure Database for PostgreSQL Flexible Server with zone-redundant HA is the Azure equivalent. Feature parity roughly, cost model slightly different.
>
> Where GCP wins: if you need scale beyond what standard Postgres handles, AlloyDB for Postgres exists as a distinct service — think of it as GCP's Aurora Postgres. Compute-storage separation, columnar accelerator for OLAP, 2x faster than Aurora Postgres in benchmarks. If your workload is Postgres-heavy and needs Aurora-like scaling, GCP has a stronger story than raw RDS Postgres or Azure Flexible Server.
>
> For a straightforward Postgres deployment though, all three clouds deliver production-grade managed Postgres. Choose based on cloud-native integration (which cloud does the rest of your stack live in?), operational familiarity, and cost per your specific access patterns."

---
---

# 9. Firestore, Bigtable, Spanner

GCP splits what AWS calls "DynamoDB" into **three distinct products**. Each solves different problems. This section is where GCP's data offerings genuinely diverge from AWS and Azure — Spanner especially has no direct equivalent.

## The Three Products at a Glance

| Product | Type | Use Case | AWS Analogue | Azure Analogue |
|---|---|---|---|---|
| **Firestore** | Document DB | Mobile/web apps, small-medium scale | DynamoDB (basic) | Cosmos DB (NoSQL API) |
| **Bigtable** | Wide-column | Massive time-series, IoT, analytics | DynamoDB (scale) / Keyspaces | Cosmos DB (Cassandra API) |
| **Spanner** | Distributed SQL | Global, strongly-consistent SQL | Aurora Global / DynamoDB Global Tables | Cosmos DB (with strong consistency) |

---

## 9a. Firestore

**Concept:** Serverless document database, mobile/web-friendly, real-time listeners, offline sync.

**Two modes at creation time (cannot change later):**
- **Native mode** — modern, supports real-time listeners, better client SDKs
- **Datastore mode** — legacy Datastore API, backward compat

**For anything new: Native mode.**

### Key Features

- Document/collection model (like MongoDB or Cosmos DB Document API)
- Real-time listeners — clients subscribe, get pushed updates via WebSocket-like connection
- Offline persistence in mobile SDKs (queue writes offline, sync when back online)
- Serverless — no capacity provisioning
- Auto-scaling reads and writes

### AWS/Azure Comparison

DynamoDB is closer to Firestore than to Bigtable or Spanner — same "managed NoSQL for apps" niche. But DynamoDB doesn't have real-time listeners; you'd add DynamoDB Streams + Lambda + WebSocket API Gateway to fake it.

Azure Cosmos DB (SQL API) is Firestore's closest Azure equivalent, but Cosmos also has WebSocket-like Change Feed which requires more plumbing than Firestore's built-in listeners.

Firestore's real-time story is genuinely differentiated for mobile/web apps.

### Real Configuration

```bash
# Enable Firestore in Native mode (one-time per project)
gcloud firestore databases create --location=us-central1

# Firestore is typically used via SDK, not gcloud commands
# Example writes via curl (using access token):
TOKEN=$(gcloud auth application-default print-access-token)
curl -X POST \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  --data '{"fields":{"name":{"stringValue":"Aakash"},"role":{"stringValue":"DevOps"}}}' \
  "https://firestore.googleapis.com/v1/projects/myapp-prod/databases/(default)/documents/users/user123"
```

### Interview Q&A — Firestore

**Q: A team is building a mobile app with a chat feature. Firestore, DynamoDB, or Cosmos DB?**

> "Firestore, unambiguously. Real-time listeners are built in — chat clients subscribe to a message collection and receive updates as they happen with no polling and no additional infrastructure. Offline persistence handles poor mobile connectivity gracefully.
>
> DynamoDB requires you to build a real-time layer on top: DynamoDB Streams → Lambda → API Gateway WebSockets → clients. More moving parts, more failure modes, more code.
>
> Cosmos DB has Change Feed which is similar to Firestore under the hood, but you still need to build a WebSocket layer on top for client push. Not as turnkey.
>
> Firestore's cost model favors this use case too — you pay per document read/write plus network bandwidth. For chat apps with moderate message rates, cost is predictable and low.
>
> Where Firestore falls short: complex queries. It supports basic filters and ordering but no JOINs, no aggregate queries beyond count. If your data model needs SQL-like flexibility, look elsewhere. But for chat, notification feeds, real-time dashboards — Firestore is the shortest path from idea to production."

---

## 9b. Bigtable

**Concept:** Petabyte-scale wide-column NoSQL, HBase-compatible API. For massive time-series, IoT sensor data, analytics workloads with high throughput.

### Key Features

- Petabyte scale, single-digit millisecond latency at that scale
- HBase-compatible API (Java clients, HBase shell all work)
- Cluster-based — you provision nodes (unlike serverless Firestore)
- Multi-cluster replication across regions (async or global)
- Integrates natively with Dataflow, BigQuery, Dataproc for analytics

### AWS/Azure Comparison

Closest AWS equivalent: DynamoDB at massive scale (but different API and data model). AWS Keyspaces (Managed Cassandra) is architecturally closer.
Closest Azure equivalent: Cosmos DB Cassandra API.

**Bigtable is the go-to for genuine massive-scale time-series and analytics workloads.** Google uses it internally for search indexing, Gmail, Ads — it's battle-tested at their scale.

### When to Choose Bigtable

- Time-series at scale (financial ticks, IoT sensors, monitoring metrics)
- Ad-tech / analytics workloads with billions of rows
- Existing HBase applications migrating to cloud (drop-in compat)

**When NOT to choose Bigtable:**
- Small datasets (below ~1TB) — expensive overhead
- Transactional workloads needing SQL semantics — use Spanner
- Simple app databases — use Firestore

### Real Configuration

```bash
# Create a Bigtable instance
gcloud bigtable instances create prod-metrics \
  --display-name="Production Metrics" \
  --cluster-config=id=prod-metrics-c1,zone=us-central1-a,nodes=3 \
  --cluster-config=id=prod-metrics-c2,zone=us-east1-a,nodes=3 \
  --instance-type=PRODUCTION

# Create a table
cbt -project=myapp-prod -instance=prod-metrics createtable events \
  "families=cf1:maxversions=1"

# Write via cbt (Bigtable's admin CLI)
cbt -project=myapp-prod -instance=prod-metrics set events row1 cf1:count=42
```

---

## 9c. Spanner

**Concept:** Globally distributed, horizontally scalable, strongly consistent SQL database. **No other major cloud has an equivalent.** This is arguably GCP's biggest technical differentiator.

### Key Features

- SQL interface (Postgres and GoogleSQL dialects)
- ACID transactions across globally distributed data
- 99.999% availability SLA (five nines)
- Horizontal scaling — add nodes, capacity increases linearly
- Automatic sharding, no manual partitioning
- Uses Google's TrueTime API (atomic clock + GPS) to achieve external consistency across regions

### The Uniqueness

The CAP theorem says you can't have Consistency + Availability during a network Partition. Spanner effectively achieves both because Google's infrastructure has such low probability of partition that they can guarantee both under normal conditions.

No AWS or Azure product does this. AWS Aurora Global has async cross-region replication (eventually consistent). Cosmos DB has multi-region writes but with different consistency trade-offs. Spanner is genuinely different — strongly consistent globally, at scale.

### AWS/Azure Comparison

| Feature | AWS Aurora Global | Azure Cosmos DB (Strong) | GCP Spanner |
|---|---|---|---|
| Global writes | No (single writer region) | Yes (limited by consistency) | Yes, fully consistent |
| Cross-region consistency | Eventual | Bounded staleness or strong (single region only) | Strong, external |
| SQL interface | Yes (MySQL/Postgres) | No (Cosmos SQL is limited) | Yes (Postgres or GoogleSQL) |
| Horizontal scaling | Manual sharding | Automatic (partition key) | Automatic |
| Cost | Moderate | High | High (but unique capability) |

### When to Choose Spanner

- Financial systems where global consistency matters (payments, ledgers, inventory)
- Global multi-tenant SaaS where a user in Europe and one in Asia hit the same data with strict consistency
- Migrating from a sharded MySQL/Postgres setup that's become operationally painful

**When NOT to choose Spanner:**
- Small workloads — it's expensive
- Workloads that fit comfortably in a single-region Postgres — use Cloud SQL or AlloyDB

### Real Configuration

```bash
# Create a Spanner instance
gcloud spanner instances create prod-spanner \
  --config=regional-us-central1 \
  --description="Production Spanner" \
  --nodes=1

# For multi-region:
gcloud spanner instances create prod-spanner-global \
  --config=nam-eur-asia1 \
  --nodes=3

# Create a database
gcloud spanner databases create appdb \
  --instance=prod-spanner \
  --ddl-file=schema.sql

# Query via CLI
gcloud spanner databases execute-sql appdb \
  --instance=prod-spanner \
  --sql="SELECT * FROM users LIMIT 10"
```

### Interview Q&A — When to Use Spanner

**Q: A global fintech needs strongly consistent transactions across regions. Walk through GCP vs AWS vs Azure options.**

> "This is Spanner's canonical use case. Financial transactions require ACID guarantees — no eventual consistency, no read-your-writes anomalies, no partition tolerance shortcuts.
>
> GCP Spanner: designed exactly for this. Multi-region config (say `nam-eur-asia1`), all writes are strongly consistent globally. A payment initiated in Europe is immediately visible in Asia with correct balance. SQL interface, so existing SQL knowledge transfers.
>
> AWS: Aurora Global Database gives you cross-region reads, but writes are single-region. Failover across regions is 1-2 minutes with some data loss risk. For active-active global consistency, AWS doesn't have a direct equivalent. Some teams use DynamoDB Global Tables with careful design, but you're giving up strong consistency at global scale.
>
> Azure: Cosmos DB with Strong consistency works within a single region, or with 'Bounded Staleness' cross-region. Not equivalent to Spanner's external consistency.
>
> The catch with Spanner: it's expensive. Single-region minimum is ~$1000/month. Multi-region starts around $3000/month. Only makes sense for workloads that genuinely need this capability.
>
> Practical decision: if the workload TRULY needs global strong consistency for correctness reasons (financial systems, inventory, regulatory), Spanner is worth the cost and is the only cloud service that delivers. If you can accept eventual consistency with careful application design, AWS or Azure are cheaper.
>
> Interview signal: knowing Spanner exists AND knowing when NOT to use it (which is most of the time) is the senior answer."

---
---

# 10. Cloud Functions & Cloud Run

GCP has **two distinct serverless compute products**, and understanding both is critical for interviews because **Cloud Run has no direct equivalent in AWS or Azure** at the same level of maturity.

---

## 10a. Cloud Functions

**Concept:** Event-driven, code-based serverless. Deploy a function, GCP runs it in response to triggers.

### AWS Equivalent

**Cloud Functions = AWS Lambda**

### Azure Equivalent

**Cloud Functions = Azure Functions**

### Three-Way Comparison

| Feature | AWS Lambda | Azure Functions | Cloud Functions (2nd gen) |
|---|---|---|---|
| Trigger types | Broad (S3, SQS, API Gateway, etc.) | Broad (Blob, Cosmos, Queue, HTTP) | HTTP + Pub/Sub + Storage + Eventarc |
| Cold start | Fast for JS/Python, slow for JVM | Fast for JS/Python, slow for JVM | Similar |
| Max duration | 15 minutes | 10 min Consumption / unlimited Premium | 60 min (2nd gen) |
| Memory | Up to 10GB | Up to 14GB (Premium) | Up to 32GB (2nd gen) |
| Pricing | Requests + GB-second | Requests + GB-second | Requests + GB-second |

### Cloud Functions 1st Gen vs 2nd Gen

Two versions exist:
- **1st gen** — original, simpler, fewer features. Legacy.
- **2nd gen** — built on Cloud Run under the hood, more features (longer duration, more concurrency per instance, better scaling).

**Always use 2nd gen for new work.** It's essentially Cloud Run with function-style deployment ergonomics.

### Real Configuration

```bash
# Deploy a Python HTTP function (2nd gen)
gcloud functions deploy hello-world \
  --gen2 \
  --runtime=python312 \
  --region=us-central1 \
  --source=. \
  --entry-point=hello_world \
  --trigger-http \
  --allow-unauthenticated \
  --memory=512Mi

# Deploy triggered by Cloud Storage event
gcloud functions deploy process-upload \
  --gen2 \
  --runtime=python312 \
  --region=us-central1 \
  --source=. \
  --entry-point=process_upload \
  --trigger-event-filters="type=google.cloud.storage.object.v1.finalized" \
  --trigger-event-filters="bucket=myapp-prod-uploads" \
  --service-account=processing-sa@myapp-prod.iam.gserviceaccount.com
```

---

## 10b. Cloud Run

**Concept:** Fully managed serverless containers. Deploy a container image, GCP runs it in response to HTTP requests, auto-scales to zero when idle, scales up to thousands of instances under load.

### AWS Equivalent

**Cloud Run ≈ AWS App Runner** (approximately, but Cloud Run is more mature and flexible)

Also related to AWS Fargate for containers, but Fargate is minimum-instance-based (not scale-to-zero).

### Azure Equivalent

**Cloud Run ≈ Azure Container Apps** (approximately, but Cloud Run is simpler and more mature)

### Why Cloud Run Is Genuinely Different

Cloud Run's pricing model, developer experience, and scaling behavior are best-in-class among managed container services. Key differentiators:

**1. True scale-to-zero.** Idle service costs $0. You pay per request + CPU-seconds during requests. AWS App Runner has some scale-to-zero support but with cold-start caveats. Container Apps supports scale-to-zero but is more complex to configure.

**2. Request-based pricing.** Cloud Run bills per 100ms of CPU used per request. Not per instance-hour. For sporadic traffic, this is dramatically cheaper than running containers 24/7.

**3. Any container works.** As long as your container listens on the port from `$PORT` env var, it runs. No AWS Lambda base image requirements. No custom runtimes. Docker image, deploy, done.

**4. Concurrency per instance is configurable.** By default one Cloud Run instance handles 80 concurrent requests. Not 1 request per instance like Lambda. This makes containerized apps much more efficient than function-per-request.

**5. Sidecar containers supported.** You can deploy multiple containers per Cloud Run service — one main container plus sidecars (like a Kubernetes pod). Useful for auth proxies, logging agents.

### Cloud Run Modes

Two flavors:
- **Cloud Run services** — long-lived, HTTP-triggered, request-driven scaling
- **Cloud Run jobs** — batch execution, run once, exit (like Kubernetes Jobs)

### Real Configuration — Cloud Run Service

```bash
# Build and push container to Artifact Registry
gcloud builds submit --tag us-central1-docker.pkg.dev/myapp-prod/repo/api:v1

# Deploy to Cloud Run
gcloud run deploy api \
  --image=us-central1-docker.pkg.dev/myapp-prod/repo/api:v1 \
  --region=us-central1 \
  --platform=managed \
  --service-account=api-sa@myapp-prod.iam.gserviceaccount.com \
  --allow-unauthenticated \
  --port=8080 \
  --memory=512Mi \
  --cpu=1 \
  --min-instances=0 \
  --max-instances=100 \
  --concurrency=80 \
  --timeout=60s \
  --set-env-vars="ENV=production,DB_HOST=10.0.20.5" \
  --vpc-connector=projects/myapp-prod/locations/us-central1/connectors/serverless-vpc

# Update to new version (blue-green style via revisions)
gcloud run deploy api \
  --image=us-central1-docker.pkg.dev/myapp-prod/repo/api:v2 \
  --region=us-central1 \
  --no-traffic  # Deploy new revision without sending traffic

# Gradually shift traffic
gcloud run services update-traffic api \
  --region=us-central1 \
  --to-revisions=api-v2=20,api-v1=80

# Full cutover
gcloud run services update-traffic api \
  --region=us-central1 \
  --to-latest
```

### Production Use Case

**Full application as Cloud Run services:**

```
Project: myapp-prod
├── Cloud Run service: web (frontend, allow-unauthenticated)
│   ├── Container: nginx serving React build
│   ├── Concurrency: 250 (mostly static file serving)
│   ├── Min instances: 1 (always warm for fast first response)
│   └── Custom domain: www.aakashrao.dev via domain mapping
│
├── Cloud Run service: api (backend, require auth)
│   ├── Container: FastAPI app
│   ├── Concurrency: 80 (mix of CPU-heavy and I/O)
│   ├── Min instances: 0 (scale to zero when idle)
│   ├── VPC connector: for Cloud SQL private IP access
│   └── Service Account: api-sa with Cloud SQL + Storage roles
│
├── Cloud Run job: nightly-etl
│   ├── Runs at 03:00 UTC via Cloud Scheduler
│   ├── Container: dbt runner
│   └── Timeout: 1 hour, max 3 retries
│
└── Ingress: Global External Application LB routes:
    ├── /*  → web service (via serverless NEG)
    ├── /api/*  → api service (via serverless NEG)
    └── Cloud CDN caches /static/* on the LB
```

Zero infrastructure to manage. No Kubernetes cluster. No VMs. No autoscaling policies. Deploy, done.

### Cloud Run vs Cloud Functions — Which to Pick?

| Consideration | Cloud Functions | Cloud Run |
|---|---|---|
| Language flexibility | Fixed runtimes (Python 3.12, Node 20, etc.) | Any container |
| Portability | GCP-only | Container-native, portable |
| Complexity | Low (single file works) | Slightly higher (Dockerfile needed) |
| Concurrency per instance | 1 (each request = new instance) | Configurable (up to 1000) |
| Max duration | 60 min (2nd gen) | 60 min (services), unlimited (jobs) |
| Custom system deps | Limited | Unlimited (any container) |

**Rule of thumb:**
- **Event-driven glue code, simple HTTP handlers** → Cloud Functions
- **Anything containerized, any real application** → Cloud Run
- **When in doubt, Cloud Run** — it's more flexible and 2nd gen Functions is Cloud Run under the hood anyway

### Console Path

- Console → **Cloud Run** → **Services** or **Jobs**
- Service detail: **Revisions**, **Metrics**, **Logs**, **Triggers**, **Networking**, **Security**
- Traffic splitting: **Manage Traffic** button on service detail

### Memory Hint

- **Cloud Functions 2nd gen** = Lambda equivalent (fixed runtimes)
- **Cloud Run** = "AWS App Runner + Azure Container Apps but done right"
- **Scale to zero** by default (cost efficiency)
- **Request-based pricing** — pay per CPU used, not per instance-hour
- **Concurrency per instance** = configurable (game-changer vs Lambda's 1-per-instance)
- **Any container works** — Dockerfile is enough

### Interview Q&A — Cloud Run

**Q: A team is running dockerized microservices on ECS Fargate. Would you migrate to Cloud Run? Why or why not?**

> "Depends on their access patterns and integration needs, but Cloud Run is often a strong win.
>
> When Cloud Run beats Fargate:
>
> First, cost for spiky or sporadic workloads. Fargate has minimum instance count and charges per CPU/RAM-hour of allocated capacity. Cloud Run bills per 100ms of CPU actually used per request AND supports scale-to-zero. A service with 100 requests/day pays essentially $0 on Cloud Run and $50-100/month on Fargate.
>
> Second, developer experience for simple deployments. `gcloud run deploy --image=...` and you're live. No task definitions, service definitions, ALB target groups, security groups. One command.
>
> Third, concurrency per instance. Cloud Run defaults to 80 concurrent requests per instance. Fargate is one request handled per instance thread. For I/O-bound apps, Cloud Run is dramatically more efficient with fewer instances.
>
> When Fargate might win:
>
> AWS-native integrations. If the service depends on other AWS services with tight IAM integration, staying on Fargate reduces cross-cloud complexity.
>
> Long-running or high-concurrency workloads at steady load. Fargate's pricing per allocated capacity can be cheaper than Cloud Run's per-request billing if you're at 100% utilization 24/7.
>
> Specific networking needs. Fargate integrates deeply with VPC networking, service discovery, service mesh. Cloud Run has serverless VPC connectors and Direct VPC egress but the model is different.
>
> Practical recommendation: for greenfield services on a fresh GCP project, Cloud Run is the default. For migrating existing Fargate services with AWS integrations, evaluate case-by-case. The Cloud Run developer experience is worth optimizing for even if there's some transition cost."

**Q: Compare Cloud Run to Azure Container Apps and AWS App Runner.**

> "All three are managed container services with auto-scaling. The nuances matter.
>
> Cloud Run: the most mature and simplest. Deploy a container, done. Scale-to-zero works, request-based pricing works, concurrency-per-instance is configurable. Been GA since 2019 and battle-tested.
>
> Azure Container Apps: newer (2022 GA), built on Kubernetes + KEDA under the hood. More features exposed (Dapr, KEDA scaling on any metric, revisions with traffic splitting). Also more concepts to learn — environments, replicas, scale rules. If you need advanced scaling on custom metrics like queue depth or Kafka lag, Container Apps has richer primitives.
>
> AWS App Runner: newest of the three (2021 GA), simplest of the three. Deploy from container registry or from source code repo. Scales based on requests. Less feature-rich than Cloud Run or Container Apps — no traffic splitting, less configurability. Good for simple web services, less good for anything complex.
>
> Rough ranking by developer experience: Cloud Run > App Runner > Container Apps for simple cases. Cloud Run > Container Apps > App Runner for advanced use cases.
>
> Portfolio tip: build a demo project on Cloud Run and mention it in interviews as evidence of cross-cloud fluency. It's a service that's genuinely fun to use and easy to demonstrate."

---
---

# 11. IaC — Deployment Manager, Config Connector, Terraform

## Concept

Infrastructure as Code on GCP.

## AWS Equivalent

**Deployment Manager ≈ CloudFormation** (both being deprecated in favor of Terraform-first approaches)

## Azure Equivalent

**Deployment Manager ≈ ARM templates** (raw); **Terraform is the modern choice on all three**

## The Reality: Use Terraform

GCP's Deployment Manager was Google's original IaC tool. It's **being deprecated** — end of life announced for 2025-2026. Not a good choice for new projects.

**Modern GCP IaC options:**
1. **Terraform** (with GCP provider) — the practical choice, works cross-cloud
2. **Config Connector** — Kubernetes-based IaC, provisions GCP resources via K8s CRDs (unique but niche)
3. **Cloud Foundation Toolkit** — Terraform modules maintained by Google for common patterns
4. **Deployment Manager** — legacy, don't start here

## Terraform on GCP — The Practical Choice

Same HCL syntax as AWS/Azure. Provider is `google` (or `google-beta` for newer features).

```hcl
terraform {
  required_providers {
    google = {
      source  = "hashicorp/google"
      version = "~> 5.0"
    }
  }
  
  backend "gcs" {
    bucket = "myapp-terraform-state"
    prefix = "prod"
  }
}

provider "google" {
  project = "myapp-prod"
  region  = "us-central1"
}

resource "google_storage_bucket" "uploads" {
  name          = "myapp-prod-user-uploads"
  location      = "us-central1"
  storage_class = "STANDARD"
  
  uniform_bucket_level_access = true
  
  lifecycle_rule {
    action { type = "SetStorageClass"; storage_class = "NEARLINE" }
    condition { age = 30 }
  }
}

resource "google_cloud_run_v2_service" "api" {
  name     = "api"
  location = "us-central1"
  
  template {
    containers {
      image = "us-central1-docker.pkg.dev/myapp-prod/repo/api:v1"
      resources {
        limits = { cpu = "1", memory = "512Mi" }
      }
    }
    service_account = google_service_account.api.email
    scaling {
      max_instance_count = 100
    }
  }
}
```

Same workflow: `terraform init`, `plan`, `apply`, `destroy`. State stored in a GCS bucket (like S3 backend for AWS Terraform, or Azure Storage for Azure).

## Config Connector — The Niche But Cool Option

Config Connector lets you manage GCP resources via Kubernetes CRDs (Custom Resource Definitions). Install it in your GKE cluster, and you can write:

```yaml
apiVersion: storage.cnrm.cloud.google.com/v1beta1
kind: StorageBucket
metadata:
  name: myapp-prod-user-uploads
  namespace: default
spec:
  location: us-central1
  storageClass: STANDARD
  uniformBucketLevelAccess: true
```

Apply with `kubectl apply -f`, GCP resources get created. GitOps for GCP infrastructure — same tools you use for K8s manifests (ArgoCD, Flux) now manage GCP resources.

Niche but powerful for GKE-centric teams. AWS has Controllers for Kubernetes (ACK), Azure has ASO (Azure Service Operator). Same pattern.

## Three-Way IaC Comparison

| Approach | AWS | Azure | GCP |
|---|---|---|---|
| Native declarative | CloudFormation | ARM / Bicep | Deployment Manager (dying) |
| Native CDK-style | CDK (multi-lang) | Bicep | (nothing well-adopted) |
| Terraform | Excellent support | Excellent support | Excellent support |
| K8s-based | Controllers for Kubernetes (ACK) | Azure Service Operator | Config Connector |

**Recommendation across all three clouds: Terraform.** Best cross-cloud investment. Ecosystem is huge. Portable skills.

## Interview Q&A — IaC

**Q: What's your IaC recommendation for a team doing multi-cloud on AWS, Azure, and GCP?**

> "Terraform, unambiguously. Zero debate.
>
> Reasoning: any multi-cloud team benefits from one tool that speaks all three clouds. Terraform's provider ecosystem covers not just AWS/Azure/GCP but also GitHub, Cloudflare, Datadog, Snowflake, and hundreds of others. State management, modules, and workflows work identically across providers.
>
> Alternative approaches all have gaps:
> - CloudFormation is AWS-only
> - Bicep is Azure-only
> - Deployment Manager is GCP-only AND being deprecated
> - CDK is AWS-first with Azure/GCP support that's less mature
>
> Pulumi is worth mentioning as a Terraform alternative — real programming languages instead of HCL. Some teams love it. Terraform is more established and has larger community.
>
> Structure recommendation for multi-cloud Terraform:
> - Separate root modules per cloud per environment: `envs/prod/aws/`, `envs/prod/azure/`, `envs/prod/gcp/`
> - Shared modules for common patterns (bucket, DB, cluster)
> - State backends per cloud in that cloud's native storage (S3, Azure Storage, GCS)
> - Automation via Atlantis, Terraform Cloud, or Spacelift for PR-based workflows
>
> Interview signal: candidates who dogmatically insist on native tools (CloudFormation-only, Bicep-only) look inflexible. Senior answer acknowledges native tools have their place for single-cloud shops but Terraform wins for anyone crossing clouds."

---
---

# 12. Cloud DNS

## Concept

Cloud DNS is GCP's managed DNS hosting service. Global anycast, high availability, programmatic API.

## AWS Equivalent

**Cloud DNS ≈ Route 53** (DNS hosting only — Route 53 does more)

## Azure Equivalent

**Cloud DNS = Azure DNS**

## Three-Way Comparison

| Feature | Route 53 | Azure DNS | Cloud DNS |
|---|---|---|---|
| DNS hosting | Yes | Yes | Yes |
| Domain registration | Yes | No (App Service Domains) | Yes (Cloud Domains) |
| Health checks + routing policies | Built-in | Traffic Manager (separate) | (via Load Balancer health checks) |
| Private zones | Yes | Yes | Yes |
| Alias records | Yes (Route 53-specific) | Yes (Azure-specific) | (via LB integration) |

## Key Differences from BOTH

**1. Cloud DNS Doesn't Have Native Routing Policies**

AWS Route 53 has weighted, latency-based, geolocation, and failover routing built in. Azure has Traffic Manager as a separate service. Cloud DNS is DNS only — no routing policies.

For routing intelligence on GCP, you use the Global External Application Load Balancer instead (see [[#6-cloud-load-balancing]]). The LB's anycast IP handles latency-based routing at the load balancer layer, not DNS. Different mental model.

**2. Cloud Domains for Registration**

`Cloud Domains` is GCP's domain registrar service — buy a domain and point it at Cloud DNS with one click. Comparable to Route 53 Domains.

**3. Private DNS Zones**

Same as AWS Route 53 Private Hosted Zones and Azure Private DNS Zones. Resolve internal names within a VPC.

## Real Configuration

```bash
# Create a public DNS zone
gcloud dns managed-zones create aakashrao-dev-zone \
  --dns-name=aakashrao.dev. \
  --description="Public zone for aakashrao.dev" \
  --visibility=public

# Get name servers (point your registrar here if not using Cloud Domains)
gcloud dns managed-zones describe aakashrao-dev-zone --format="value(nameServers)"

# Create an A record
gcloud dns record-sets create www.aakashrao.dev. \
  --zone=aakashrao-dev-zone \
  --type=A \
  --ttl=300 \
  --rrdatas=34.120.10.5

# Update via transaction
gcloud dns record-sets transaction start --zone=aakashrao-dev-zone
gcloud dns record-sets transaction add 34.120.10.6 \
  --name=api.aakashrao.dev. --ttl=300 --type=A --zone=aakashrao-dev-zone
gcloud dns record-sets transaction execute --zone=aakashrao-dev-zone

# Create a private DNS zone
gcloud dns managed-zones create internal-zone \
  --dns-name=internal.aakashrao.dev. \
  --visibility=private \
  --networks=prod-vpc \
  --description="Internal service discovery"
```

## Console Path

- Console → **Network Services** → **Cloud DNS**
- Zone detail: **Record sets**, **Registrations** (if using Cloud Domains)

## Memory Hint

- **Cloud DNS** = "Route 53 without the routing policies" (routing lives at the LB layer)
- **Public zone** = internet-resolvable
- **Private zone + VPC binding** = internal service discovery
- **Cloud Domains** = the registrar-side product (like Route 53 Domains)

---
---

# 13. Cloud CDN

## Concept

Cloud CDN caches HTTP(S) content at Google's edge locations globally.

## AWS Equivalent

**Cloud CDN ≈ CloudFront** (but with different attachment model)

## Azure Equivalent

**Cloud CDN ≈ Azure CDN / Front Door CDN** (Front Door is the more comparable modern service)

## Key Differences from BOTH

**1. Cloud CDN Is a Feature of the Load Balancer, Not a Separate Service**

Unlike CloudFront and Front Door which are standalone products, Cloud CDN is enabled with a **checkbox on a Load Balancer backend**. Same LB, same URL map, edge caching added.

Simpler configuration — one resource. Less flexibility — can't have CDN without an LB.

**2. Cache Invalidation**

Same concept as CloudFront invalidations:
```bash
gcloud compute url-maps invalidate-cdn-cache web-url-map \
  --host=www.aakashrao.dev \
  --path="/static/*"
```

## Cloud CDN Configuration

```bash
# Enable Cloud CDN on an existing backend service
gcloud compute backend-services update web-backend \
  --enable-cdn \
  --cache-mode=CACHE_ALL_STATIC \
  --default-ttl=3600 \
  --max-ttl=86400 \
  --global

# Cache modes:
# - CACHE_ALL_STATIC: cache static content, respect Cache-Control on dynamic
# - USE_ORIGIN_HEADERS: fully rely on origin's Cache-Control
# - FORCE_CACHE_ALL: cache everything regardless of headers (dangerous — use carefully)

# Invalidate cache for a path
gcloud compute url-maps invalidate-cdn-cache web-url-map \
  --host=www.aakashrao.dev \
  --path="/static/app.js"
```

## Console Path

- Console → **Network Services** → **Cloud CDN**
- Or directly: Load Balancer → Backend service → **Enable Cloud CDN** toggle

## Memory Hint

- **Cloud CDN** = "CloudFront but bundled into the LB config"
- **One checkbox** on a backend service — that's the entire configuration surface
- **CACHE_ALL_STATIC** default — reasonable starting point

## Interview Q&A — Cloud CDN

**Q: A team wants to add a CDN in front of their existing GCP web application. How does the setup differ from AWS?**

> "Structurally different but conceptually simpler on GCP.
>
> AWS: create a CloudFront distribution, configure origins (usually ALB or S3), set up cache behaviors per path pattern, attach WAF, update DNS to point to CloudFront. Separate resource, separate ID, separate metrics.
>
> GCP: on the existing Global External Application Load Balancer, go to the backend service and toggle 'Enable Cloud CDN.' That's it. Same LB serves as origin, same URL map defines cache behavior per path (via backend service selection), same domain, same certificate, same everything.
>
> Advantages of GCP's approach:
> - Fewer moving parts
> - Cache behavior per path is defined in the same URL map you already use for routing
> - Metrics and logs are unified with the LB
>
> Trade-offs:
> - Less flexibility — CDN and LB are one resource, can't scale them independently
> - CloudFront has more edge features (Lambda@Edge, Origin Shield) that Cloud CDN doesn't match
> - Some enterprise features (custom certificates in specific edge locations, private CDN) are stronger in CloudFront
>
> For most web app CDN needs, Cloud CDN's simplicity wins. For heavy edge computing or specific enterprise CDN requirements, CloudFront remains stronger."

---

# 14. GKE — Google Kubernetes Engine

> **Tags:** #gcp #gke #kubernetes #core
> **Status:** Critical — this is the deepest section for a reason. Kubernetes was invented at Google. GKE is arguably the most mature managed K8s offering, and it's the strongest cross-cloud interview material for someone with a Kubernetes background.
> **Interview Relevance:** ⭐⭐⭐⭐⭐

## Concept

**GKE (Google Kubernetes Engine)** is GCP's managed Kubernetes service. Google open-sourced Kubernetes in 2014 based on their internal Borg system, so GKE has a philosophical claim to being "K8s done right" — features often land here before EKS or AKS.

Two operating modes:
- **Standard mode** — you manage node pools, node sizing, upgrades. Full control. Like EKS or AKS.
- **Autopilot mode** — Google manages nodes entirely. You define pods, Google provisions the exact node capacity needed, per-pod billing. No node concept for you as a user. Closest analog: Fargate for EKS, but more integrated and cheaper for spiky workloads.

## AWS Equivalent

- **EKS (Elastic Kubernetes Service)** — managed control plane, worker nodes you manage (or Fargate for serverless).

## Azure Equivalent

- **AKS (Azure Kubernetes Service)** — managed control plane (free), you manage nodes.

## Key Differences from BOTH

| Aspect | AWS EKS | Azure AKS | GCP GKE |
|---|---|---|---|
| Control plane cost | ~$73/mo per cluster | Free | Standard: ~$73/mo (first cluster free per billing account); Autopilot: no separate charge |
| Node management | You manage OR Fargate | You manage | You manage OR Autopilot |
| Serverless pods | Fargate (limited features) | Virtual Nodes (ACI-backed) | **Autopilot (fully featured, no compromise)** |
| Kubernetes version freshness | Follows upstream, slower | Follows upstream | **Often first to release new versions** |
| Networking | VPC CNI (assigns VPC IPs to pods) | Azure CNI or kubenet | **VPC-native (alias IPs) — pods get real VPC IPs by default** |
| Workload identity | IRSA (IAM Roles for Service Accounts) | Workload Identity (via Entra) | **Workload Identity (SA-to-SA mapping — cleanest of the three)** |
| Auto-upgrade | Manual by default | Manual by default | **Auto-upgrade enabled by default** on release channels |
| DNS | CoreDNS | CoreDNS | CoreDNS + kube-dns (Google's fork, being deprecated in favor of CoreDNS) |
| Ingress default | ALB Controller (add-on) | App Gateway Ingress Controller | **GKE Ingress (built-in, provisions Global External LB)** |

**The big differences to lead with in interviews:**

1. **Autopilot** — no other cloud has an equivalent that's this seamless. Fargate has limitations (no DaemonSets, no privileged pods, no host networking); Autopilot supports almost everything Standard supports. You pay per pod's CPU/memory/ephemeral-storage request, second-billed. For spiky or unpredictable workloads, it's dramatically cheaper than provisioning nodes.
2. **Release channels** — Rapid / Regular / Stable channels for automatic upgrades. Set once, forget. EKS and AKS require manual intervention.
3. **VPC-native networking by default** — pods get IPs from a dedicated subnet range, routable across the VPC, no overlay. Simplifies debugging and hybrid connectivity massively.
4. **Workload Identity is the cleanest** — bind a Kubernetes ServiceAccount to a GCP ServiceAccount via annotation. Pods automatically get GCP creds. No sidecars, no init containers, no token brokers.

## Real Configuration

```bash
# ==========================================
# STANDARD MODE — full control
# ==========================================

# Create a Standard cluster
gcloud container clusters create prod-cluster \
  --region=asia-south1 \
  --release-channel=regular \
  --enable-ip-alias \
  --network=prod-vpc \
  --subnetwork=prod-subnet-asia \
  --cluster-secondary-range-name=pods-range \
  --services-secondary-range-name=services-range \
  --workload-pool=my-project-956179.svc.id.goog \
  --enable-shielded-nodes \
  --shielded-secure-boot \
  --enable-master-authorized-networks \
  --master-authorized-networks=10.0.0.0/24 \
  --enable-private-nodes \
  --master-ipv4-cidr=172.16.0.0/28 \
  --num-nodes=1 \
  --machine-type=e2-medium

# Flags explained:
#   --release-channel=regular     : auto-upgrade on regular channel (stable is more conservative)
#   --enable-ip-alias             : VPC-native (pods get real VPC IPs). ALWAYS set this.
#   --workload-pool=...           : enables Workload Identity Federation
#   --enable-private-nodes        : nodes have no public IPs
#   --master-ipv4-cidr            : control plane's private CIDR (must be /28)
#   --enable-shielded-nodes       : verified boot, integrity monitoring

# Add a second node pool (e.g., high-memory for a specific workload)
gcloud container node-pools create memory-pool \
  --cluster=prod-cluster \
  --region=asia-south1 \
  --machine-type=n2-highmem-4 \
  --num-nodes=1 \
  --enable-autoscaling \
  --min-nodes=1 \
  --max-nodes=5 \
  --node-taints=workload=memory-intensive:NoSchedule \
  --node-labels=pool=memory

# Add a Spot node pool for batch jobs (up to 91% cheaper)
gcloud container node-pools create spot-pool \
  --cluster=prod-cluster \
  --region=asia-south1 \
  --machine-type=e2-standard-4 \
  --spot \
  --enable-autoscaling \
  --min-nodes=0 \
  --max-nodes=10 \
  --node-taints=cloud.google.com/gke-spot=true:NoSchedule

# Get kubeconfig
gcloud container clusters get-credentials prod-cluster --region=asia-south1

# ==========================================
# AUTOPILOT MODE — fully managed pods
# ==========================================

gcloud container clusters create-auto autopilot-cluster \
  --region=asia-south1 \
  --release-channel=regular \
  --network=prod-vpc \
  --subnetwork=prod-subnet-asia

# That's it. No node config. No autoscaler. No node pool decisions.
# Deploy a pod, Google provisions capacity. Pay per pod's resource requests.

# ==========================================
# WORKLOAD IDENTITY — pod-to-GCP-service auth
# ==========================================

# 1. Create GCP service account
gcloud iam service-accounts create app-gsa \
  --display-name="App workload identity"

# 2. Grant it a role (e.g., read from Cloud Storage)
gcloud projects add-iam-policy-binding my-project-956179 \
  --member="serviceAccount:app-gsa@my-project-956179.iam.gserviceaccount.com" \
  --role="roles/storage.objectViewer"

# 3. Create Kubernetes ServiceAccount
kubectl create serviceaccount app-ksa -n production

# 4. Bind them (this is the magic)
gcloud iam service-accounts add-iam-policy-binding \
  app-gsa@my-project-956179.iam.gserviceaccount.com \
  --role=roles/iam.workloadIdentityUser \
  --member="serviceAccount:my-project-956179.svc.id.goog[production/app-ksa]"

# 5. Annotate the K8s SA
kubectl annotate serviceaccount app-ksa -n production \
  iam.gke.io/gcp-service-account=app-gsa@my-project-956179.iam.gserviceaccount.com

# 6. In your deployment spec, use serviceAccountName: app-ksa
# Pods now authenticate to GCP as app-gsa. Zero secrets, zero sidecars.
```

## Production Use Case

**Scenario:** A team is running a Node.js + Postgres app on GKE Standard. Traffic is highly variable — heavy business hours, near-zero at night. They're overpaying because nodes are provisioned for peak load.

**Approach (senior-level answer):**

> "First, break the workload apart by traffic pattern:
> - Steady, predictable pods (auth service, background workers) → main node pool, e2-standard-2 with autoscaling min=2 max=6, 1-year committed use discount on the min. Locks in ~37% off for the baseline you're always running.
> - Bursty pods (main app) → **migrate to Autopilot cluster** or add an Autopilot node pool if we want to stay in Standard. Pay only for what pods actually request during the burst; scale to zero at night.
> - Batch/analytics pods (report generation, data pipelines) → dedicated **Spot node pool** with taints, so only tolerating pods land there. Up to 91% cheaper. Add PodDisruptionBudgets so if Spot nodes get preempted, we don't lose too many replicas at once.
>
> Then Workload Identity for every namespace's pods — no more static secrets in Kubernetes secrets for GCP API access. Cloud SQL Auth Proxy runs as a sidecar or a separate deployment, authenticating via Workload Identity to the Cloud SQL instance — no DB passwords in K8s at all.
>
> For rollouts: enable release channel `regular`, set maintenance windows to align with the app's lowest-traffic hours. GKE handles control plane and node upgrades automatically.
>
> Cost outcome: typically 40-60% reduction vs a fixed-size Standard cluster once you factor in Spot + Autopilot + committed use."

## Console Path

- Console → **Kubernetes Engine** → **Clusters**
- **Workloads** for pod-level view (integrated with cluster)
- **Services & Ingress** for LB integration

## Memory Hint

- **GKE Standard** = "EKS/AKS-style: you manage nodes"
- **GKE Autopilot** = "Fargate but it actually works for real workloads"
- **Workload Identity** = "IRSA but simpler — just annotate the ServiceAccount"
- **Release channels** = "Rapid = bleeding edge, Regular = normal, Stable = conservative"
- **Always `--enable-ip-alias`** — VPC-native is the default modern choice

## Interview Q&A — GKE

**Q: Walk me through how you'd choose between GKE Standard and Autopilot for a new project.**

> "It comes down to three factors: workload predictability, cost model, and control needs.
>
> Autopilot wins when:
> - Traffic is variable or unpredictable — you pay per-pod resource request, per-second billed, so idle capacity costs nothing
> - Team wants to focus on apps, not infrastructure — no node sizing, no autoscaler tuning, no OS patching
> - Workloads are standard — HTTP services, workers, jobs
>
> Standard wins when:
> - You need specific machine types — GPUs, high-memory, ARM (T2A)
> - Workloads require host networking, privileged pods, DaemonSets across all nodes, or custom kernel modules — Autopilot restricts these
> - You have steady baseline load — committed use discounts on Standard nodes beat Autopilot's per-pod pricing
> - You want fine control over networking, kubelet config, or custom node images
>
> My default recommendation for a new project: **start with Autopilot**. If it hits a limitation you actually need, migrate specific workloads to a Standard cluster. Most teams over-optimize for control they never end up using."

**Q: Explain Workload Identity in GKE — how it compares to IRSA in EKS and Azure AD Workload Identity in AKS.**

> "All three solve the same problem: giving Kubernetes pods cloud-native credentials without embedding secrets. But the mechanisms differ meaningfully.
>
> **EKS IRSA** — you associate an IAM Role with a Kubernetes ServiceAccount via an OIDC provider. Pods running under that SA get temporary AWS credentials via the AWS SDK's default credential chain. Requires setting up an OIDC identity provider in IAM, per cluster.
>
> **AKS Workload Identity** — federated credential in Entra maps a Kubernetes SA to an Entra managed identity. Pods get Entra tokens via projected volume. Newer feature — replaced the older 'aad-pod-identity' which had reliability issues.
>
> **GKE Workload Identity** — Kubernetes ServiceAccount is bound to a GCP ServiceAccount via IAM binding, plus an annotation on the KSA. That's it. Google runs an identity proxy on every node that intercepts metadata server requests from pods and returns the mapped GSA's tokens.
>
> Why GKE's is cleanest: no separate OIDC provider setup per cluster (workload-pool is a single project-level setting), no sidecar or projected volume config, and the mapping is bidirectional-explicit — the GSA has to explicitly allow the KSA to impersonate it. Very hard to accidentally over-grant."

**Q: Kubernetes was invented at Google. Does that actually matter for GKE, or is it marketing?**

> "It's partly historical, partly real. The historical part: Borg predates Kubernetes by a decade, and many concepts — pods, controllers, declarative reconciliation — are Borg-derived. GKE inherits operational patterns Google refined internally for years.
>
> The real part: GKE often ships upstream K8s features first. Autopilot didn't exist elsewhere for years. Workload Identity was mature on GKE before AKS shipped its equivalent. GKE's control plane is genuinely more robust — I've seen fewer control plane incidents on GKE than EKS in comparable production usage.
>
> That said, EKS has caught up substantially in the last two years — Karpenter, EKS Auto Mode, Fargate improvements. AKS has closed the gap too. It's not a landslide anymore. But if I were starting a Kubernetes-heavy project on a clean slate, GKE Autopilot on the Regular release channel is still my default recommendation."

---

# 15. VPC Networking

> **Tags:** #gcp #vpc #networking #mental-model-shift
> **Status:** Critical — this is THE section where GCP mental model diverges most from AWS and Azure. Get this wrong in an interview and it signals you haven't really used GCP.
> **Interview Relevance:** ⭐⭐⭐⭐⭐

## Concept

A **GCP VPC (Virtual Private Cloud)** is a **global** software-defined network. Subnets inside a VPC are regional, but the VPC itself spans every GCP region. A single VPC can have subnets in `asia-south1`, `us-central1`, and `europe-west3` simultaneously — all natively connected, no peering, no VPN.

This is the single biggest architectural divergence between GCP and AWS/Azure. Getting it wrong wastes weeks in a migration.

## AWS Equivalent

- **VPC (Virtual Private Cloud)** — regional (bound to a single region), subnets are AZ-scoped.

## Azure Equivalent

- **VNet (Virtual Network)** — regional (bound to a single region), subnets are inside the VNet region.

## Key Differences from BOTH

| Aspect | AWS VPC | Azure VNet | GCP VPC |
|---|---|---|---|
| Scope | Regional | Regional | **Global** |
| Subnet scope | AZ (one AZ per subnet) | Regional (spans zones in region) | Regional (one subnet spans all zones in a region) |
| Cross-region connectivity | VPC Peering or Transit Gateway | VNet Peering (regional or global) | **Native — subnets in different regions in the same VPC talk directly** |
| Firewall rules | Security Groups (stateful, per-ENI) + NACLs (stateless, per-subnet) | NSGs (stateful, per-NIC or per-subnet) | **VPC Firewall Rules (stateful, VPC-wide, filtered by tags/SAs)** |
| Firewall attachment | Per-instance or per-subnet | Per-NIC or per-subnet | **VPC-level, applied by tag/service account/CIDR** |
| Route propagation | Route tables per subnet | Route tables per subnet | Routes are VPC-level, filtered by tags |
| Default deny | SGs default-deny inbound | NSGs default-deny inbound | **Default VPC has default-allow rules; custom VPCs default-deny** |
| Pod IPs (K8s) | VPC CNI assigns VPC IPs | Azure CNI or overlay | **Alias IPs — dedicated pod CIDR block per node** |

**The mental shifts to internalize:**

1. **VPC is global.** You almost never need multiple VPCs for multi-region deployments. In AWS, a global app is 5 VPCs + peering + Transit Gateway. In GCP, it's 1 VPC.
2. **Firewall rules aren't attached to instances or subnets** — they're VPC-level, and they select targets by **network tags** or **service accounts**. Add tag `web-server` to your firewall rule, add tag `web-server` to any instance, done.
3. **No default NACL equivalent.** GCP firewall rules are stateful only. No stateless layer. Simpler, but you lose the belt-and-suspenders defense-in-depth pattern.
4. **Auto mode vs Custom mode VPCs.** Auto mode auto-creates one subnet per region with pre-defined CIDRs. Convenient for demos, banned in production because CIDRs overlap across projects. **Always create Custom mode VPCs in production.**
5. **Shared VPC** — host project owns the VPC, service projects consume subnets. Cross-project networking without peering. Very common in enterprise GCP setups.

## Real Configuration

```bash
# ==========================================
# CUSTOM VPC — production-grade
# ==========================================

# Create the VPC (custom mode — no auto subnets)
gcloud compute networks create prod-vpc \
  --subnet-mode=custom \
  --bgp-routing-mode=regional \
  --mtu=1460

# --bgp-routing-mode=regional  : Cloud Router only advertises routes from its region
#                                (use 'global' if you want cross-region route advertisement)

# Add subnets — one per region
gcloud compute networks subnets create prod-subnet-asia \
  --network=prod-vpc \
  --region=asia-south1 \
  --range=10.10.0.0/20 \
  --secondary-range=pods-range=10.100.0.0/16,services-range=10.200.0.0/20 \
  --enable-private-ip-google-access

gcloud compute networks subnets create prod-subnet-us \
  --network=prod-vpc \
  --region=us-central1 \
  --range=10.20.0.0/20

# --enable-private-ip-google-access : instances without external IPs can reach
#                                     Google APIs (Cloud Storage, BigQuery, etc.)
# --secondary-range                 : for GKE VPC-native pod/service IPs

# ==========================================
# FIREWALL RULES — tag-based
# ==========================================

# Rule 1: Allow SSH from IAP (Google's identity-aware proxy)
gcloud compute firewall-rules create allow-iap-ssh \
  --network=prod-vpc \
  --direction=INGRESS \
  --action=ALLOW \
  --rules=tcp:22 \
  --source-ranges=35.235.240.0/20 \
  --target-tags=ssh-allowed

# 35.235.240.0/20 = IAP's source range. IAP proxies SSH from the browser,
# so opening SSH ONLY to this range means no public SSH exposure.

# Rule 2: Allow HTTP/HTTPS from anywhere to web servers
gcloud compute firewall-rules create allow-web \
  --network=prod-vpc \
  --direction=INGRESS \
  --action=ALLOW \
  --rules=tcp:80,tcp:443 \
  --source-ranges=0.0.0.0/0 \
  --target-tags=web-server

# Rule 3: Allow web servers to reach DB — using service account, not tag
gcloud compute firewall-rules create allow-web-to-db \
  --network=prod-vpc \
  --direction=INGRESS \
  --action=ALLOW \
  --rules=tcp:5432 \
  --source-service-accounts=web-sa@my-project-956179.iam.gserviceaccount.com \
  --target-service-accounts=db-sa@my-project-956179.iam.gserviceaccount.com

# Service-account-based firewalling is the strongest pattern:
# Even if someone spoofs a tag (unlikely but possible via impersonation),
# they can't spoof running as a different service account.

# Rule 4: Explicit deny SSH from internet (defense in depth)
gcloud compute firewall-rules create deny-public-ssh \
  --network=prod-vpc \
  --direction=INGRESS \
  --action=DENY \
  --rules=tcp:22 \
  --source-ranges=0.0.0.0/0 \
  --priority=900

# Lower priority number = higher precedence. Priority 900 beats default rules.

# ==========================================
# CLOUD NAT — outbound internet from private VMs
# ==========================================

gcloud compute routers create prod-router \
  --network=prod-vpc \
  --region=asia-south1

gcloud compute routers nats create prod-nat \
  --router=prod-router \
  --region=asia-south1 \
  --nat-all-subnet-ip-ranges \
  --auto-allocate-nat-external-ips

# ==========================================
# VPC PEERING (for cross-VPC connectivity)
# ==========================================

# Peer prod-vpc with a shared services VPC
gcloud compute networks peerings create prod-to-shared \
  --network=prod-vpc \
  --peer-network=shared-services-vpc \
  --peer-project=shared-services-project

# Peering is non-transitive: A↔B, B↔C does NOT give A↔C.
```

## Production Use Case

**Scenario:** Company runs workloads in `asia-south1` (Mumbai) and `us-central1` (Iowa). They're migrating from AWS where they had two VPCs (one per region) connected via Transit Gateway. Asks: "Do we need two GCP VPCs?"

**Answer:**

> "No — and this is one of the biggest wins moving from AWS to GCP.
>
> Create **one custom-mode VPC** globally. Add two subnets: one in each region with non-overlapping CIDRs. Instances in Mumbai and Iowa can talk to each other over Google's backbone, natively, no peering, no Transit Gateway equivalent, no per-GB inter-region peering cost (though egress between regions still applies).
>
> Firewall rules apply once, VPC-wide, filtered by tag or service account. Instead of duplicating security groups across two VPCs and keeping them in sync, we define `allow-app-to-db` once and it works everywhere.
>
> Where I would use a second VPC: strict tenant isolation (a separate customer's workload), or environments that legally can't be on the same network (compliance boundary). Otherwise, one global VPC with well-designed subnet CIDRs is the pattern.
>
> Migration mapping:
> - AWS VPC (Mumbai) → subnet in `asia-south1`, CIDR 10.10.0.0/20
> - AWS VPC (Ohio) → subnet in `us-central1`, CIDR 10.20.0.0/20
> - AWS Transit Gateway → gone. Native cross-region routing.
> - AWS Security Groups → GCP Firewall Rules with `--target-service-accounts`
> - AWS NACLs → no equivalent. Rely on defense-in-depth through firewall rules and IAM."

## Console Path

- Console → **VPC network** → **VPC networks**
- **VPC network** → **Firewall** for rules
- **VPC network** → **Routes** for custom routes
- **Hybrid Connectivity** → **Cloud Routers / VPN / Interconnect**

## Memory Hint

- **VPC is GLOBAL, subnets are regional** — the opposite of AWS/Azure
- **Firewall rules are VPC-level, targeted by tag or SA** — not attached to instances
- **Always custom-mode VPCs in production** — auto-mode is for demos
- **Service-account-based firewalling** > tag-based > CIDR-based (strongest to weakest)
- **Shared VPC** = one team owns the network, other teams consume subnets

## Interview Q&A — VPC Networking

**Q: A candidate mentioned GCP VPCs are "just like AWS VPCs but with different names." How would you correct that?**

> "That framing misses the most important architectural difference — GCP VPCs are global, AWS VPCs are regional.
>
> In AWS, if you have workloads in Mumbai and Ohio, you have two VPCs and you need Transit Gateway or VPC Peering to connect them. Each VPC has its own route tables, its own NACLs, its own security groups that you have to keep in sync.
>
> In GCP, one VPC spans both regions. Subnets are regional, but the VPC — the security boundary, the firewall rule scope, the routing plane — is global. That single design choice cascades: fewer VPCs to manage, no cross-region peering resources, firewall rules defined once, cheaper networking topology.
>
> The second big difference is how firewall rules attach. AWS security groups attach to ENIs — every instance has its own set. GCP firewall rules exist at the VPC level and select their targets via network tags or service accounts. Instead of maintaining N security groups for N tiers of workload, you maintain N firewall rules that apply based on labels.
>
> So they're not 'the same with different names' — they're fundamentally different network models, and the GCP model is generally simpler at scale."

**Q: When would you deliberately choose to create multiple VPCs in GCP?**

> "Three legitimate reasons:
>
> 1. **Tenant isolation** — hosting workloads for different customers or business units that must not have any network path between them. Even with well-crafted firewall rules, having them in the same VPC means a misconfiguration could bridge them. Separate VPCs make bridging require explicit peering, which is easier to audit.
>
> 2. **Environment boundary (rare)** — some organizations require prod and non-prod to be on entirely different networks. Personally I prefer separate projects with separate VPCs over separate subnets in one VPC for prod/non-prod — cleaner IAM boundaries, easier billing separation.
>
> 3. **Compliance requirement** — PCI-scoped workloads on their own network, HIPAA data on its own network. Auditors like clean boundaries.
>
> For everything else — multi-region app, microservices, dev/stage — I'd stay in one VPC with well-designed subnet CIDRs and firewall rules targeted by service account."

**Q: Explain the practical difference between using network tags vs service accounts as firewall rule targets.**

> "Both work; service accounts are the stronger choice.
>
> Network tags are string labels applied to instances. Anyone with `compute.instances.setTags` permission on an instance can change its tags — and thus change what firewall rules apply to it. If a tag says 'db-client-allowed,' a compromised or misconfigured instance can add that tag to itself and gain database access.
>
> Service accounts are attached to instances at creation and (mostly) not changeable at runtime — changing an instance's service account requires stopping the instance, which shows up in audit logs. Firewall rules targeting a service account can only be matched by instances actually running as that service account.
>
> In production: use service account targeting for anything security-sensitive — database access, secret manager access, cross-tier traffic. Reserve tags for broad, non-sensitive rules like 'allow health checks to any web tier.' If I'm reviewing someone's GCP setup and I see tag-based firewalls guarding databases, I flag it."

---

# 16. BigQuery

> **Tags:** #gcp #bigquery #data-warehouse #flagship
> **Status:** Not core DevOps, but you can't have a serious GCP conversation without it — it shapes many GCP architectures and is the flagship service everyone associates with GCP.
> **Interview Relevance:** ⭐⭐⭐

## Concept

**BigQuery** is GCP's serverless data warehouse — a petabyte-scale columnar analytical database that separates storage and compute. You store data cheaply, run SQL queries that scan terabytes in seconds, pay per bytes-processed or reserved slots.

Key traits: no infrastructure to manage, standard SQL interface, ML models trainable via SQL (BigQuery ML), streaming inserts, integration with every GCP data service.

## AWS Equivalent

- **Redshift** — traditional cluster-based warehouse; **Redshift Serverless** is closer but still less integrated than BigQuery
- **Athena** — serverless SQL over S3, closer conceptually but not a full warehouse

## Azure Equivalent

- **Azure Synapse Analytics** (dedicated SQL pools + serverless SQL pools) — largest analog
- **Microsoft Fabric** — newer unified analytics platform including warehouse

## Key Differences from BOTH

| Aspect | AWS Redshift | Azure Synapse | GCP BigQuery |
|---|---|---|---|
| Provisioning model | Cluster (RA3/DC2) or Serverless | Dedicated SQL pool or Serverless | **Serverless-first (or reserved slots)** |
| Pricing | Node-hours or RPU-hours | DWU-hours or per-TB scanned | **Per-TB scanned or slot-hours** |
| Storage/compute separation | Yes (RA3, Serverless) | Yes | **Yes, and long-term storage auto-tiers cheaper** |
| Streaming ingest | Kinesis → COPY or streaming ingestion | Event Hubs → Synapse | **Native streaming inserts (100k rows/sec per table)** |
| ML | Redshift ML (via SageMaker) | Synapse ML | **BigQuery ML — train models in SQL, no export** |
| Federated queries | Yes (Spectrum over S3) | External tables over ADLS | **Yes, over Cloud Storage, Cloud SQL, Spanner** |

**Why BigQuery shapes GCP architectures:**
- Most analytics workloads default to it — cheap to try (1TB free query quota per month), no cluster to size, no maintenance
- It's often the "sink" for Cloud Storage, Pub/Sub, Firestore exports
- Data engineers on GCP think BigQuery-first; on AWS, it's often S3 + Athena/Redshift; on Azure, Synapse or Fabric

## Real Configuration

```bash
# Create a dataset (like a schema/database)
bq mk --dataset --location=asia-south1 my-project-956179:analytics

# Create a table from a schema file
bq mk --table analytics.events \
  event_id:STRING,user_id:STRING,event_time:TIMESTAMP,properties:JSON

# Load data from Cloud Storage
bq load --source_format=NEWLINE_DELIMITED_JSON \
  analytics.events \
  gs://my-data-bucket/events/*.json

# Run a query (dry-run to see bytes scanned before you're billed)
bq query --dry_run --use_legacy_sql=false \
  'SELECT user_id, COUNT(*) FROM analytics.events WHERE DATE(event_time) = "2026-07-15" GROUP BY user_id'

# Stream inserts (from an app)
# Handled via the BigQuery API, not gcloud — typically from a Cloud Function
# or Cloud Run service processing Pub/Sub messages.

# Set up scheduled query (like a materialized view refresh)
bq query --use_legacy_sql=false --destination_table=analytics.daily_summary \
  --schedule='every 24 hours' \
  --display_name='Daily user summary' \
  'SELECT DATE(event_time) as day, COUNT(DISTINCT user_id) as dau FROM analytics.events GROUP BY day'
```

## Production Use Case

**Scenario:** Application on GCP writes ~10M events/day to Pub/Sub. Need to make these queryable for analytics dashboards and ad-hoc analysis, cheaply.

**Approach:**

> "Pub/Sub → Dataflow (streaming) → BigQuery. Dataflow reads from Pub/Sub, does light transformation (schema validation, timestamp normalization), streams into BigQuery via the Storage Write API.
>
> Table design:
> - Partition on `event_time` (daily partitions). Queries scanning last 7 days only scan those 7 partitions.
> - Cluster on `user_id` and `event_type`. Filter queries prune to relevant clusters.
>
> Cost control:
> - Set a project-level query cost cap (`bq --maximum_bytes_billed` in scheduled queries)
> - Use `--dry_run` in the dashboarding layer before executing expensive queries
> - Partition expiration on the events table (auto-delete data older than 90 days)
>
> For dashboards: Looker Studio (free, connects natively) or Looker (heavier BI tool, also GCP-native).
>
> Cost estimate for 10M events/day: storage ~$20/mo (long-term storage after 90 days is half price), query costs entirely usage-dependent — with partitioning and clustering, most analytics dashboards should scan under 1TB/month, staying in the free tier."

## Console Path

- Console → **BigQuery** → **SQL workspace**

## Memory Hint

- **BigQuery** = "Snowflake, but Google-native and serverless from day one"
- **Pay per bytes scanned** — always dry-run before executing on production data
- **Partition + cluster** — these are your two knobs for query cost control

## Interview Q&A — BigQuery

**Q: Why does BigQuery get mentioned so often in GCP interviews when it's a data service, not core DevOps?**

> "Because GCP is genuinely the strongest cloud for analytics workloads, and BigQuery is why. Many organizations pick GCP specifically for its data stack — BigQuery + Dataflow + Pub/Sub + Vertex AI — and the compute/networking around it becomes secondary.
>
> As a DevOps engineer on GCP, you end up managing infrastructure that feeds BigQuery: Pub/Sub topics, Dataflow pipelines, Cloud Composer (managed Airflow) DAGs, IAM for who can query what, cost governance for who can spend how much on scans. Even if you don't write queries, you're the plumber. Being able to speak intelligently about BigQuery's model — partitioning, clustering, streaming inserts, reserved slots vs on-demand — signals you understand the workloads your infrastructure supports."

**Q: A team is running Redshift on AWS and considering migration to BigQuery. What's the honest trade-off?**

> "BigQuery wins on operations: no cluster to size, no vacuum, no distribution keys, no sort keys. You create a dataset, load data, query. Redshift RA3 and Serverless have closed the gap but haven't matched the zero-touch operations of BigQuery.
>
> BigQuery wins on scalability for spiky workloads: on-demand pricing means idle time costs nothing beyond storage. Redshift Serverless charges RPU-hours even when idle if configured with a base capacity.
>
> Redshift wins on predictable, high-QPS workloads: reserved-node Redshift can be cheaper than BigQuery slots for constant-load OLAP.
>
> Redshift wins on tight AWS integration: if the rest of the stack is on AWS, moving analytics off-cloud creates egress costs and cross-cloud latency.
>
> Honest recommendation: if the team is committed to a multi-cloud future or GCP-heavy, migrate. If they're deeply AWS-integrated with steady predictable analytics load, Redshift RA3 with reserved instances is probably fine and cheaper than switching. Migration is not free — SQL dialect differences, security model changes, retraining data engineers."

---

# 17. Three-Way Cross-Cloud Interview Master Q&A

> **Tags:** #gcp #cross-cloud #interview #trilateral #master-qa
> **Status:** The payoff section. Use this to demonstrate trilateral fluency in interviews.
> **Interview Relevance:** ⭐⭐⭐⭐⭐

## The Terminology Cheat Sheet (Three-Way)

| Category | AWS | Azure | GCP |
|---|---|---|---|
| **Account/Project boundary** | Account | Subscription | Project |
| **Org hierarchy** | Organization → OU → Account | Tenant → Management Group → Subscription → RG | Organization → Folder → Project |
| **Identity provider** | IAM + IAM Identity Center | Entra ID | Cloud Identity + IAM |
| **RBAC unit** | IAM Role / Policy | Azure RBAC Role Assignment | IAM Role Binding |
| **Machine identity** | IAM Role (assumed) | Managed Identity | Service Account |
| **VM** | EC2 | Virtual Machine | Compute Engine |
| **VM autoscaling group** | Auto Scaling Group | Virtual Machine Scale Set | Managed Instance Group |
| **Block storage** | EBS | Managed Disk | Persistent Disk |
| **Object storage** | S3 | Blob Storage | Cloud Storage |
| **File storage (NFS)** | EFS | Azure Files | Filestore |
| **Relational DB (managed)** | RDS / Aurora | Azure SQL / Azure DB for Postgres | Cloud SQL / AlloyDB |
| **NoSQL (key-value)** | DynamoDB | Cosmos DB (Table API) | Bigtable / Firestore (Datastore mode) |
| **NoSQL (document)** | DynamoDB (single-table) | Cosmos DB (Core / SQL API) | Firestore (Native mode) |
| **Globally consistent SQL** | Aurora Global (not truly consistent) | Cosmos DB (with Strong consistency, regional) | **Spanner (truly globally consistent)** |
| **Data warehouse** | Redshift | Synapse / Fabric | BigQuery |
| **Serverless functions** | Lambda | Azure Functions | Cloud Functions |
| **Serverless containers** | Fargate / App Runner | Container Apps | **Cloud Run** |
| **Managed Kubernetes** | EKS | AKS | GKE |
| **Container registry** | ECR | ACR | Artifact Registry |
| **Load balancer (L7)** | ALB | Application Gateway | Application Load Balancer (Global External) |
| **Load balancer (L4)** | NLB | Load Balancer (Standard) | Network Load Balancer (Passthrough) |
| **Global LB / CDN** | CloudFront + Global Accelerator | Front Door | **Global External LB + Cloud CDN (bundled)** |
| **CDN** | CloudFront | Azure CDN / Front Door | Cloud CDN |
| **DNS** | Route 53 | Azure DNS | Cloud DNS |
| **Network (regional/global)** | VPC (regional) | VNet (regional) | **VPC (global)** |
| **Firewall (network)** | Security Group + NACL | NSG | VPC Firewall Rules |
| **API gateway** | API Gateway | API Management | API Gateway / Apigee |
| **Message queue** | SQS | Service Bus / Storage Queue | Pub/Sub (also does pub-sub) |
| **Pub-sub messaging** | SNS | Event Grid | Pub/Sub |
| **Streaming (Kafka-like)** | Kinesis / MSK | Event Hubs | Pub/Sub Lite |
| **Data pipeline (ETL)** | Glue / Step Functions | Data Factory / Synapse Pipelines | Dataflow / Cloud Composer |
| **IaC (native)** | CloudFormation / CDK | ARM / Bicep | Deployment Manager (deprecated) |
| **IaC (cross-cloud)** | Terraform | Terraform | Terraform |
| **CI/CD (native)** | CodePipeline + CodeBuild + CodeDeploy | Azure DevOps Pipelines | Cloud Build + Cloud Deploy |
| **Secrets manager** | Secrets Manager / Parameter Store | Key Vault | Secret Manager |
| **Monitoring (metrics/logs)** | CloudWatch | Azure Monitor / Log Analytics | Cloud Monitoring / Cloud Logging |
| **APM / tracing** | X-Ray | Application Insights | Cloud Trace |

## Cost Model Differences (One-Liner Each)

| Cloud | Discount Model | Compute Billing | Key Cost Wins |
|---|---|---|---|
| **AWS** | Reserved Instances / Savings Plans (1yr/3yr commit) | Per-second (60s min) | Spot Instances (up to 90% off), Savings Plans flexible across services |
| **Azure** | Reserved Instances (1yr/3yr) + Hybrid Benefit for Windows/SQL | Per-second | Hybrid Benefit slashes Windows/SQL licensing costs; Spot VMs |
| **GCP** | **Automatic Sustained Use Discounts** (up to 30% off, no commitment) + Committed Use Discounts (1yr/3yr) | Per-second (60s min) | **SUDs apply without you doing anything**; Custom Machine Types (pay for exact CPU/RAM you need); Spot VMs (up to 91% off) |

**Interview soundbite:** "GCP's cost model is the most forgiving — you get up to 30% off automatically just by running instances continuously, no upfront planning. AWS and Azure require you to actively commit to Reserved Instances or Savings Plans to get comparable discounts. Add Custom Machine Types on top and you're paying for exactly what you use."

---

## Master Q&A — Multi-Cloud Senior Interview Questions

### Q1: "You describe yourself as multi-cloud (AWS + Azure + GCP). Realistically, how do you decide which cloud for a workload?"

> "Honestly, the choice is usually made before I get involved — it's driven by enterprise agreements, existing ecosystem, or team skills. But when I actually have influence:
>
> **AWS by default** — broadest service catalog, most mature ecosystem, largest hiring pool. If nothing else drives the decision, AWS is the safe pick.
>
> **Azure when** — the org runs heavy Microsoft stack (M365, Windows Server, SQL Server, .NET, AD). Hybrid Benefit alone can save 40% on Windows/SQL licensing. Entra ID integration with corporate identity is unmatched.
>
> **GCP when** — data-heavy workloads (BigQuery is genuinely best-in-class), Kubernetes-first architecture (GKE Autopilot has no real equivalent), or teams that value operational simplicity (Cloud Run, sustained-use discounts, no cluster/DWU sizing exercises).
>
> For most orgs, the real answer is 'the cloud your team knows.' Cross-cloud migrations are expensive and rarely produce the savings people expect. My job is usually to make the chosen cloud work well, not to advocate for switching."

### Q2: "Design a 3-tier app on GCP for a moderate-scale SaaS workload."

> "Three tiers: web/API, application, data. Multi-region-ready but starting in `asia-south1`.
>
> **Networking:**
> - One custom-mode VPC (global, so it'll extend to additional regions later without redesign)
> - Regional subnet in asia-south1 with secondary ranges for GKE pods and services
> - Cloud NAT for outbound-only egress from private nodes
> - Firewall rules using service-account targeting (not tags) for anything security-relevant
>
> **Load balancing / edge:**
> - Global External Application Load Balancer with anycast IP
> - Managed SSL cert (Google auto-renews)
> - Cloud CDN toggled on for static content
> - Cloud Armor policies for WAF + DDoS
>
> **Compute (app tier):**
> - GKE cluster on Autopilot, Regular release channel
> - Main app deployed as Deployment with HPA on CPU + custom metric (request latency)
> - Workload Identity for pod-to-GCP authentication — no static secrets in K8s
> - Cloud SQL Auth Proxy sidecar for DB access
>
> **Data tier:**
> - Cloud SQL Postgres, Regional HA, private IP only (via Private Service Connect)
> - Read replicas for reporting queries
> - Daily automated backups + PITR
> - Firestore for user session data / real-time features (mobile clients get real-time listeners for free)
>
> **Async / eventing:**
> - Pub/Sub for inter-service events
> - Cloud Run services for lightweight event consumers (scale to zero, request-based billing)
>
> **Observability:**
> - Cloud Logging (aggregates GKE + Cloud Run + Cloud SQL logs automatically)
> - Cloud Monitoring dashboards + alerting
> - Cloud Trace for distributed tracing (auto-instrumented if using OpenTelemetry)
>
> **CI/CD:**
> - Cloud Build for CI (or GitHub Actions with Workload Identity Federation — no long-lived keys)
> - Artifact Registry for container images with vulnerability scanning
> - Cloud Deploy for progressive rollouts to GKE
>
> **Cost optimization from day one:**
> - Autopilot for main workloads — pay per pod, no idle capacity
> - Committed use discounts on the steady baseline of Cloud SQL and any Standard-mode node pools (if added later)
> - Spot node pool for batch/analytics workloads
> - Cloud CDN reduces egress costs meaningfully at scale
>
> **Compare this to my Azure 3-tier answer** (in [[Azure-Mastery-Notes]]): fewer networking resources (no regional VNet duplication for multi-region readiness), simpler LB (one product, not App Gateway + Front Door), Autopilot has no direct Azure equivalent. Compare to AWS: no ALB Controller add-on needed for GKE Ingress, no separate NACL layer, one VPC instead of one per region."

### Q3: "When is GCP genuinely the right choice over AWS or Azure? Not marketing — real reasons."

> "Four scenarios where I'd actively recommend GCP:
>
> 1. **Data / analytics is the core workload.** BigQuery + Dataflow + Pub/Sub + Vertex AI is a coherent stack that competitors haven't matched. Redshift and Synapse work, but they require more operational effort and integrate less cleanly. If your product IS the data, GCP wins.
>
> 2. **Kubernetes-first architecture.** GKE — especially Autopilot — is measurably ahead of EKS and AKS in operational maturity, feature velocity, and cost efficiency for variable workloads. If 80%+ of your compute will be K8s, GCP is the strongest platform.
>
> 3. **Containers-and-scale-to-zero microservices.** Cloud Run has no true equivalent. App Runner and Container Apps come close but limit concurrency-per-instance, don't support arbitrary containers as flexibly, and have less mature routing/traffic-splitting. If your architecture is 'a mesh of stateless containerized services,' Cloud Run is a differentiator worth the platform choice.
>
> 4. **Globally-consistent transactional data.** Spanner is the only true answer to 'I need SQL with global strong consistency and horizontal scalability.' Aurora Global replicates but reads can be stale. Cosmos DB has strong consistency but only within a region. If you genuinely need this — global inventory, financial ledgers, gaming leaderboards — Spanner is the reason.
>
> **When NOT to pick GCP** (honest):
> - Windows/SQL Server-heavy workloads (Azure wins on licensing)
> - Deep AWS-service integration already (migrating won't pay off)
> - Team has no GCP skills and no time to train (rare skill in the Indian market — AWS is dominant)
> - Compliance requirements where AWS/Azure have more certifications in your specific region"

### Q4: "What's the biggest architectural mistake you see engineers make when moving from AWS or Azure to GCP?"

> "Not internalizing that VPCs are global. I've seen migrations where teams create one GCP project per AWS region — because that's what they'd do in AWS — and end up with N VPCs, N sets of firewall rules to keep in sync, VPC peering everywhere. It works, but it fights the platform.
>
> The GCP-native pattern: one project (or a small number, split by environment or business unit), one global VPC per project, subnets per region. Firewall rules defined once, targeted by service account. When you add a new region, you add a subnet — not a whole VPC.
>
> Second most common mistake: not using Workload Identity for pod-to-GCP auth in GKE. Teams port over patterns from EKS where they mounted IAM credentials into pods, or from AKS where they used aad-pod-identity. Workload Identity is simpler and more secure — just annotate the ServiceAccount — but requires understanding the SA-to-SA binding model.
>
> Third: over-using basic IAM roles (owner, editor, viewer). Teams from AWS are used to fine-grained IAM policies; on GCP they see 'Editor' as a convenient default and grant it broadly. Editor is nearly Owner — grants nearly all write access across nearly all services. Predefined roles or custom roles are the correct answer in production."

### Q5: "Compare cost optimization strategies across all three clouds."

> "AWS: commitment-based. Savings Plans and Reserved Instances are the primary discount lever — 40-72% off in exchange for 1-3 year spend commitments. Spot Instances for interruptible workloads (up to 90% off). Requires active planning and regular re-evaluation.
>
> Azure: commitment + licensing. Reserved Instances similar to AWS. Hybrid Benefit is the unique lever — if you have on-prem Windows Server or SQL Server licenses under Software Assurance, you can 'bring' them to Azure and eliminate the licensing portion of VM cost. For Windows-heavy shops, this alone can justify Azure over AWS. Spot VMs for interruptibles.
>
> GCP: passive + commitment. Sustained Use Discounts apply automatically — run a VM for a full month and you get up to 30% off, no commitment, no upfront work. Committed Use Discounts on top for another 20-30% if you can commit to CPU/RAM totals for 1-3 years (note: CUDs are commit-to-resource-totals, not commit-to-specific-machines like AWS RIs, which is more flexible). Custom Machine Types let you pay for exactly the CPU/RAM you need — no over-provisioning to the next tier. Spot VMs up to 91% off.
>
> Real-world rule of thumb: for equivalent workloads without any optimization, GCP tends to be 10-20% cheaper because of automatic SUDs. With aggressive optimization on all three, AWS can catch up via Savings Plans and Azure can beat both if you have Hybrid Benefit-eligible workloads. The 'cheapest cloud' answer depends entirely on your workload profile and licensing situation."

---

# 18. Wrap-Up — What's Next

> **Tags:** #gcp #next-steps #action-plan

## Practical Next Steps (Not Just More Notes)

1. **Sign up for GCP Free Tier + $300 credit.** Real hands-on time on GCP is worth more than another 100 pages of notes. Free tier includes an e2-micro VM in us-west/central/east regions running continuously — enough for a persistent lab.
2. **Recreate your `flask-mongo-app` on Cloud Run + Firestore.** Same app, different platform. This is the fastest way to internalize how GCP differs from AWS in practice. Two Cloud Run services (Flask + a worker), Firestore for storage, Artifact Registry for images, Cloud Build for CI. Should take a weekend.
3. **IaC exercise: your Terraform ECS+ALB project → equivalent GCP stack.** Rewrite in Terraform for GCP: Cloud Run + Global External LB + Cloud SQL. Same app, three clouds' worth of Terraform in the same repo — this becomes portfolio gold for MNC interviews.
4. **Deploy the same 3-tier app across AWS, Azure, and GCP.** Even a trivial app. The exercise of getting a real workload running on all three cements the terminology and forces you to hit the actual differences. Put it on GitHub. Interviewers love this.
5. **Complete the GCP Associate Cloud Engineer path on Google Cloud Skills Boost** (formerly Qwiklabs). Free with GCP account. Hands-on labs > video lectures.

## Priority Order

Given your priorities (Cloud/DevOps role transition, hands-on > theory):

1. **Now (this week):** Sign up GCP free tier, spin up a VM, deploy a container to Cloud Run
2. **This month:** Recreate `flask-mongo-app` on GCP, add it to portfolio
3. **Next month:** Triple-cloud IaC repo demonstrating the same 3-tier app across all three
4. **Ongoing:** Interview cycles — the real learning loop. Reference these notes when specific gaps show up in interview feedback.

## What NOT to Do

- **Don't** try to get a GCP certification before you have hands-on time. Cert-first is theory-first, and it won't land you the role.
- **Don't** keep re-reading these notes without practice. Cloud knowledge decays fast without use.
- **Don't** try to be equally deep on all three clouds. AWS is your strongest, keep it that way. Azure and GCP fluency at the level of "I can compare architectures and speak intelligently" is enough for 95% of MNC senior DevOps roles.

## Security Reminder

You've stored credentials (AWS keys, GitHub PAT, MongoDB passwords, Docker Hub tokens) in plain-text Obsidian notes before. Do NOT do this with GCP service account keys — they're just as dangerous. Use:

- **Workload Identity Federation** for CI/CD (GitHub Actions can auth to GCP with zero long-lived keys via OIDC)
- **Application Default Credentials** for local dev (`gcloud auth application-default login`)
- **Secret Manager** for anything the app needs at runtime — never in code, never in Obsidian
- If you ever paste a service account key into a note by accident: **rotate immediately** (`gcloud iam service-accounts keys delete`) and audit the key's recent usage in Cloud Audit Logs

---

## Cross-References

- [[AWS-Mastery-Notes]] — foundational cloud knowledge, IAM depth, S3/EC2/ECS/EKS
- [[Azure-Mastery-Notes]] — the second leg, Entra ID, AKS, VNet, Bicep
- [[00-Fundamentals]] — DevOps foundations
- [[Kubernetes-Deep-Dive]] — K8s concepts referenced heavily in GKE section
- [[Terraform-Notes]] — the practical IaC choice for cross-cloud work

---

**End of GCP Mastery Notes.**

You now have a trilateral cloud reference. When an interviewer asks "have you worked across clouds?" — you can answer with concrete architectural comparisons, not vague familiarity. That's the difference between a mid-level and senior DevOps engineer.

Next real conversation to have: which of these three clouds does your target company actually run on? Optimize interview prep depth toward that. But keep the other two at "can compare and contrast intelligently" — that's the trilateral advantage.