# Azure Mastery — Complete DevOps Notes

**Tags:** #azure #devops #cloud #multicloud #interview #masterreview
**Status:** ✅ Active Reference
**Interview Relevance:** 🔴 Maximum
**Level:** DevOps Engineer / Cloud Engineer (2-3 YOE, cross-cloud)
**Prerequisites:** [[AWS-Mastery-Notes]] — this doc assumes AWS fluency and maps forward

> **How to use this note:**
> Every service follows: Concept → AWS Equivalent → Key Differences → Real Config → Production Use Case → Console Path → Memory Hint → Interview Q&A.
> The AWS mapping is deliberate — you're not learning Azure from scratch, you're translating what you already know. When the mental model is different (RBAC vs IAM policies, resource groups vs tags), the "Key Differences" section calls it out explicitly.

---

# TABLE OF CONTENTS

1. [Azure Fundamentals — Subscriptions, Resource Groups, Tenants](#1-azure-fundamentals)
2. [Entra ID & RBAC — Identity and Access](#2-entra-id--rbac)
3. [Virtual Machines — Compute](#3-virtual-machines)
4. [Managed Disks — Block Storage](#4-managed-disks)
5. [Blob Storage — Object Storage](#5-blob-storage)
6. [Load Balancer, Application Gateway, Front Door](#6-load-balancer-app-gateway-front-door)
7. [Virtual Machine Scale Sets (VMSS)](#7-vmss--auto-scaling)
8. [Azure SQL Database — Managed Relational](#8-azure-sql-database)
9. [Cosmos DB — Multi-Model NoSQL](#9-cosmos-db)
10. [Azure Functions — Serverless](#10-azure-functions)
11. [ARM Templates & Bicep — IaC](#11-arm--bicep)
12. [Azure DNS](#12-azure-dns)
13. [Azure CDN & Front Door — Content Delivery](#13-azure-cdn)
14. [AKS — Azure Kubernetes Service](#14-aks)
15. [Networking — VNets, Subnets, NSGs](#15-networking)
16. [Cross-Cloud Interview Master Q&A](#16-cross-cloud-interview-master-qa)

---
---

# 1. Azure Fundamentals

Before any service, you need to understand Azure's resource hierarchy. This is **the single biggest mental-model shift from AWS** and gets asked in interviews constantly.

## The Hierarchy

```
Microsoft Entra ID Tenant  (identity boundary — like an AWS Organization)
    ↓
Management Group           (group of subscriptions with shared policy)
    ↓
Subscription               (billing + quota boundary — like an AWS Account)
    ↓
Resource Group             (logical container — NO AWS equivalent)
    ↓
Resource                   (VM, storage account, DB, etc.)
```

## Key Concepts

**Tenant (Entra ID)**
- The identity root — one company = one tenant, usually
- Contains users, groups, applications
- Not billing-related; purely identity

**Subscription**
- The **billing container** — every resource belongs to a subscription
- Also a quota boundary (per-subscription limits on VMs, IPs, etc.)
- Closest analogue: an AWS Account
- Enterprises typically use multiple: `prod-sub`, `dev-sub`, `sandbox-sub`

**Resource Group (RG)**
- **This concept doesn't exist in AWS.** Critical to understand.
- A logical folder holding related resources (VM + its disk + NIC + public IP all live together)
- Resources CAN'T span RGs, but CAN span regions within one RG
- Deleting the RG deletes all resources in it — **incredibly useful for cleanup**
- Every resource must belong to exactly one RG

**Resource**
- The actual thing: VM, storage account, database, VNet, etc.
- Every resource has a globally unique Resource ID: `/subscriptions/{sub-id}/resourceGroups/{rg}/providers/Microsoft.Compute/virtualMachines/{vm-name}`

## AWS ↔ Azure Hierarchy Mapping

| AWS | Azure |
|---|---|
| Organization | Entra ID Tenant + Management Groups |
| Account | Subscription |
| (implicit — you use tags) | **Resource Group** |
| Resource (EC2, S3 bucket) | Resource (VM, Storage Account) |
| Region | Region (mostly same concept) |
| Availability Zone | Availability Zone (same concept, but not all regions support them) |

## Why Resource Groups Matter (a lot)

In AWS, you use tags + IAM to organize resources logically. In Azure, the Resource Group **is** the organizational primitive.

Real production patterns:
```
rg-prod-web-eastus         → all production web tier resources in East US
rg-prod-db-eastus          → all production database resources in East US
rg-dev-sandbox-aakash      → developer sandbox, deletable anytime
```

**Killer feature:** `az group delete --name rg-dev-sandbox-aakash` nukes everything inside — 47 resources gone in one command. In AWS you'd script this or hit "delete everything" one resource type at a time.

## Regions and Availability Zones

- Regions: `eastus`, `westeurope`, `centralindia`, `southeastasia`, etc.
- AZs: numbered 1, 2, 3 within a region (like AWS)
- **Not all Azure regions have AZs** — check before designing for HA
- Some regions have paired regions built in for DR (e.g., East US ↔ West US)

## Real Configuration

```bash
# Login (interactive)
az login

# List subscriptions
az account list --output table

# Set active subscription
az account set --subscription "MSDN-Subscription-Name"

# Create a resource group
az group create --name rg-devops-learning --location eastus

# List resource groups
az group list --output table

# Delete a resource group (and everything in it — dangerous!)
az group delete --name rg-devops-learning --yes --no-wait

# List all resources in an RG
az resource list --resource-group rg-devops-learning --output table
```

## Console Path

**Azure Portal home** — `portal.azure.com`
Everything anchors from here. Key spots to know:
- **All resources** (left sidebar) — flat list of everything
- **Resource groups** — grouped view, matches how you actually think
- **Subscriptions** (search bar) — billing and quota view
- **Microsoft Entra ID** (search bar) — identity

## Memory Hint

- **Tenant** = the whole company (identity)
- **Subscription** = the credit card (billing + quota)
- **Resource Group** = the project folder (organization)
- **Resource** = the actual thing

"AWS has accounts + tags. Azure has subscriptions + resource groups + tags."
The RG replaces half of what tags do in AWS.

## Interview Q&A — Fundamentals

**Q: You've worked in AWS. Walk me through the equivalent hierarchy in Azure and where they differ.**

> "AWS is Organization → Account → Resources, with tags providing logical grouping. Azure inserts an extra layer: Tenant → Subscription → Resource Group → Resource.
>
> The Tenant is Entra ID — pure identity boundary, one per company usually. Subscription is closest to an AWS Account — it's the billing container and quota boundary. Enterprises use multiple subscriptions to separate prod, dev, sandbox — same pattern as AWS multi-account strategies.
>
> The genuinely new concept is Resource Group. It's a mandatory logical container — every resource must live in exactly one RG. In AWS you fake this with tags plus IAM boundaries. In Azure it's structural.
>
> Practical implications: RGs enable atomic cleanup — `az group delete` removes 50 resources in one command. They also naturally scope Role-Based Access Control — grant Contributor role at RG level and it inherits down. In AWS you'd write more granular IAM policies to achieve the same.
>
> The trap for AWS engineers moving to Azure: over-nesting resources into too few RGs. Best practice: one RG per environment per tier — `rg-prod-web-eastus`, `rg-prod-db-eastus`. Not one giant `rg-production` with everything mixed."

**Q: Can a resource span multiple resource groups or regions?**

> "A resource lives in exactly one RG and one region. Non-negotiable at creation time. You can move resources between RGs later — Azure supports it for most resource types with `az resource move` — but at any moment a resource has one parent RG.
>
> Regions are more flexible. Different resources in one RG can be in different regions. So `rg-prod-webapp` might have VMs in East US and a Storage Account in Central India. This differs from AWS where region choice is per-resource and you don't have the RG concept forcing you to think about it.
>
> Practical note: keeping related resources in the same region matters more than keeping them in the same RG. Cross-region latency and egress costs will kill you fast if you mix regions carelessly."

---
---

# 2. Entra ID & RBAC

## Concept

Azure splits identity and authorization into two systems:
- **Microsoft Entra ID** (formerly Azure Active Directory / Azure AD) — identity provider
- **Azure RBAC** — authorization on Azure resources

Entra ID handles WHO. RBAC handles WHAT they can do.

## AWS Equivalent

| AWS | Azure |
|---|---|
| IAM Users | Entra ID Users |
| IAM Groups | Entra ID Groups |
| IAM Roles (assumable) | Entra ID Service Principals + Managed Identities |
| IAM Policies (JSON) | RBAC Role Assignments |
| AWS SSO | Entra ID (natively an IdP) |

**The big shift:** AWS IAM is one system doing both identity and authorization. Azure splits it. This matters because Entra ID also handles Microsoft 365, third-party SaaS SSO, and enterprise-wide identity — it's much bigger than just cloud resource access.

## Key Differences from AWS

**1. Role-Based, not Policy-Based**
AWS IAM: you write JSON policies with `Action` and `Resource`.
Azure: you assign **pre-defined roles** (Contributor, Reader, Owner) to a **scope** (subscription / RG / resource).

You *can* write custom roles in JSON, but 90% of production uses built-in roles. This is simpler than AWS IAM once you learn the vocabulary.

**2. Scope is Hierarchical and Inherited**
```
Assignment at Subscription scope  → applies to all RGs and resources
Assignment at RG scope            → applies to all resources in that RG
Assignment at Resource scope      → applies to just that one resource
```
In AWS you'd write a policy with a `Resource` ARN pattern. In Azure you pick where to attach the role assignment.

**3. Managed Identities Beat IAM Roles**
Azure's answer to "IAM role attached to EC2" is **Managed Identity**. Two flavors:
- **System-assigned**: tied to one resource's lifetime — VM deleted, identity gone
- **User-assigned**: standalone identity, attach to multiple resources

Same purpose as AWS Instance Profiles but cleaner API — no STS token juggling in your app code, the SDK just works.

**4. Conditional Access = MFA on Steroids**
Entra ID has policies like "require MFA when accessing from outside corporate network" or "block sign-in from unmanaged devices." Much richer than AWS IAM's MFA setting.

## Built-in RBAC Roles You Must Know

| Role | Scope of Power | AWS Analogue |
|---|---|---|
| **Owner** | Full access + can grant access to others | Admin |
| **Contributor** | Full access, CANNOT grant access | PowerUser |
| **Reader** | View everything, change nothing | ReadOnlyAccess |
| **User Access Administrator** | Only manages access, nothing else | Split of IAM admin |

Then hundreds of service-specific roles: `Virtual Machine Contributor`, `Storage Blob Data Reader`, `AKS Cluster Admin Role`, etc.

## Real Configuration

```bash
# List all users
az ad user list --output table

# Create a user
az ad user create \
  --display-name "Aakash Rao" \
  --user-principal-name aakash@yourdomain.onmicrosoft.com \
  --password 'TempPass123!' \
  --force-change-password-next-sign-in true

# Assign Contributor role at RG scope
az role assignment create \
  --assignee aakash@yourdomain.onmicrosoft.com \
  --role "Contributor" \
  --scope /subscriptions/{sub-id}/resourceGroups/rg-dev-sandbox

# Assign at subscription scope (broader)
az role assignment create \
  --assignee aakash@yourdomain.onmicrosoft.com \
  --role "Reader" \
  --scope /subscriptions/{sub-id}

# List role assignments for a user
az role assignment list --assignee aakash@yourdomain.onmicrosoft.com --output table

# Create a Service Principal (like AWS IAM Role for CI/CD)
az ad sp create-for-rbac \
  --name "github-actions-deployer" \
  --role Contributor \
  --scopes /subscriptions/{sub-id}/resourceGroups/rg-prod-web

# This returns appId, password, tenant — save these; they're your credentials
```

## Production Use Case

**Scenario:** Your team of 8 needs Azure access. Two developers need to deploy to dev, two SREs need prod access, one manager needs read-only across everything.

**Wrong way (AWS habit):** create 8 individual role assignments per RG per person = 40+ assignments.

**Right way:**
1. Create Entra ID **groups**: `grp-developers`, `grp-sres`, `grp-managers`
2. Add users to groups
3. Assign roles to groups at appropriate scope:
   - `grp-developers` → Contributor on `rg-dev-*` RGs
   - `grp-sres` → Contributor on `rg-prod-*` RGs
   - `grp-managers` → Reader at subscription scope
4. Onboarding a new engineer = add them to a group. Done.

**Bonus production pattern:** Use **Privileged Identity Management (PIM)** — engineers get elevated permissions only when needed, with approval workflow, audit trail, and auto-expiry. No standing prod admin access. Not free (requires Entra ID P2 license) but critical for compliance environments.

## Console Path

- Search bar → "Microsoft Entra ID" → **Users** (create/manage identities)
- Any resource / RG / subscription → **Access control (IAM)** blade → **Role assignments** tab
- The "IAM" blade name is confusing — Microsoft uses "IAM" for Azure RBAC UI even though the identity itself lives in Entra ID

## Memory Hint

- **Entra ID** = who you are (identity)
- **RBAC** = what you can do (authorization)
- **Role** = a bundle of permissions (pre-defined, don't write JSON unless you must)
- **Scope** = where the role applies (sub / RG / resource)
- **Managed Identity** = "instance profile for Azure" — no credentials to manage
- "AWS says: write a policy. Azure says: pick a role and stick it somewhere."

## Interview Q&A — Entra ID & RBAC

**Q: An AWS engineer joins your Azure-heavy team. Walk them through the mental shift for identity and access.**

> "Three shifts to internalize.
>
> First, identity and authorization are separate systems in Azure. Entra ID owns identity — users, groups, service principals — and it's a full enterprise identity provider used well beyond cloud resource access. RBAC handles authorization on Azure resources. In AWS both are IAM.
>
> Second, Azure is role-first, not policy-first. You rarely write JSON. You assign pre-built roles like Contributor, Reader, Storage Blob Data Contributor to a scope. The scope is hierarchical: subscription > resource group > resource, and assignments cascade down. This is simpler than AWS IAM in practice — less permission-crafting, more picking from a catalog.
>
> Third, Managed Identities replace IAM roles for compute. When you turn on a system-assigned managed identity on a VM, Azure creates a service principal tied to that VM's lifetime. Your app calls Azure SDK, the SDK grabs a token from the instance metadata endpoint automatically. No secrets in code, no STS assume-role dance. Cleaner than AWS instance profiles in my opinion.
>
> One gotcha for AWS folks: Azure has TWO 'admin' concepts — Owner and Contributor. Contributor has full resource access but CAN'T grant access to others. Owner can. This split doesn't exist in AWS's AdministratorAccess. Use Contributor by default; reserve Owner for people who actually need to manage permissions."

**Q: Design a least-privilege setup for a GitHub Actions pipeline deploying to Azure.**

> "Create a Service Principal specifically for the pipeline — not a human user. That gives you a clean audit trail and no shared credentials.
>
> `az ad sp create-for-rbac` scoped to just the resource groups the pipeline touches. Not subscription-wide. If it deploys to `rg-prod-web-eastus`, that's the only scope it gets.
>
> Role choice depends on what the pipeline does. If it's Terraform managing existing resources, Contributor on those RGs. If it also creates role assignments or Key Vault access policies, upgrade to User Access Administrator on top — or better, split into a two-stage pipeline where the identity-changing stage runs with a separately-scoped SP.
>
> Store the SP credentials in GitHub Actions secrets. Modern practice: use OpenID Connect federation instead of long-lived secrets — GitHub Actions gets a short-lived token from Entra ID per run. Same pattern as AWS OIDC federation to IAM roles. No password in secrets, less to rotate.
>
> For extra safety: enable Entra ID sign-in logs and set up alerts on unusual SP activity — logins from unexpected IPs, permission escalation attempts. Same defense-in-depth mindset as AWS CloudTrail."

**Q: When would you write a custom RBAC role instead of using a built-in one?**

> "Almost never — but here's when.
>
> Built-in roles cover 95% of cases. The catalog has hundreds — 'Storage Blob Data Reader,' 'AKS Cluster User Role,' 'DNS Zone Contributor.' Search before you write custom.
>
> Custom roles make sense when the built-in role is too broad but the next-narrower one is too narrow. Real example: a backup automation needs to create VM snapshots and read blob storage, but nothing else. 'Contributor' is too broad — it can also delete VMs. There's no built-in that combines just those two capabilities. So you write a custom role with exactly `Microsoft.Compute/snapshots/write` + `Microsoft.Storage/storageAccounts/blobServices/read`.
>
> Downsides of custom roles: harder to audit ('who has this role and what does it grant?'), version drift across environments, and they don't automatically get new sub-permissions when Azure adds new APIs. So the operational tax is real.
>
> Interview trap: candidates who default to custom roles look inexperienced. The right instinct is 'built-in first, custom only when justified.'"

---
---

# 3. Virtual Machines

## Concept

Azure Virtual Machines are IaaS compute — rent a VM by the hour, install what you want.

## AWS Equivalent

**Azure VM = AWS EC2 Instance**

Same fundamental service. Same use cases. Different terminology and defaults.

## Key Differences from AWS

**1. VMs Have Multiple Companion Resources**
When you create an AWS EC2 instance, you get: the instance, an EBS volume, and (optionally) an ENI. That's roughly one thing conceptually.

When you create an Azure VM, you get *separate resources*:
- Virtual Machine (compute)
- Network Interface Card (NIC)
- Public IP address (separate resource, not attached to NIC by default)
- OS Disk (managed disk, separate resource)
- Data Disks (separate resources)
- Network Security Group (attached to NIC or subnet)

**Implication:** deleting the VM does NOT delete its disk or NIC by default. You have to clean up separately, or use the "delete with VM" option, or nuke the RG. Common gotcha for AWS engineers — you'll accumulate orphaned resources fast.

**2. VM Sizes Instead of Instance Types**
AWS uses `t3.micro`, `m5.large`, `c5.xlarge`. Azure uses `Standard_B1s`, `Standard_D2s_v5`, `Standard_E4ds_v5`.

Series letter meanings:
- **B** = Burstable (like AWS `t` family) — cheap, low CPU baseline with bursts
- **D** = General purpose (like AWS `m`) — balanced
- **E** = Memory optimized (like AWS `r`) — DB workloads
- **F** = Compute optimized (like AWS `c`) — CPU heavy
- **N** = GPU (like AWS `p`, `g`)
- **L** = Storage optimized (like AWS `i`)

Number = vCPU count roughly. Suffix `s` = premium storage support. `v5` = generation.

**3. Availability Sets vs Availability Zones**
Azure has TWO HA constructs:
- **Availability Set**: spreads VMs across fault domains WITHIN one datacenter — protects against rack failure, not AZ failure. Older concept.
- **Availability Zone**: spreads VMs across separate physical datacenters — real HA. Newer, better, use this.

For anything new, use AZs. Availability Sets are a legacy pattern from before AZs existed in Azure.

**4. Pricing Model**
- Pay-as-you-go (like AWS on-demand)
- **Reserved Instances** — commit 1 or 3 years, up to 72% off
- **Spot VMs** — like AWS Spot, up to 90% off, evictable
- **Azure Hybrid Benefit** — reuse on-prem Windows/SQL licenses, huge for enterprises

## Real Configuration

```bash
# Create a resource group first
az group create --name rg-vm-lab --location eastus

# Create a Linux VM (this creates NIC, public IP, disk, NSG automatically)
az vm create \
  --resource-group rg-vm-lab \
  --name vm-web-01 \
  --image Ubuntu2204 \
  --size Standard_B1s \
  --admin-username azureuser \
  --generate-ssh-keys \
  --public-ip-sku Standard

# List VMs
az vm list --output table

# Get public IP
az vm show -d --resource-group rg-vm-lab --name vm-web-01 --query publicIps -o tsv

# SSH in
ssh azureuser@<public-ip>

# Stop (still billed for allocated resources)
az vm stop --resource-group rg-vm-lab --name vm-web-01

# Deallocate (stops billing for compute — different from stop!)
az vm deallocate --resource-group rg-vm-lab --name vm-web-01

# Start
az vm start --resource-group rg-vm-lab --name vm-web-01

# Delete
az vm delete --resource-group rg-vm-lab --name vm-web-01 --yes
```

## The Stop-vs-Deallocate Gotcha

This trips up every AWS engineer. In Azure:

- **Stop** = OS shuts down, but VM is still "Allocated" — you're STILL BEING BILLED for compute
- **Deallocate** = VM released from the underlying host, compute billing STOPS (storage still charged)

In AWS, stopping an EC2 instance immediately stops compute billing. Not in Azure.

**Always deallocate, not stop, for cost savings on non-running VMs.** Missing this has caused real bill shock — imagine 20 "stopped" dev VMs racking up $5k/month.

## User Data → Custom Script Extension / cloud-init

AWS user_data has two Azure equivalents:

**cloud-init** — for Linux, runs at first boot, same concept as AWS user_data:
```bash
az vm create \
  --resource-group rg-vm-lab \
  --name vm-web-01 \
  --image Ubuntu2204 \
  --admin-username azureuser \
  --generate-ssh-keys \
  --custom-data cloud-init.yaml
```

**Custom Script Extension** — runs scripts on already-running VMs:
```bash
az vm extension set \
  --resource-group rg-vm-lab \
  --vm-name vm-web-01 \
  --name customScript \
  --publisher Microsoft.Azure.Extensions \
  --settings '{"fileUris": ["https://storage.blob.core.windows.net/scripts/install.sh"], "commandToExecute": "./install.sh"}'
```

## Production Use Case

**Standard 3-tier web app on Azure VMs:**

```
rg-prod-web-eastus
├── VNet: vnet-prod-eastus (10.0.0.0/16)
├── Subnet: snet-web (10.0.1.0/24)
│   ├── VM: vm-web-01 (AZ 1) — Standard_D2s_v5
│   ├── VM: vm-web-02 (AZ 2) — Standard_D2s_v5
│   └── VM: vm-web-03 (AZ 3) — Standard_D2s_v5
├── Load Balancer: lb-web (Public, Standard SKU)
│   └── Backend pool = the 3 VMs
└── NSG: nsg-web (allow 443 from Internet, 22 from bastion only)

Each VM: OS disk (Premium SSD), cloud-init installs Docker + monitoring agent
Managed identity assigned → pulls secrets from Key Vault
```

Deploy via Terraform or Bicep, not portal, for anything real.

## Console Path

- Portal → **Virtual machines** (search bar)
- Create VM: **+ Create** → **Azure virtual machine** → wizard
- VM detail: **Overview** blade → Start / Stop / Restart / Connect buttons
- **Networking** blade shows NIC, NSG rules, public IP
- **Disks** blade shows attached OS and data disks
- **Extensions + applications** blade for post-provisioning scripts

## Memory Hint

- **B-series** = Basic/Burstable (like AWS t3)
- **D-series** = Default (like AWS m5)
- **E-series** = Extra memory (like AWS r5)
- **F-series** = Fast CPU (like AWS c5)
- **Stop ≠ Deallocate** — this is the money-losing trap

## Interview Q&A — Virtual Machines

**Q: Migrating an EC2-based workload to Azure — what surprises AWS engineers?**

> "Three big ones.
>
> First, the resource sprawl. An EC2 instance is one thing. An Azure VM is five to seven separate resources — VM, NIC, public IP, OS disk, data disks, NSG. All need lifecycle management. If you delete the VM through the console, its disk and NIC often stick around costing money. Solution: put all VM-related resources in a dedicated RG and delete the RG when done, or use `--delete-with-vm` flags.
>
> Second, the stop-vs-deallocate distinction. AWS `stop` immediately stops compute billing. Azure has both — `stop` keeps you billed, `deallocate` actually stops billing. Every AWS engineer touching Azure for the first time gets bitten by this. If you have non-prod VMs that don't need to run overnight, deallocate them; use Azure Automation runbooks to schedule this.
>
> Third, Availability Zones aren't universal. Azure has 60+ regions but not all support AZs. If HA design requires multi-AZ, verify region support first — you don't want to design for AZs, deploy, then discover your chosen region only has Availability Sets.
>
> Positive surprise: Managed Identities are cleaner than instance profiles. Enable on the VM, code just works with Azure SDK. No STS credentials fetching in application code."

**Q: How would you handle patching for a fleet of 100 Azure VMs?**

> "For fleet patching I'd use Azure Update Manager — it's the native service, replaces the old Automation Update Management. Enrol the VMs, define maintenance windows per environment, patches apply during the window with automatic reboot handling and pre-scripts if needed.
>
> Structure the fleet with tags so you can group updates: `env=prod`, `env=dev`, `role=web`. Dev environments get patches immediately after Patch Tuesday. Prod gets them a week later with change management approval. Same wave-based rollout pattern as AWS SSM Patch Manager.
>
> For the truly modern approach though — I'd push for replacing patching with immutable infrastructure. VMs never get patched in place. Instead: Packer builds a new VM image weekly with latest patches. VMSS with new image rolls out gradually. Old VMs get destroyed. This is what I'd aim for as a maturity target, not what I'd necessarily start with on day 1.
>
> Monitoring: Azure Monitor + Log Analytics workspace with the Update Compliance workbook. Any VM more than 30 days behind on critical patches fires an alert. Compliance dashboards get exported weekly for security review."

**Q: You need to run a batch job that takes 4 hours, uses 32 cores, and cost matters more than reliability. What Azure compute would you choose?**

> "Spot VM, without question. Up to 90% discount vs pay-as-you-go for the same hardware. Perfect fit — batch job is fault-tolerant, restartable, cost-sensitive, time isn't super critical.
>
> Choose an F-series or D-series VM with 32 vCPUs — `Standard_F32s_v2` for compute-heavy work. Set eviction policy to 'Deallocate' so if evicted you can restart later without losing the disk. Set max price to pay-as-you-go rate so you pay whatever's current spot price up to that ceiling.
>
> Wrap it in a script that checkpoints progress to Blob Storage every 15 minutes. If evicted mid-run, the restart picks up from last checkpoint.
>
> If the batch is a long-term recurring pattern, upgrade the design: use Azure Batch service, which manages spot VM pools automatically, handles retries, and gives you a job queue abstraction. Same underlying spot VMs, less script glue. That's the AWS Batch equivalent.
>
> For interviews, they're checking whether you know Spot exists AND when to use it. The answer isn't 'always Spot to save money' — Spot for fault-tolerant workloads, on-demand for user-facing latency-sensitive stuff, Reserved for known steady workloads."

---
---

# 4. Managed Disks

## Concept

Azure Managed Disks are the persistent block storage attached to VMs. Called "managed" because Azure handles the underlying storage account layer for you — you just deal with disks.

## AWS Equivalent

**Azure Managed Disk = AWS EBS Volume**

Same concept. Same lifecycle model (persistent, snapshotable, resizable). Different tier names and defaults.

## Key Differences from AWS

**1. Four Performance Tiers (vs AWS's gp3/io2/st1/sc1)**

| Azure Tier | AWS Equivalent | Use Case |
|---|---|---|
| **Standard HDD** | AWS `st1`/`sc1` | Dev/test, backups, cold data |
| **Standard SSD** | AWS `gp2` (lower end) | Web servers, light workloads |
| **Premium SSD** | AWS `gp3` / `io1` | Production apps, databases |
| **Ultra Disk** | AWS `io2 Block Express` | High-transaction databases, latency-critical |

**Premium SSD is the sane default for production.** Standard SSD for dev. Ultra Disk only when you can prove you need it — it's expensive and requires specific VM sizes.

**2. Bursting Behavior**
Premium SSD supports **credit-based bursting** — small disks can burst above their baseline IOPS for a limited time. Same idea as AWS `gp3` but with a credit system. Useful for spiky workloads on small disks.

**3. Zone-Redundant Storage (ZRS) Tier**
For Premium SSD and Standard SSD, you can pick **ZRS** which replicates the disk across three AZs synchronously. AWS EBS doesn't have this — you'd use snapshots + cross-AZ restore. ZRS gives you HA at the disk level directly. Costs more but simplifies HA design.

**4. Snapshots Work Differently**
Azure snapshots are:
- Full or incremental (you choose)
- Stored in either Standard HDD (cheaper) or Premium SSD (faster restore)
- Cross-region: use `az snapshot copy` or Azure Backup vault

AWS snapshots are always incremental after the first, always stored in S3 behind the scenes. Azure gives you more control but also more decisions to make.

## Real Configuration

```bash
# Create a data disk
az disk create \
  --resource-group rg-vm-lab \
  --name disk-data-01 \
  --size-gb 128 \
  --sku Premium_LRS \
  --location eastus

# SKU options: Standard_LRS, StandardSSD_LRS, Premium_LRS, UltraSSD_LRS
# Add _ZRS to any for zone-redundancy: Premium_ZRS, StandardSSD_ZRS

# Attach to VM
az vm disk attach \
  --resource-group rg-vm-lab \
  --vm-name vm-web-01 \
  --name disk-data-01

# On the VM (Linux): initialize and mount
sudo lsblk                     # find the new disk, likely /dev/sdc
sudo mkfs.ext4 /dev/sdc
sudo mkdir /data
sudo mount /dev/sdc /data
echo '/dev/sdc /data ext4 defaults,nofail 0 0' | sudo tee -a /etc/fstab

# Resize (online, no downtime for supported scenarios)
az disk update --resource-group rg-vm-lab --name disk-data-01 --size-gb 256
# Then on the VM: sudo growpart /dev/sdc 1 && sudo resize2fs /dev/sdc1

# Create a snapshot
az snapshot create \
  --resource-group rg-vm-lab \
  --name snap-disk-data-01-$(date +%Y%m%d) \
  --source disk-data-01
```

## Production Use Case

**Database VM disk layout:**
```
vm-mysql-01 (Standard_E4ds_v5, 4 vCPU, 32GB RAM)
├── OS Disk        (Premium SSD, 128GB) — /
├── Data Disk 1    (Premium SSD, 1TB, LRS)  — /var/lib/mysql
├── Data Disk 2    (Premium SSD, 512GB, LRS) — /var/log/mysql
└── Backup target: Azure Backup vault, daily snapshots, 30-day retention
```

Why split data and logs onto separate disks? Same reason as AWS — separate IOPS budgets, easier to snapshot data disk without hitting log I/O, cleaner to resize independently.

## Console Path

- Portal → **Disks** (search bar)
- Or navigate: VM → **Disks** blade → shows OS disk + attached data disks with `Create and attach a new disk` / `Attach existing disks` options
- Snapshots → **Snapshots** (separate resource type in search)

## Memory Hint

- **LRS** = Locally Redundant (single AZ, cheapest)
- **ZRS** = Zone Redundant (3 AZs, HA built-in)
- **Premium SSD** = default production choice, always
- **Ultra Disk** = "I have measured IOPS need > 20,000" or don't bother

## Interview Q&A — Managed Disks

**Q: Compare Azure Managed Disks tiers to AWS EBS. When would you use Ultra Disk?**

> "The tiers map roughly one-to-one. Standard HDD = AWS `st1` for backup and cold workloads. Standard SSD = AWS `gp2` for basic production. Premium SSD is the workhorse tier — equivalent to AWS `gp3` — this is what I'd use by default for anything in production.
>
> Ultra Disk equates to AWS `io2 Block Express`. It's for high-transaction databases where sustained IOPS above 20,000 and sub-millisecond latency matter. Think Oracle RAC, high-frequency trading data, mission-critical OLTP.
>
> The catch with Ultra Disk: it doesn't work with all VM sizes, has to be in a specific zone, can't be the OS disk. It's optimized for specific extreme workloads, not general use.
>
> Practical decision rule: start with Premium SSD, benchmark actual IOPS and latency needs, only move to Ultra Disk if data shows you need it. Ultra costs multiples more per GB and comes with operational complexity."

---
---

# 5. Blob Storage

## Concept

Azure Blob Storage is object storage — same category as AWS S3 or GCP Cloud Storage. Store files as objects, access via HTTP API, essentially unlimited scale.

## AWS Equivalent

**Azure Blob Storage = AWS S3**

Same category, similar features, different terminology and naming rules.

## Key Differences from AWS

**1. Storage Account Wraps Everything**
This is a real mental-model shift. AWS: you create a bucket. Azure: you create a **Storage Account** first, and inside that account you create **containers** (which are like buckets).

```
Storage Account (globally unique name, has performance tier, redundancy, endpoints)
├── Blob Service
│   ├── Container 1  (like an S3 bucket — but scoped to this account)
│   ├── Container 2
│   └── Container 3
├── File Service (SMB shares — like AWS EFS)
├── Queue Service (like AWS SQS)
└── Table Service (basic NoSQL key-value — cheap alternative to Cosmos DB)
```

A single Storage Account can host all four service types. This is why the naming levels are: `storage_account.blob.core.windows.net/container/blob.png`.

**2. Three Blob Types**
- **Block blobs** — regular files (99% of use cases). Equivalent to S3 objects.
- **Append blobs** — optimized for append-only logs
- **Page blobs** — random-access, used for VHDs (Azure Managed Disks are actually page blobs under the hood)

For interviews and DevOps day-to-day: block blobs. Others exist, know they exist, move on.

**3. Access Tiers**

| Azure Tier | AWS S3 Class | Min Storage Duration |
|---|---|---|
| **Hot** | Standard | None |
| **Cool** | Standard-IA | 30 days |
| **Cold** | Glacier Instant Retrieval | 90 days |
| **Archive** | Glacier Deep Archive | 180 days |

Same lifecycle logic as S3: hot data cheap to access, cold data cheap to store expensive to access. **Archive** requires rehydration (up to 15 hours) before you can read the blob — same as Glacier Deep Archive.

**4. Redundancy Options (Not Just One Region)**

| Option | Copies | Location | AWS Equivalent |
|---|---|---|---|
| **LRS** | 3 | Single AZ | No direct AWS parallel |
| **ZRS** | 3 | Three AZs | S3 (default multi-AZ) |
| **GRS** | 6 | Primary region + paired region | S3 Cross-Region Replication |
| **RA-GRS** | 6 | Same as GRS + read from secondary | S3 CRR with cross-region reads |
| **GZRS** | 6 | ZRS + geo-redundant | Best combined durability |

You pick this at Storage Account level. Changing later requires migration effort. Default in new accounts: `RA-GRS` or `GRS` depending on region.

**5. SAS Tokens Are the Common Access Pattern**
Shared Access Signature (SAS) tokens are Azure's answer to S3 pre-signed URLs — time-limited, scope-limited URL that grants access without needing IAM credentials.

Three types:
- **User Delegation SAS** — signed by Entra ID credentials (preferred, auditable)
- **Service SAS** — signed by account key (works but less secure)
- **Account SAS** — broadest scope, use sparingly

## Real Configuration

```bash
# Create storage account (name must be 3-24 chars, lowercase, unique globally)
az storage account create \
  --name stdevopsaakash01 \
  --resource-group rg-storage-lab \
  --location eastus \
  --sku Standard_LRS \
  --kind StorageV2 \
  --access-tier Hot

# Get account key
az storage account keys list --resource-group rg-storage-lab --account-name stdevopsaakash01

# Create a container
az storage container create \
  --name uploads \
  --account-name stdevopsaakash01 \
  --auth-mode login

# Upload a file
az storage blob upload \
  --account-name stdevopsaakash01 \
  --container-name uploads \
  --name hello.txt \
  --file ./hello.txt \
  --auth-mode login

# List blobs
az storage blob list \
  --account-name stdevopsaakash01 \
  --container-name uploads \
  --auth-mode login \
  --output table

# Download
az storage blob download \
  --account-name stdevopsaakash01 \
  --container-name uploads \
  --name hello.txt \
  --file ./downloaded.txt \
  --auth-mode login

# Generate a SAS token for a specific blob (read-only, 1 hour)
az storage blob generate-sas \
  --account-name stdevopsaakash01 \
  --container-name uploads \
  --name hello.txt \
  --permissions r \
  --expiry $(date -u -d '1 hour' '+%Y-%m-%dT%H:%MZ') \
  --auth-mode login \
  --as-user
```

## Production Use Case

**Application file uploads with least-privilege access:**

```
Storage Account: stprodwebfiles (GZRS for durability)
├── Container: user-uploads (private)
├── Container: processed (private)
└── Container: public-assets (public read via CDN)

App tier (AKS):
├── Managed Identity assigned
├── Role: Storage Blob Data Contributor scoped to user-uploads
└── Uploads directly via SDK, no keys

Client browsers:
├── Backend generates User Delegation SAS token (5 min TTL)
└── Client uploads directly to blob via SAS URL

Lifecycle:
├── uploads/*: Hot for 30 days → Cool for 90 → Archive after 1 year
└── Delete after 7 years (compliance retention)
```

Note: **no account keys anywhere**. Backend uses managed identity, clients get short-lived SAS tokens signed by user delegation. Same maturity level as AWS S3 with IAM roles + presigned URLs.

## Console Path

- Portal → **Storage accounts** (search)
- Storage account detail:
  - **Containers** (left blade) → click into container → upload/manage blobs
  - **Access keys** — the account keys (avoid using directly)
  - **Shared access signature** — generate SAS
  - **Configuration** — access tier, redundancy
  - **Lifecycle management** — rules for tier transitions and expiration
  - **Data protection** — versioning, soft delete, immutability



## Memory Hint

- **Storage Account** = the AWS "bucket" concept split into two layers (account + container)
- **Hot / Cool / Cold / Archive** — temperature = access frequency (hot = frequent)
- **LRS < ZRS < GRS < GZRS** — increasing redundancy, increasing cost
- **SAS = pre-signed URL** — time-limited, scoped, safer than sharing keys

## Interview Q&A — Blob Storage

**Q: Compare Blob Storage to S3. What surprises AWS engineers?**

> "Three things.
>
> First, the Storage Account layer. In AWS you just create a bucket. In Azure you first create a Storage Account with performance tier and redundancy settings, THEN create containers inside it. The Storage Account is the pricing and redundancy boundary; containers are just organization within it. AWS engineers try to create 'buckets' and get confused by the wrapper.
>
> Second, redundancy is more granular. AWS S3 is multi-AZ by default in a region, with CRR as an add-on for cross-region. Azure gives you five choices — LRS, ZRS, GRS, RA-GRS, GZRS — spanning single-AZ up to multi-region-multi-AZ. You explicitly pick this per storage account. It's more control but more decisions.
>
> Third, SAS tokens are richer than S3 pre-signed URLs. You can generate SAS at container level (not just blob), grant multiple permissions in one token, and use User Delegation SAS which is signed by Entra ID credentials so it's auditable per user. AWS pre-signed URLs are simpler but less granular.
>
> Practical gotcha: Storage Account names must be globally unique across all of Azure — like S3 bucket names. Reserve yours early or you'll fight for names. Standard practice: prefix with `st` (for storage), then a domain identifier."

**Q: Design a static website hosting solution on Azure — HTTPS, custom domain, global CDN. Compare to AWS S3+CloudFront."**

> "The Azure equivalent of S3 + CloudFront + Route 53 + ACM is Blob Storage + Front Door + Azure DNS + managed certificates.
>
> Storage Account with 'Static website' feature enabled — this creates a `$web` container automatically and gives you a primary endpoint URL. Upload your build artifacts there.
>
> Azure Front Door in front of the storage account. Front Door is Azure's global L7 load balancer / CDN combined — think CloudFront + AWS Global Accelerator merged into one product. Origin points to the static website endpoint. Front Door handles global caching at 100+ edge POPs, HTTPS with managed certs (free), and WAF.
>
> Azure DNS zone for the domain. CNAME `www` and Alias record on the apex pointing to Front Door.
>
> Managed certificate on Front Door for HTTPS — auto-renewing, no ACM equivalent to manage.
>
> Result: HTTPS static site with global CDN, custom domain, DDoS protection via Front Door, sub-100ms load times worldwide. Same as the AWS pattern. Slight edge to Azure here — Front Door bundles CDN + WAF + global load balancing in one service, whereas in AWS you compose CloudFront + WAF + Route 53 separately.
>
> Cost: for a small static site, both are under $10/month. Front Door pricing is a bit different — you pay per rule and per request, plus data egress. CloudFront's free tier is more generous for hobby projects."

**Q: How do you protect a Blob container from accidental exposure?**

> "Layered defense, same principle as AWS S3 protection.
>
> Layer 1: Storage account level — disable public blob access at the account. Under Configuration, set 'Allow Blob public access' to Disabled. This overrides container-level settings — nothing in this account can be public. Analogous to S3 'Block Public Access' at account level.
>
> Layer 2: Network restrictions. Configure Storage Firewall to allow access only from specific VNets and IP ranges. For internal apps, use Private Endpoints — the storage account becomes reachable only via a private IP inside your VNet, not the public endpoint at all.
>
> Layer 3: Identity-based access only. Turn off Shared Key access if possible. Force everything to use Entra ID authentication. This makes SAS tokens and managed identities the only access paths, and every access is auditable to a specific identity.
>
> Layer 4: Encryption. Enabled by default at rest with Microsoft-managed keys. For compliance workloads, use customer-managed keys (CMK) via Azure Key Vault — same pattern as S3 with KMS.
>
> Layer 5: Monitoring. Enable diagnostic logs to Log Analytics. Alert on unusual patterns: mass downloads, access from unexpected countries, sudden spike in failed auth attempts. Same defense-in-depth as AWS S3 with CloudTrail.
>
> Real practice: I audit storage accounts monthly with Azure Policy — enforce 'no public blob access' as a compliance rule, and any non-compliant account gets flagged automatically. Same as AWS Config rules for S3 public access."

---
---

# 6. Load Balancer, App Gateway, Front Door

Azure fragments what AWS bundles into ALB/NLB across **four distinct services**. Interviews test whether you know which to pick.

## The Four Options

| Service | Layer | Scope | AWS Equivalent | Use Case |
|---|---|---|---|---|
| **Load Balancer (Basic/Standard)** | L4 (TCP/UDP) | Regional | AWS NLB | High-perf TCP/UDP, internal services |
| **Application Gateway** | L7 (HTTP/HTTPS) | Regional | AWS ALB | Web apps, path/host routing, WAF |
| **Front Door** | L7 (HTTP/HTTPS) | Global | AWS CloudFront + Global Accelerator | Global apps, CDN, edge routing |
| **Traffic Manager** | DNS-level | Global | AWS Route 53 with routing policies | DNS-based geo/failover routing |

**Rule of thumb:**
- HTTP web traffic in one region → **Application Gateway**
- HTTP web traffic globally → **Front Door**
- TCP/UDP (databases, custom protocols) → **Load Balancer**
- DNS-based routing across regions → **Traffic Manager**

## Load Balancer (Standard SKU) — L4

Regional, high-performance L4 load balancer. Handles TCP and UDP. Millions of connections per second, sub-millisecond latency.

- **Public** — internet-facing (frontend gets public IP)
- **Internal** — private, front-end IP inside a VNet

**Two SKUs:** Basic (legacy, being deprecated) and Standard. Always use Standard for new deployments — it's the one with AZ support, HA, and modern features.

## Application Gateway — L7

Regional L7 load balancer. Understands HTTP/HTTPS, does SSL termination, path-based routing, host-based routing, WAF integration.

**Direct AWS ALB analogue.** If you know ALB deeply, App Gateway is a smooth transition — same mental model.

Key features:
- Path routing: `/api/*` → api backend pool, `/admin/*` → admin pool
- Host routing: `api.myapp.com` vs `www.myapp.com`
- WAF v2 built-in (OWASP rules, custom rules)
- Session affinity
- SSL termination + end-to-end TLS
- URL redirect / rewrite

## Front Door — Global L7

Global anycast network. **This is the killer service without a direct AWS single-service equivalent** — it combines CloudFront (CDN) + Global Accelerator (anycast routing) + ALB (L7 routing) + WAF into one.

Use cases:
- Global web apps needing low latency worldwide
- Multi-region active-active with automatic failover
- Static content CDN with dynamic content acceleration
- WAF at edge

## Real Configuration

**Application Gateway (basic web app):**
```bash
# Create the App Gateway (this takes 5-10 mins)
az network application-gateway create \
  --name agw-prod-web \
  --resource-group rg-prod-web-eastus \
  --location eastus \
  --sku Standard_v2 \
  --priority 100 \
  --capacity 2 \
  --vnet-name vnet-prod-eastus \
  --subnet snet-appgw \
  --public-ip-address pip-agw-prod \
  --servers 10.0.1.10 10.0.1.11 10.0.1.12 \
  --http-settings-port 80 \
  --http-settings-protocol Http

# Enable WAF (upgrade to WAF_v2 SKU)
az network application-gateway update \
  --name agw-prod-web \
  --resource-group rg-prod-web-eastus \
  --sku WAF_v2

# Add a path-based routing rule for /api
az network application-gateway url-path-map create \
  --gateway-name agw-prod-web \
  --resource-group rg-prod-web-eastus \
  --name api-path-map \
  --paths /api/* \
  --address-pool api-pool \
  --http-settings apiHttpSettings \
  --default-address-pool webPool \
  --default-http-settings appGatewayBackendHttpSettings
```

## Production Use Case

**Multi-region active-active web app with Front Door:**
```
Front Door Profile (global, single entry point: myapp.azurefd.net)
├── Route: /*  →  Origin Group: web-origins
│                 ├── Origin: agw-eastus.eastus.cloudapp.azure.com (priority 1)
│                 └── Origin: agw-westeurope.westeurope.cloudapp.azure.com (priority 1)
│                 (equal priority = latency-based routing to closest)
│
└── WAF Policy: block SQL injection, XSS, rate-limit /login to 10/min per IP

Each Application Gateway:
├── SSL termination with managed cert
├── Backend pool: 3 VMs across 3 AZs
└── Health probe /health, 15s interval
```

User in India hits Front Door edge in Chennai → routed to eastus origin (closer than westeurope) → App Gateway → backend VM. Failover if eastus origin unhealthy.

## Console Path

- **Load balancer** — Portal → "Load balancers"
- **Application Gateway** — Portal → "Application gateways"
- **Front Door** — Portal → "Front Door and CDN profiles"

Each has: Overview, Frontend IPs, Backend pools, Listeners/Routes, Rules, Health probes, Diagnostic settings.

## Memory Hint

- **Load Balancer** = layer 4, regional, fast (AWS NLB)
- **App Gateway** = layer 7, regional, WAF (AWS ALB)
- **Front Door** = layer 7, GLOBAL, CDN (AWS CloudFront + more)
- "Regional web? App Gateway. Global web? Front Door. TCP? Load Balancer."

## Interview Q&A — Load Balancing

**Q: You need to load-balance a globally distributed web application on Azure. Walk me through your choice.**

> "Front Door, not Application Gateway. Front Door is Azure's global L7 with anycast edge routing — 100+ points of presence worldwide, users hit the nearest edge, traffic then routes to the healthiest and closest backend origin.
>
> Application Gateway is regional — great for a single-region web app, but for global you'd need to deploy App Gateway in each region and then need something in front to route users to the right region. Front Door replaces both layers.
>
> Design: single Front Door profile globally. Origin group contains one App Gateway per region — East US, West Europe, Southeast Asia. Front Door does two things: (1) latency-based routing so users hit the closest region, and (2) automatic failover if a region's App Gateway fails health probes.
>
> Add WAF at the Front Door layer — protects all regions with one policy. Add CDN caching for static assets so `/static/*` serves from edge without hitting origins at all.
>
> AWS equivalent: this is CloudFront + AWS Global Accelerator + ALB per region. Azure bundles them into Front Door + App Gateway pairs. Slightly cleaner Azure architecture in my opinion, though AWS gives you more discrete control."

**Q: When would you use Traffic Manager instead of Front Door?**

> "Traffic Manager is DNS-based routing — it returns different DNS answers based on your routing method (geographic, weighted, priority, performance). Front Door does actual traffic proxying at the L7 level.
>
> Use Traffic Manager when: you're routing to non-HTTP endpoints (databases, custom TCP services). Or when clients need to connect directly to the backend for performance reasons and you can't tolerate the extra proxy hop. Or when you need DNS-only routing without any acceleration.
>
> Use Front Door when: HTTP/HTTPS traffic where you want L7 features — WAF, path-based routing, session affinity, caching, SSL offload.
>
> Cost matters too — Traffic Manager is cheaper because it's just DNS. Front Door prices per rule and per GB proxied. For a small internal DNS failover setup, Traffic Manager makes sense. For customer-facing web apps, Front Door.
>
> AWS analogue: Traffic Manager is closest to Route 53 with routing policies. Front Door is CloudFront-like.
>
> Real practical note: some enterprises use BOTH. Traffic Manager routes between fundamentally different backends (like AWS vs Azure vs on-prem for hybrid setups), Front Door handles the web edge within Azure."

---
---

# 7. VMSS — Auto Scaling

## Concept

Virtual Machine Scale Sets — deploy and manage a fleet of identical VMs that scale in/out based on demand.

## AWS Equivalent

**VMSS = AWS Auto Scaling Group + Launch Template combined**

Same fundamental purpose. Slightly different mental model — VMSS is a single resource that encompasses both the "how to launch" definition and the "how many to run" logic. AWS separates Launch Templates from ASGs; Azure unifies them.

## Key Differences from AWS

**1. VMSS Modes: Uniform vs Flexible**

- **Uniform orchestration** — every VM identical, single VM size, faster scale, less flexibility. Older model.
- **Flexible orchestration** — mix VM sizes, spot + on-demand together, more like modern ASG mixed-instance policy. **Prefer this for new deployments.**

**2. Native Azure Integration**
VMSS integrates directly with:
- Load Balancer / App Gateway backends (auto-registers)
- Managed Identity (whole scale set gets one identity)
- Application Health extension (custom health signals)
- Automatic OS image updates (rolling patching built-in)

**3. Autoscale Rules Are More Expressive**
Azure autoscale supports multiple simultaneous rules with time-based schedules and can consume Application Insights custom metrics natively. Similar power to AWS but different syntax.

## Real Configuration

```bash
# Create VMSS with autoscaling
az vmss create \
  --resource-group rg-prod-web-eastus \
  --name vmss-web-prod \
  --image Ubuntu2204 \
  --vm-sku Standard_D2s_v5 \
  --instance-count 3 \
  --admin-username azureuser \
  --generate-ssh-keys \
  --vnet-name vnet-prod-eastus \
  --subnet snet-web \
  --load-balancer lb-web \
  --upgrade-policy-mode automatic \
  --orchestration-mode Flexible \
  --zones 1 2 3

# Add autoscale profile
az monitor autoscale create \
  --resource-group rg-prod-web-eastus \
  --resource vmss-web-prod \
  --resource-type Microsoft.Compute/virtualMachineScaleSets \
  --name autoscale-web \
  --min-count 3 \
  --max-count 20 \
  --count 3

# Add scale-out rule: CPU > 70% for 5 min → add 2 instances
az monitor autoscale rule create \
  --resource-group rg-prod-web-eastus \
  --autoscale-name autoscale-web \
  --condition "Percentage CPU > 70 avg 5m" \
  --scale out 2

# Add scale-in rule: CPU < 30% for 10 min → remove 1 instance
az monitor autoscale rule create \
  --resource-group rg-prod-web-eastus \
  --autoscale-name autoscale-web \
  --condition "Percentage CPU < 30 avg 10m" \
  --scale in 1
```

## Production Use Case

**Web tier with predictable business-hours pattern:**

```
VMSS: vmss-prod-web (Flexible orchestration, 3 AZs)
├── Baseline: 5 instances (24/7)
├── Business hours (9AM-6PM IST weekdays): scale to 15 minimum
├── Reactive scaling:
│   ├── CPU > 70% for 5 min → +3 instances (aggressive scale-out)
│   ├── HTTP request rate > 500/sec per instance → +5 instances
│   └── CPU < 30% for 15 min → -1 instance (gentle scale-in)
├── Max cap: 40 instances (cost guardrail)
└── Health check: App Gateway probe on /health, 15s interval

Rolling upgrade policy: 20% batch, automatic if health probe passes
Application Health extension reports app-level health → VMSS replaces bad instances
```

## Console Path

- Portal → **Virtual machine scale sets**
- Overview → Instance count, running/stopped counts, load balancer status
- **Scaling** blade → autoscale rules, schedules
- **Instances** blade → list of individual VMs, can SSH, reimage, upgrade individually
- **Upgrade policy** blade → automatic / manual / rolling settings

## Memory Hint

- **VMSS** = "fleet of clones with autoscale" (AWS ASG merged with Launch Template)
- **Flexible** = use this (mixed sizes, spot, modern)
- **Uniform** = don't use (legacy, single size only)
- **Cool-down** ≈ Azure's version of AWS scaling cooldown, prevents thrashing

## Interview Q&A — VMSS

**Q: Coming from AWS Auto Scaling Groups, what's the mental model shift for VMSS?**

> "AWS separates concerns: Launch Template defines what an instance looks like, ASG defines scaling policy. Two resources referencing each other.
>
> VMSS bundles both. One resource contains the VM spec (image, size, extensions, network config) and the scaling behavior. Simpler to think about in some ways, less composable in others.
>
> Practical differences that matter day-to-day:
>
> First, orchestration mode. Choose Flexible for new deployments — supports mixed VM sizes and spot instances in the same set, closer to modern ASG mixed-instance policy. Uniform is single-size only, older, still around for compatibility.
>
> Second, rolling upgrades are first-class. You update the VMSS model (change image, extension config, size) and it rolls the change through instances in configurable batches. AWS you'd script this or use CodeDeploy. Azure builds it in.
>
> Third, health signals. Beyond load balancer health probes, VMSS has the Application Health extension — your app reports health to a URL, the extension reports to VMSS which uses it for instance replacement decisions. More granular than the classic 'ELB health check' pattern.
>
> Fourth, autoscale rules can reference custom Application Insights metrics natively — scale based on queue depth, active user sessions, business metrics. AWS supports this via CloudWatch custom metrics, but the wiring is cleaner in Azure."

**Q: You inherit a VMSS running Uniform orchestration in production. Would you migrate it to Flexible? What's the process?**

> "Yes, I'd migrate it — but carefully. Flexible unlocks spot integration, mixed VM sizes, and better zone distribution. Long-term maintenance cost of staying on Uniform is higher.
>
> Migration path: you can't in-place convert. You deploy a new Flexible VMSS in parallel, weight traffic between them via the load balancer, then decommission the old one.
>
> Steps:
> 1. Provision the new Flexible VMSS with matching configuration
> 2. Add it to the same load balancer backend pool alongside the existing Uniform set
> 3. Verify traffic hitting both, both pass health checks
> 4. Gradually reduce Uniform capacity while increasing Flexible capacity
> 5. Once Flexible carries all traffic and has been stable for a week, remove Uniform from backend, then delete
>
> Watch out for: any external dependencies referencing specific instance names or IDs. VMSS instance names change per set. Anything hard-coded needs updating.
>
> If the environment has strict downtime requirements, plan the migration during a low-traffic window and have rollback ready — the old VMSS stays intact until you're sure the new one is stable."

---
---

# 8. Azure SQL Database

## Concept

Azure SQL Database is the managed relational database service — Microsoft's flagship, deeply tuned. Runs SQL Server engine as a service.

## AWS Equivalent

**Azure SQL Database ≈ AWS RDS for SQL Server, but more service-integrated**

For MySQL/PostgreSQL, Azure has separate services: **Azure Database for MySQL** and **Azure Database for PostgreSQL**. They're managed like RDS but the flagship (Azure SQL) is what interviews mostly focus on.

## Key Differences from AWS

**1. Three Deployment Options**

| Option | AWS Analogue | Use Case |
|---|---|---|
| **Single Database** | RDS single instance | One DB, one workload |
| **Elastic Pool** | (no direct AWS parallel) | Many DBs sharing compute — SaaS multi-tenant |
| **Managed Instance** | RDS Custom | Near-100% SQL Server compat, lift-and-shift |

**Elastic Pool is unique** — you pool compute (DTUs or vCores) across many databases with varying load patterns. Great for SaaS with hundreds of tenant databases where individual load is unpredictable. AWS has no direct equivalent.

**2. Purchasing Models: DTU vs vCore**

- **DTU (Database Transaction Unit)** — bundled abstract unit combining CPU, IOPS, memory. Simpler pricing, harder to tune.
- **vCore** — separate CPU and storage. Modern, better for performance tuning, supports Azure Hybrid Benefit.

**Always pick vCore for anything new.** DTU is legacy.

**3. Service Tiers (Performance/HA)**

- **Basic / Standard / Premium** — DTU model tiers
- **General Purpose** — vCore, standard workloads, single instance with fast failover
- **Business Critical** — vCore, HA with local SSD, sync replicas, read replicas
- **Hyperscale** — vCore, up to 100TB, near-instant point-in-time restore, unlimited read scale-out

Business Critical ≈ AWS RDS Multi-AZ. Hyperscale ≈ Aurora — the "Aurora killer" AWS engineers should evaluate.

**4. Built-in Features That Cost Extra on AWS**
- Automated backups included (35-day retention)
- Automatic tuning (index recommendations, auto-apply)
- Advanced Threat Protection (SQL injection detection)
- Data Discovery & Classification
- Auditing to Log Analytics

Some of these need extra config on RDS or third-party tools.

## Real Configuration

```bash
# Create a logical server (like RDS instance parent)
az sql server create \
  --name sql-prod-aakash \
  --resource-group rg-prod-db-eastus \
  --location eastus \
  --admin-user sqladmin \
  --admin-password 'StrongPassword@2026'

# Configure firewall — allow Azure services + your IP
az sql server firewall-rule create \
  --resource-group rg-prod-db-eastus \
  --server sql-prod-aakash \
  --name AllowAllAzureServices \
  --start-ip-address 0.0.0.0 --end-ip-address 0.0.0.0
# (0.0.0.0/0.0.0.0 is the special value for "allow Azure services")

az sql server firewall-rule create \
  --resource-group rg-prod-db-eastus \
  --server sql-prod-aakash \
  --name AllowMyIP \
  --start-ip-address 203.0.113.42 --end-ip-address 203.0.113.42

# Create the database (General Purpose, 2 vCore)
az sql db create \
  --resource-group rg-prod-db-eastus \
  --server sql-prod-aakash \
  --name db-orders \
  --edition GeneralPurpose \
  --family Gen5 \
  --capacity 2 \
  --zone-redundant false

# Get connection string
az sql db show-connection-string \
  --server sql-prod-aakash \
  --name db-orders \
  --client sqlcmd
```

## Production Use Case

**E-commerce order database, production tier:**

```
Azure SQL Server: sql-prod-orders (Entra ID auth + SQL auth disabled)
├── Database: db-orders
│   ├── Tier: Business Critical, Gen5, 4 vCore, 500GB
│   ├── Zone-redundant: Yes (3-AZ replication)
│   ├── Backup retention: 35 days short-term + 12 months long-term
│   ├── Automatic tuning: enabled (create/drop indexes, force last-good plan)
│   ├── Threat Protection: enabled, alerts to Security Center
│   └── Read replica: 1 (for reporting workload)
├── Networking: Private Endpoint in snet-db (no public access)
├── Auditing: enabled to Log Analytics workspace
└── Access: Managed Identity from app tier, no passwords
```

Reads and writes from app tier via managed identity. Reporting queries route to read replica via connection string with `ApplicationIntent=ReadOnly`.

## Console Path

- Portal → **SQL databases** (search) OR **SQL servers** for the container
- Database detail:
  - **Overview** → connection strings, resource utilization
  - **Query editor (preview)** → run SQL directly in portal
  - **Backup and restore** → point-in-time restore
  - **Automatic tuning** → toggle features
  - **Networking** → firewall rules, private endpoints
  - **Security** → auditing, threat protection, data classification
  - **Replicas** → geo-replication and read scale-out

## Memory Hint

- **Single DB** → one workload
- **Elastic Pool** → SaaS multi-tenant (unique to Azure)
- **Managed Instance** → lift-and-shift from on-prem SQL Server
- **Business Critical** = Multi-AZ (RDS)
- **Hyperscale** = Aurora equivalent
- **Always vCore, never DTU** (for new deployments)

## Interview Q&A — Azure SQL

**Q: Compare Azure SQL Database Business Critical tier to AWS RDS Multi-AZ.**

> "Similar goal, different mechanics.
>
> RDS Multi-AZ maintains a synchronous standby in a different AZ. Standby doesn't serve traffic — pure passive backup. Automatic failover in ~60 seconds.
>
> Azure SQL Business Critical maintains multiple synchronous replicas — one primary + three secondaries — using an Always On Availability Group under the hood. Local SSD for storage, sub-millisecond latency. Failover under 30 seconds typically.
>
> Two Business Critical wins over Multi-AZ:
>
> One: read scale-out is included. Attach `ApplicationIntent=ReadOnly` to your connection string and reads route to a secondary replica automatically. Same feature costs extra on RDS via Read Replicas.
>
> Two: local SSD storage. Business Critical is designed for high-transaction OLTP with sub-ms latency. RDS uses network-attached EBS — good but higher latency floor.
>
> Where RDS wins: cross-region read replicas are easier to configure. Azure SQL supports geo-replication but the setup is slightly clunkier.
>
> If I'm designing a new OLTP database and cost isn't the primary constraint, I'd pick Business Critical over Multi-AZ RDS for the sub-ms latency and free read replica included. If I'm cost-sensitive, General Purpose tier is enough — similar architecture to Multi-AZ RDS."

**Q: A team has 200 SaaS tenants, each with their own database, unpredictable load per tenant. How would you cost-optimize on Azure?**

> "Classic Elastic Pool use case. Azure's answer to this exact problem — AWS doesn't have a native equivalent, teams usually use RDS multi-tenant with schema-per-tenant or database-per-tenant on shared instances.
>
> Elastic Pool: allocate a pool of vCore and storage capacity, then share it across all 200 databases. When Tenant A is idle, its capacity is free for Tenant B who's active. Because tenants rarely all peak simultaneously, you can support 200 databases with maybe 30% of the capacity of provisioning each individually.
>
> Design:
> - One Azure SQL Server, one Elastic Pool with 16 vCore Gen5, 500GB pooled storage
> - All 200 tenant databases attached to this pool
> - Per-database min and max vCore caps so noisy neighbors can't starve others
> - Autoscale the pool based on DTU/vCore utilization
>
> Migration path: script move databases into the pool one at a time, verify performance, iterate on pool sizing.
>
> Reporting story: for cross-tenant analytics you'd typically ETL to a separate warehouse — Synapse or Fabric — rather than querying the pool databases. Pool is optimized for the OLTP workload.
>
> This kind of design question is where knowing an Azure-native feature (Elastic Pool) puts you ahead of engineers who only know AWS patterns."

---
---

# 9. Cosmos DB

## Concept

Azure Cosmos DB is a globally distributed, multi-model NoSQL database. Multiple wire protocols supported: SQL (native), MongoDB, Cassandra, Gremlin, Table.

## AWS Equivalent

**Cosmos DB ≈ AWS DynamoDB + DocumentDB + Neptune, unified**

Closer to DynamoDB in typical usage, but Cosmos does more — it speaks multiple APIs and offers stronger global distribution primitives out of the box.

## Key Differences from AWS

**1. Multi-Model API Selection**

You pick an API at container creation time:

| API | Use Case | AWS Analogue |
|---|---|---|
| **NoSQL (SQL API)** | Default, document DB | DynamoDB |
| **MongoDB** | Compat with existing Mongo apps | DocumentDB |
| **Cassandra** | Column-family workloads | Keyspaces |
| **Gremlin** | Graph traversal | Neptune |
| **Table** | Key-value | DynamoDB in simple mode |

The API can't be changed later. Pick based on existing code — new apps almost always use NoSQL (SQL) API.

**2. Global Distribution Is First-Class**

Cosmos DB was designed for multi-region from day one. Enable geo-replication with a checkbox:
- Add any region → data replicates asynchronously
- Multi-region writes → any region accepts writes, Cosmos merges via configured conflict resolution
- Compare to DynamoDB Global Tables — similar idea but Cosmos feels more integrated and mature

**3. Five Consistency Levels (Not Just Eventual/Strong)**

Cosmos offers a **consistency spectrum**:
- Strong (like DynamoDB strongly consistent read)
- Bounded Staleness
- Session (default — reads see own writes, most apps want this)
- Consistent Prefix
- Eventual (like DynamoDB eventually consistent read)

Each has different latency/availability trade-offs. This is a favorite interview topic — "explain the consistency levels" — because it shows depth beyond just knowing the service exists.

**4. Request Units (RU/s) Instead of RCU/WCU**

Cosmos measures throughput in RUs (Request Units per second). One RU roughly = 1 KB point read cost.

- **Provisioned throughput** — pre-allocate RU/s per container or database
- **Autoscale** — set max RU/s, Cosmos scales between 10%-100% of that automatically
- **Serverless** — pay-per-request, no capacity planning, for spiky low-volume workloads

**5. Change Feed Built In**

Every Cosmos container has a change feed — an ordered log of every modification. Consume it like a Kafka topic. Enables event-driven architectures, materialized views, ETL to other systems. DynamoDB's equivalent is Streams — Cosmos change feed is more general and richer.

## Real Configuration

```bash
# Create Cosmos DB account (NoSQL API)
az cosmosdb create \
  --name cosmos-prod-aakash \
  --resource-group rg-prod-db-eastus \
  --locations regionName=eastus failoverPriority=0 isZoneRedundant=true \
  --locations regionName=westeurope failoverPriority=1 isZoneRedundant=false \
  --default-consistency-level Session \
  --enable-multiple-write-locations true

# Create a database inside the account
az cosmosdb sql database create \
  --account-name cosmos-prod-aakash \
  --resource-group rg-prod-db-eastus \
  --name AppDB

# Create a container with autoscale throughput
az cosmosdb sql container create \
  --account-name cosmos-prod-aakash \
  --resource-group rg-prod-db-eastus \
  --database-name AppDB \
  --name Contacts \
  --partition-key-path "/tenantId" \
  --max-throughput 4000

# List keys (avoid using — prefer Managed Identity)
az cosmosdb keys list \
  --name cosmos-prod-aakash \
  --resource-group rg-prod-db-eastus
```

## Partition Key — Critical Design Decision

Same as DynamoDB — choose partition key carefully. Bad partition key = throttling, hot partitions, unhappy engineers.

Rules:
- High cardinality (many distinct values)
- Even distribution (no partition disproportionately large)
- Aligns with query pattern (queries should filter on partition key when possible)

Example: For a multi-tenant SaaS `Contacts` table, `/tenantId` is usually right — spreads load per tenant, most queries scope to one tenant anyway.

Bad example: `/status` where values are `active/inactive/pending` — only 3 values, poor distribution.

## Production Use Case

**Global e-commerce cart service:**

```
Cosmos DB Account: cosmos-prod-cart
├── APIs: NoSQL (SQL API)
├── Regions:
│   ├── East US (primary, zone-redundant)
│   ├── West Europe (secondary, multi-write enabled)
│   └── Southeast Asia (secondary, multi-write enabled)
├── Consistency: Session (default per-user consistency)
├── Multi-region writes: ON (each region takes writes locally)
│
├── Database: cart-db
│   └── Container: carts
│       ├── Partition key: /userId
│       ├── Throughput: Autoscale 400-40,000 RU/s
│       ├── TTL: 30 days (auto-expire abandoned carts)
│       └── Change feed → Azure Function → analytics pipeline
│
└── Access: Managed Identity (RBAC), no keys
```

User in India adds items to cart → writes to Southeast Asia region locally → replicates to other regions in seconds. User travels to Europe next day, hits West Europe region → sees their cart (session consistency guarantees this because same user).

## Console Path

- Portal → **Azure Cosmos DB** (search)
- Account detail:
  - **Data Explorer** — query and modify data in-portal
  - **Replicate data globally** — add/remove regions with a map interface
  - **Default consistency** — change consistency level
  - **Keys** — connection strings (prefer RBAC)
  - **Scale** — throughput configuration per container
  - **Change feed** — inspect / configure processors

## Memory Hint

- **Cosmos DB** = "DynamoDB Global Tables but on steroids and multi-API"
- **RU/s** = "one point-read of 1KB ≈ 1 RU"
- **Session consistency** = default, use this unless you have a specific reason not to
- **Change Feed** = built-in Kafka-like stream, free with the container
- "Pick partition key like you're picking a DynamoDB hash key — same rules apply"

## Interview Q&A — Cosmos DB

**Q: Compare Cosmos DB to DynamoDB. When would you choose Cosmos?**

> "Similar category, different strengths.
>
> Feature parity for the basics: managed NoSQL, key-value / document, elastic scaling, global distribution. Both handle massive scale.
>
> Cosmos wins on:
>
> First, multi-region writes maturity. Both support it, but Cosmos's design and conflict resolution feels more production-ready. DynamoDB Global Tables have caught up but Cosmos had a head start.
>
> Second, consistency levels. Cosmos offers five levels on a spectrum. DynamoDB gives you strong or eventual. If your app needs 'session' consistency — user always reads their own writes but doesn't care about global ordering — Cosmos gives you that natively without eventual consistency's staleness issues.
>
> Third, multi-model. If you have existing MongoDB or Cassandra code, Cosmos speaks those wire protocols. DynamoDB is DynamoDB — port your code.
>
> DynamoDB wins on:
>
> First, ecosystem integration if you're already deep in AWS. Cosmos DB with Lambda vs DynamoDB with Lambda — the AWS pairing is smoother.
>
> Second, sometimes cost. For simple key-value patterns at scale with predictable load, DynamoDB provisioned mode can be cheaper. Cosmos's flexibility comes at a price premium.
>
> In an interview: if the question is 'what would you choose,' probe requirements first. Multi-region writes needed? Session consistency important? Legacy MongoDB code to migrate? Those tilt to Cosmos. AWS-native shop with simple key-value? DynamoDB is fine."

**Q: Walk me through Cosmos DB consistency levels.**

> "Five levels, ordered from strongest to weakest guarantees.
>
> Strong: linearizable. Any read returns the most recent committed write across all regions. Highest latency, highest cost. Use when correctness beats performance — financial ledgers, inventory counters.
>
> Bounded Staleness: reads can lag writes by a defined number of versions or a time window. So you might be up to 5 seconds or 100 versions behind. Useful when you want mostly-fresh reads but can accept a small staleness bound.
>
> Session: my go-to default. Guarantees read-your-own-writes for a given client session, monotonic reads within the session, but different sessions may see slightly different orderings. Perfect for user-facing apps — the user always sees their most recent action.
>
> Consistent Prefix: never see out-of-order writes. If writes happened in order A, B, C, you might see just A or A,B or A,B,C — never A,C without B. No time bound on freshness though.
>
> Eventual: cheapest, most available, most stale. Reads may lag arbitrarily behind writes. Fine for analytics dashboards where slight delay doesn't matter.
>
> Practical rule: start with Session. Move to Strong for the specific tables that need it — usually few. Never use Eventual for user-facing reads. This is exactly the kind of nuanced answer that separates senior candidates."

**Q: You have a Cosmos container throttling with 429 errors. How do you diagnose?**

> "429 = Request Unit budget exceeded. Systematic diagnosis:
>
> First, check the metric 'Total Request Units' in Azure Monitor. See if consumption is bumping against provisioned throughput. If yes, obvious answer: increase throughput or enable autoscale.
>
> But usually the more interesting answer is uneven partition load — a 'hot partition.' Check the 'Normalized RU Consumption' metric per partition key range. If one partition is at 100% while others are at 5%, you have a hot partition. Total throughput might be fine on paper but one partition is bottlenecked.
>
> Root cause: bad partition key. Skew in the data or query pattern. Example: partition key `/status` with 3 values, 90% of writes go to `active` — that partition drowns while others idle.
>
> Fix: redesign the partition key. This is a rebuild — Cosmos can't change partition key in place. You provision a new container with a better key, dual-write during migration, backfill, cut over. Painful. That's why partition key choice is a big deal upfront.
>
> Second common cause: expensive queries scanning across many partitions. Check for cross-partition queries in the query metrics. Rewrite them to filter on partition key when possible, or add composite indexes.
>
> Third: burst traffic that momentarily exceeds provisioned RU. Enable autoscale so throughput can absorb bursts up to 10x the baseline."

---
---

# 10. Azure Functions

## Concept

Azure Functions is serverless compute — run code in response to events without managing servers.

## AWS Equivalent

**Azure Functions = AWS Lambda**

Same category, similar developer experience. Different hosting plans and pricing model.

## Key Differences from AWS

**1. Hosting Plans (Not Just One Model)**

AWS Lambda: one pricing model — requests + duration. That's it.

Azure Functions: three hosting plans:

| Plan | AWS Analogue | Use When |
|---|---|---|
| **Consumption** | Lambda default | Sporadic events, pay-per-execution |
| **Premium** | Lambda Provisioned Concurrency | Warm instances, VNet integration, no cold starts |
| **App Service Plan** | (no direct AWS parallel) | Function runs on your existing Web App VMs — you already pay for them |

For most workloads → Consumption. Latency-sensitive → Premium. Already using App Service → App Service plan.

**2. Bindings — Declarative Integration**

Lambda uses SDK calls in code to interact with S3, DynamoDB, etc.

Functions has **input and output bindings** — declare in config that "this function's input is a Blob at this path, output is a Cosmos document." The runtime handles I/O plumbing. Less code, more declarative.

Example — a function triggered by blob upload, writing to Cosmos:
```json
{
  "bindings": [
    {
      "type": "blobTrigger",
      "direction": "in",
      "name": "myBlob",
      "path": "uploads/{name}"
    },
    {
      "type": "cosmosDB",
      "direction": "out",
      "name": "outDoc",
      "databaseName": "AppDB",
      "collectionName": "processed"
    }
  ]
}
```
Your function code just receives `myBlob` as a parameter and returns a value assigned to `outDoc`. Runtime handles reading blob and writing to Cosmos.

**3. Durable Functions**

Serverless stateful orchestration — chained functions, fan-out/fan-in, human interaction, timers. AWS equivalent is Step Functions, but Durable Functions are code-first (write orchestrator in C#/Python/JS) vs Step Functions' JSON state machines. Different flavor of the same idea.

**4. Function App Wraps Multiple Functions**

Lambda: one function per resource.
Functions: a "Function App" contains multiple related functions sharing runtime, config, and plan. Feels more like a small microservice with several routes/triggers.

## Real Configuration

```bash
# Create a storage account (Functions needs one for state)
az storage account create \
  --name stfuncdevops01 \
  --resource-group rg-func-lab \
  --location eastus \
  --sku Standard_LRS

# Create a Function App on Consumption plan (Python 3.11)
az functionapp create \
  --resource-group rg-func-lab \
  --consumption-plan-location eastus \
  --runtime python \
  --runtime-version 3.11 \
  --functions-version 4 \
  --name func-devops-aakash \
  --storage-account stfuncdevops01 \
  --os-type Linux

# Deploy code (from local project directory)
func azure functionapp publish func-devops-aakash

# Enable Managed Identity for the function (best practice)
az functionapp identity assign \
  --name func-devops-aakash \
  --resource-group rg-func-lab

# Grant it access to a Cosmos DB
az cosmosdb sql role assignment create \
  --account-name cosmos-prod-aakash \
  --resource-group rg-prod-db-eastus \
  --role-definition-id 00000000-0000-0000-0000-000000000002 \
  --principal-id <function-managed-identity-principal-id> \
  --scope /
```

## Production Use Case

**Image processing pipeline (same as classic Lambda example):**

```
User uploads image → Blob Storage (uploads container)
    ↓ (blob trigger)
Function: process-image
    ├── Reads blob, resizes to thumbnail + medium + large
    ├── Writes 3 blobs to processed container
    ├── Writes metadata to Cosmos DB via output binding
    └── Sends notification via Service Bus output binding
    ↓
Function: send-email
    ├── Triggered by Service Bus message
    ├── Sends email via SendGrid connector
    └── Logs to Application Insights
```

Both functions live in one Function App on Consumption plan. Pay only for actual invocations. Managed Identity for Cosmos access, no keys.

## Console Path

- Portal → **Function App** (search)
- Function App detail:
  - **Functions** blade → list of functions in this app
  - **Configuration** → app settings (env vars), connection strings
  - **Identity** → enable/manage Managed Identity
  - **Deployment Center** → CI/CD from GitHub/Azure DevOps
  - **Networking** → VNet integration, private endpoints (Premium+ plans only)
  - **Log stream** → live log tail
  - Individual function → **Code + Test** for in-portal editing

## Memory Hint

- **Function App** = container for one or more Functions (unlike Lambda's per-function model)
- **Consumption** = pay-per-execution (default choice)
- **Premium** = pre-warmed + VNet (no cold starts)
- **Bindings** = declarative I/O (less code than SDK calls)
- **Durable Functions** = Step Functions equivalent, code-first

## Interview Q&A — Azure Functions

**Q: A team is moving from AWS Lambda to Azure Functions. What's the mental shift?**

> "Three main shifts.
>
> First, hosting plans matter. Lambda is one thing — Azure has Consumption (pay-per-execution, default), Premium (warm, VNet integration, no cold starts), and App Service (runs on existing web app compute). Consumption is closest to Lambda default; picking the right plan for a workload is a decision Lambda doesn't force.
>
> Second, bindings. Lambda code explicitly calls AWS SDK. Functions can declare bindings — 'my input is this blob, my output goes to Cosmos DB' — and the runtime handles the I/O. It's declarative vs imperative. Some teams love it, some prefer explicit SDK calls. Once you internalize bindings, they reduce a lot of boilerplate.
>
> Third, Function Apps group functions. Lambda is one function per resource. In Azure, a Function App is a deployable unit containing multiple related functions sharing runtime, settings, and hosting plan. Feels more like a microservice than a single-purpose function.
>
> Cold starts: similar issue in both. Python and Node cold-start fast (100-300ms). C# and Java slower (1-2s). Mitigation on Azure is the Premium plan, on AWS it's Provisioned Concurrency — same idea, different plumbing.
>
> Cost model: Consumption charges per execution + GB-seconds — nearly identical to Lambda. Premium is a per-vCPU-hour flat rate — different math, need to compare based on actual invocation volume."

**Q: When would you use Durable Functions vs a chain of regular Functions?**

> "Durable Functions when the workflow has state, orchestration, or long-running steps that outlast a single function invocation.
>
> Regular chained Functions are stateless — one triggers the next via a queue or event. Fine for simple 'A → B → C' pipelines where each step is independent.
>
> Durable Functions shines with patterns like:
>
> Function chaining with state — execute A, wait for result, use it to call B, then C, all with in-memory state that persists.
>
> Fan-out/fan-in — kick off 100 parallel tasks, wait for all to complete, aggregate results. Regular Functions would need external state store to track completion.
>
> Human interaction — start a workflow, send approval email, pause until human clicks approve, then continue. Regular Functions can't 'wait' for hours or days.
>
> Long-running orchestrations — workflows that need to run for hours, days, or weeks with state preserved.
>
> Timers — schedule a step to run 3 days later without external scheduler.
>
> AWS equivalent is Step Functions with its JSON-based state machine language. Durable Functions writes the orchestrator in code (C#, Python, JS) — feels more natural to developers. Different style, same category.
>
> Interview signal: candidates who default to Durable Functions for everything are over-engineering. Reach for it when the workflow genuinely needs orchestration or state. Simple chain? Regular Functions with queue triggers is fine."

---
---

# 11. ARM & Bicep

## Concept

Azure Resource Manager (ARM) is the deployment and management layer for all Azure resources. ARM templates are JSON files declaring desired infrastructure. Bicep is a domain-specific language that transpiles to ARM — same power, way better ergonomics.

## AWS Equivalent

**ARM ≈ AWS CloudFormation (raw JSON/YAML) but ARM is chattier**
**Bicep ≈ AWS CDK or CloudFormation with cleaner syntax**

## Key Differences from AWS

**1. ARM Templates Are Verbose**
Even a simple VM in ARM JSON is 100+ lines with heavy nesting. CloudFormation YAML is more readable. Bicep fixes this.

**2. Bicep Is First-Party**
Microsoft ships Bicep as the recommended IaC language for Azure. Not a community tool — official product with tooling, extensions, and documentation. AWS's equivalent is CDK (multi-language SDKs generating CloudFormation).

**3. Terraform Is the Practical Cross-Cloud Choice**
For anyone working across clouds or with existing Terraform investment, use Terraform. Both Bicep and Terraform are legitimate on Azure. Bicep for Azure-only shops, Terraform for multi-cloud or existing HCL codebases.

## Bicep vs ARM JSON — Same Resource

**ARM JSON (the old way):**
```json
{
  "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "resources": [
    {
      "type": "Microsoft.Storage/storageAccounts",
      "apiVersion": "2023-01-01",
      "name": "stdevopsaakash01",
      "location": "eastus",
      "sku": { "name": "Standard_LRS" },
      "kind": "StorageV2",
      "properties": {
        "accessTier": "Hot"
      }
    }
  ]
}
```

**Bicep (the new way, same result):**
```bicep
resource storage 'Microsoft.Storage/storageAccounts@2023-01-01' = {
  name: 'stdevopsaakash01'
  location: 'eastus'
  sku: { name: 'Standard_LRS' }
  kind: 'StorageV2'
  properties: {
    accessTier: 'Hot'
  }
}
```

Bicep is dramatically cleaner. Type-checked. IntelliSense in VS Code. Compiles to ARM JSON at deploy time — Azure only executes ARM under the hood, Bicep is a syntactic layer.

## Real Configuration

```bash
# Install Bicep CLI (bundled with Azure CLI)
az bicep install
az bicep version

# Author main.bicep, then deploy
az deployment group create \
  --resource-group rg-iac-lab \
  --template-file main.bicep \
  --parameters environment=dev location=eastus

# Preview changes (like terraform plan)
az deployment group what-if \
  --resource-group rg-iac-lab \
  --template-file main.bicep \
  --parameters environment=dev location=eastus

# Convert existing ARM JSON to Bicep
az bicep decompile --file template.json
# Produces template.bicep — great for learning from Microsoft's quickstart samples
```

## Bicep vs Terraform — Real Talk

| | Bicep | Terraform |
|---|---|---|
| Scope | Azure only | Multi-cloud |
| State | Managed by Azure (ARM) | State file (you manage) |
| Language | DSL, Azure-native | HCL, universal |
| Learning curve | Small if you know ARM | Steeper but transferable |
| Ecosystem | Growing, Azure-focused | Massive, mature |
| Best for | Azure-only shops | Multi-cloud, larger teams |

**My recommendation for you specifically:** learn both, prefer Terraform for portfolio. Terraform skills transfer to AWS, GCP, and third-party providers (GitHub, Datadog, Cloudflare). Bicep is Azure-locked. But knowing Bicep matters for Azure interviews because you might inherit a Bicep codebase.

## Production Use Case

**Multi-environment infrastructure with Bicep modules:**

```
infra/
├── main.bicep                    # entry point
├── modules/
│   ├── networking.bicep          # VNet, subnets, NSGs
│   ├── compute.bicep             # VMSS, App Gateway
│   ├── data.bicep                # SQL, Cosmos
│   └── monitoring.bicep          # Log Analytics, Application Insights
└── environments/
    ├── dev.bicepparam            # dev-specific parameters
    ├── staging.bicepparam
    └── prod.bicepparam
```

Deploy dev: `az deployment group create --parameters environments/dev.bicepparam`
Deploy prod: `az deployment group create --parameters environments/prod.bicepparam`

Same templates, environment-specific parameters. Analogous to Terraform's tfvars.

## Console Path

- Portal → **Deployments** (under resource group) — history of ARM/Bicep deployments
- **Templates** → export any existing resource as ARM JSON (great for reverse-engineering)
- **Deploy a custom template** (search) → paste ARM/Bicep, deploy from portal

## Memory Hint

- **ARM = raw JSON, verbose, don't write it by hand**
- **Bicep = clean DSL, transpiles to ARM, first-party**
- **Terraform = multi-cloud alternative, use if you're not Azure-only**
- Bicep decompile from ARM JSON is a killer learning technique

## Interview Q&A — ARM & Bicep

**Q: When would you choose Bicep over Terraform for Azure infrastructure?**

> "Depends on team context and multi-cloud aspirations.
>
> Bicep for: Azure-only shops with no plans to go multi-cloud. Teams who value tighter integration with Azure — new preview services get Bicep support immediately, Terraform sometimes lags by weeks. Teams that want state management to be Azure's problem, not theirs.
>
> Terraform for: multi-cloud teams. Teams with existing HCL codebases. Teams valuing a larger ecosystem and community modules. Teams that want cross-provider composition — provisioning Azure resources AND GitHub secrets AND Datadog monitors from one apply.
>
> My default recommendation is Terraform for most modern teams, because 'Azure-only forever' is rare in enterprise reality, and Terraform skills carry across clouds. But if a team is genuinely Azure-committed and values the first-party experience, Bicep is a fine choice — dramatically better than raw ARM.
>
> One anti-pattern: raw ARM JSON. Nobody should be hand-writing that in 2026. If you inherit an ARM codebase, plan a Bicep migration — `az bicep decompile` handles most of the conversion automatically.
>
> Interview signal: candidates who dogmatically pick one without acknowledging context are junior. Senior answer is 'here's how I'd decide.'"

---
---

# 12. Azure DNS

## Concept

Azure DNS is a managed DNS hosting service. Hosts DNS zones and records with global anycast infrastructure.

## AWS Equivalent

**Azure DNS = AWS Route 53** (for DNS hosting specifically)

**Traffic Manager = Route 53 routing policies**

The equivalence is clean here. Same fundamental service. Some feature gaps and additions on each side.

## Key Differences from AWS

**1. Public vs Private Zones — Same as Route 53**

- **Public DNS zone** — internet-resolvable, for your domain
- **Private DNS zone** — only resolvable inside linked VNets, for internal service discovery

**2. Alias Records**

Azure DNS supports alias records that automatically track the IP of Azure resources — Traffic Manager profiles, Front Door endpoints, public IPs, etc. If the underlying resource IP changes, DNS updates automatically.

Similar to Route 53 alias records. Same concept, similar limitations (only for supported Azure resource types).

**3. No Domain Registration in Azure DNS Itself**

Route 53 handles both registration and hosting. Azure DNS only does hosting — for registration, you use **App Service Domains** (part of App Service) or buy elsewhere and point NS records to Azure.

Minor annoyance. Most enterprises register domains through a corporate registrar anyway (Cloudflare, GoDaddy, Google Domains, etc.) and point them at Azure DNS for hosting.

**4. Traffic Manager Handles Routing Policies**

Route 53 combines hosting + routing (weighted, latency, failover, geo). Azure splits: **Azure DNS** for basic hosting, **Traffic Manager** for policy-based routing.

## Real Configuration

```bash
# Create a DNS zone
az network dns zone create \
  --resource-group rg-dns-prod \
  --name aakashrao.dev

# Get the nameservers (point your registrar to these)
az network dns zone show \
  --resource-group rg-dns-prod \
  --name aakashrao.dev \
  --query nameServers

# Create an A record
az network dns record-set a add-record \
  --resource-group rg-dns-prod \
  --zone-name aakashrao.dev \
  --record-set-name www \
  --ipv4-address 20.62.146.10

# Create a CNAME
az network dns record-set cname set-record \
  --resource-group rg-dns-prod \
  --zone-name aakashrao.dev \
  --record-set-name blog \
  --cname www.aakashrao.dev

# Create an alias record pointing to a Front Door endpoint
az network dns record-set a create \
  --resource-group rg-dns-prod \
  --zone-name aakashrao.dev \
  --name @ \
  --target-resource /subscriptions/.../frontdoor-name

# Private DNS zone for internal service discovery
az network private-dns zone create \
  --resource-group rg-network-prod \
  --name internal.aakashrao.dev

# Link private zone to VNet
az network private-dns link vnet create \
  --resource-group rg-network-prod \
  --zone-name internal.aakashrao.dev \
  --name link-vnet-prod \
  --virtual-network vnet-prod-eastus \
  --registration-enabled false
```

## Production Use Case

**Public + private DNS split-brain for internal microservices:**

```
Public zone: aakashrao.dev (Azure DNS)
├── A record: www → Front Door IP
├── CNAME: blog → hosted Ghost blog
└── MX records: Google Workspace mail servers

Private zone: internal.aakashrao.dev (linked to prod VNet)
├── A record: db.internal → 10.0.20.5 (private SQL endpoint)
├── A record: cache.internal → 10.0.20.10 (private Redis endpoint)
└── A record: search.internal → 10.0.20.15 (private Elasticsearch)

Apps in the VNet resolve db.internal.aakashrao.dev to the private IP.
Public users can't resolve *.internal.aakashrao.dev at all.
```

Same split-brain DNS pattern as AWS Route 53 with private hosted zones — different implementation, same outcome.

## Memory Hint

- **Azure DNS** = Route 53 hosting (without domain registration built-in)
- **Traffic Manager** = Route 53 routing policies (separate service)
- **Alias record** = auto-tracks Azure resource IPs
- **Private DNS zone** + VNet link = internal service discovery

## Interview Q&A — Azure DNS

**Q: How would you build the DNS equivalent of a Route 53 latency-based routing setup on Azure?**

> "Two-service composition: Azure DNS for zone hosting + Traffic Manager for routing.
>
> Azure DNS hosts your zone — `aakashrao.dev` — with static records for things that don't need routing (MX, TXT, etc.).
>
> For the smart routing, create a Traffic Manager profile with the Performance routing method. Add endpoints for each region — say, `agw-eastus.eastus.cloudapp.azure.com`, `agw-westeurope.westeurope.cloudapp.azure.com`, `agw-southeastasia.southeastasia.cloudapp.azure.com`. Configure health probes.
>
> In Azure DNS, create a CNAME `www.aakashrao.dev → myapp.trafficmanager.net`. Or better, use an Alias record if the apex needs routing.
>
> User in India resolves `www.aakashrao.dev` → CNAME to `myapp.trafficmanager.net` → Traffic Manager returns the endpoint with lowest latency from the user's DNS resolver → user connects to closest App Gateway.
>
> Failover behavior: Traffic Manager health probes detect a failed endpoint and stops returning it for future queries. TTLs matter — set them low (30-60s) so failover propagates fast.
>
> This is more services stitched together than Route 53's single-service approach with routing policies built in. But it works well and gives you the flexibility to swap DNS provider without touching routing logic. Feature parity, more wiring."

---
---

# 13. Azure CDN

## Concept

Content delivery — cache content at edge locations close to users.

## AWS Equivalent

**Azure has TWO products in this space, which is confusing:**

- **Azure CDN** — traditional CDN, older product. Multiple vendor backends (Microsoft, Akamai, Verizon).
- **Azure Front Door** — modern global L7 with CDN + WAF + Load Balancing bundled.

**AWS CloudFront ≈ Azure Front Door** (for new work)
**AWS CloudFront ≈ Azure CDN** (legacy positioning)

**Recommendation:** For anything new, use Front Door. Azure CDN is being deprecated in phases — the Akamai and Verizon tiers are already retired or retiring, Microsoft tier remains but Front Door is the strategic direction.

## Front Door Recap (covered earlier in Section 6)

Since we covered Front Door in the load-balancing section, this is a shorter recap focused on CDN aspects.

**Front Door as a CDN:**
- 100+ edge POPs globally
- Cache static and dynamic content with configurable TTLs
- Compression (gzip, brotli)
- Query string caching modes
- Custom cache rules per path pattern
- Automatic HTTPS with managed certificates
- WAF at edge for security
- Custom domains supported

## Real Configuration

```bash
# Create Front Door profile (Standard SKU is fine for most cases; Premium adds more WAF + private link)
az afd profile create \
  --profile-name fd-prod-web \
  --resource-group rg-prod-web-eastus \
  --sku Standard_AzureFrontDoor

# Create an endpoint (the hostname users hit)
az afd endpoint create \
  --resource-group rg-prod-web-eastus \
  --profile-name fd-prod-web \
  --endpoint-name myapp \
  --enabled-state Enabled

# Endpoint URL: myapp-xxxxxx.z01.azurefd.net

# Create an origin group and add origins (your App Gateways or Blob endpoints)
az afd origin-group create \
  --resource-group rg-prod-web-eastus \
  --profile-name fd-prod-web \
  --origin-group-name web-origins \
  --probe-request-type GET \
  --probe-protocol Https \
  --probe-interval-in-seconds 30 \
  --probe-path /health \
  --sample-size 4 \
  --successful-samples-required 3 \
  --additional-latency-in-milliseconds 50

# Create a route matching all paths
az afd route create \
  --resource-group rg-prod-web-eastus \
  --profile-name fd-prod-web \
  --endpoint-name myapp \
  --route-name default \
  --origin-group web-origins \
  --supported-protocols Http Https \
  --https-redirect Enabled \
  --forwarding-protocol MatchRequest \
  --link-to-default-domain Enabled \
  --patterns-to-match /*
```

## Production Use Case

**Global website: static + dynamic split via Front Door:**

```
Front Door Profile: fd-prod-web (Premium SKU)
├── Endpoint: myapp.azurefd.net (also CNAMEd to www.aakashrao.dev)
├── Origins:
│   ├── Origin group: static-origins
│   │   └── Origin: stprodstatic.z13.web.core.windows.net (Blob $web endpoint)
│   └── Origin group: dynamic-origins
│       ├── Origin: agw-eastus.eastus.cloudapp.azure.com
│       └── Origin: agw-westeurope.westeurope.cloudapp.azure.com
├── Routes:
│   ├── /static/*, /images/*, /css/*, /js/* → static-origins, cache 7 days
│   ├── /api/* → dynamic-origins, no cache
│   └── /* → dynamic-origins, cache 5 min if 200 response
└── WAF policy: OWASP + custom rules

Managed certificate for www.aakashrao.dev.
Rules engine: strip cookies from static requests to improve cache hit ratio.
```

## Memory Hint

- **Front Door** = CloudFront + WAF + Global Accelerator, one service
- **Azure CDN** = older, being deprecated, don't start new projects on it
- **Endpoints** live in profiles; profiles have routes; routes point to origin groups; origin groups contain origins. Deep nesting compared to CloudFront distributions.

## Interview Q&A — CDN

**Q: You inherit a workload using Azure CDN Standard from Microsoft. Would you migrate to Front Door?**

> "Yes, planned migration. Azure CDN is on a phased sunset path — Verizon and some tiers already retired, Microsoft tier still supported but Front Door is the strategic direction. Front Door offers a superset of capabilities: global anycast, WAF v2, rules engine, private link support in Premium.
>
> Migration approach:
>
> First, provision Front Door in parallel. Point origin(s) to the same backend as the CDN.
>
> Second, test on a non-production subdomain to validate cache behavior, TTLs, and any custom rules. CDN-to-Front-Door cache behavior differs subtly — verify your caching semantics still work.
>
> Third, DNS cutover with reduced TTL. Update CNAME from `cdn-endpoint.azureedge.net` to `frontdoor-endpoint.azurefd.net`. Monitor for anomalies.
>
> Fourth, keep the old CDN endpoint alive for a week for rollback safety.
>
> Watch out for: WAF policies aren't portable directly. If you had a WAF on the CDN via a separate service, you'll rebuild it in Front Door's native WAF. Custom cache rules likewise need translation.
>
> The upside: better performance from Front Door's Microsoft-owned edge network, integrated WAF, and one less service to manage."

---
---

# 14. AKS — Azure Kubernetes Service

**This is the deepest section — Kubernetes is your strongest area, and AKS mastery is a huge multiplier for interviews at MNC roles.**

## Concept

Azure Kubernetes Service is managed Kubernetes. Azure runs the control plane, you manage node pools (or use Virtual Nodes / Fargate-like for serverless).

## AWS Equivalent

**AKS = AWS EKS**

Same category, similar responsibility split (managed control plane, self-managed or managed worker nodes). Different pricing model, different integrations, different defaults.

## Key Differences from AWS

**1. Control Plane Is FREE**

EKS: $73/month per cluster for the control plane. AKS: **free** for standard tier control plane (paid tier for uptime SLA guarantee available but not required).

**Practical impact:** Multi-cluster architecture is cheaper on Azure. Teams that would consolidate on one big EKS cluster to save $73/month can freely run per-environment AKS clusters — one for dev, one for staging, one for prod — with no control-plane cost overhead.

**2. Node Pool Structure**

Both AKS and EKS use "node pools" (AKS terminology) / "managed node groups" (EKS). Very similar. Key AKS features:
- **System node pool** — mandatory, runs system pods (CoreDNS, kube-proxy, metrics-server). Kept separate from user workloads.
- **User node pools** — for your app workloads. Multiple pools with different VM sizes, taints, labels.
- **Spot node pools** — mix spot into an existing cluster easily
- **Virtual Nodes** — serverless Fargate-equivalent via Azure Container Instances

**3. Entra ID Integration Is Native**

`kubectl` authentication via Entra ID with RBAC mapped to Kubernetes RBAC. Users sign in with their corporate identity, get short-lived tokens, cluster access governed by Entra ID group membership.

EKS supports similar via AWS IAM + `aws-auth` ConfigMap, but AKS integration is smoother because Entra ID is a proper identity provider (whereas AWS IAM is more resource-focused).

**4. Azure CNI vs Kubenet**

Two networking modes:
- **Kubenet** — pods get IPs from internal network, translated at node boundary. Fewer IPs consumed, simpler. Default in older AKS versions.
- **Azure CNI** — pods get real VNet IPs. Direct routing, better integration with other Azure services, uses more IPs. Modern default.

For interviews: know both exist, know Azure CNI is the modern choice, understand the IP consumption trade-off (Azure CNI can burn through subnet IPs fast with many pods).

**5. AGIC — Application Gateway Ingress Controller**

Native integration between AKS and Application Gateway. Instead of running an nginx ingress controller inside the cluster, AKS spawns/manages an Application Gateway that acts as the cluster ingress.

Trade-off: more Azure-native, deeper integration with WAF, no ingress controller pods to maintain. But more Azure-locked and Application Gateway has some Kubernetes-specific limitations to watch.

Alternative: Nginx Ingress Controller or Traefik running inside AKS — same as any Kubernetes, more portable.

## Real Configuration

```bash
# Create resource group
az group create --name rg-aks-prod-eastus --location eastus

# Create AKS cluster with:
# - System node pool: 3 nodes across 3 AZs (Standard_D2s_v5)
# - Azure CNI networking
# - Entra ID integration with Azure RBAC for Kubernetes
# - Managed identity for cluster
# - Cluster autoscaler enabled
az aks create \
  --resource-group rg-aks-prod-eastus \
  --name aks-prod-eastus \
  --location eastus \
  --kubernetes-version 1.29 \
  --node-count 3 \
  --node-vm-size Standard_D2s_v5 \
  --zones 1 2 3 \
  --network-plugin azure \
  --enable-managed-identity \
  --enable-aad \
  --enable-azure-rbac \
  --enable-cluster-autoscaler \
  --min-count 3 \
  --max-count 10 \
  --generate-ssh-keys \
  --tier standard

# Get credentials (updates ~/.kube/config)
az aks get-credentials \
  --resource-group rg-aks-prod-eastus \
  --name aks-prod-eastus \
  --admin  # or --admin for legacy, prefer without for Entra ID auth

# Verify
kubectl get nodes -o wide

# Add a user node pool (for app workloads, tainted so system pods don't schedule here)
az aks nodepool add \
  --resource-group rg-aks-prod-eastus \
  --cluster-name aks-prod-eastus \
  --name userpool \
  --node-count 3 \
  --node-vm-size Standard_D4s_v5 \
  --zones 1 2 3 \
  --enable-cluster-autoscaler \
  --min-count 3 \
  --max-count 20 \
  --labels workload=general \
  --node-taints workload=general:NoSchedule

# Add a spot node pool for batch workloads (up to 90% discount)
az aks nodepool add \
  --resource-group rg-aks-prod-eastus \
  --cluster-name aks-prod-eastus \
  --name spotpool \
  --priority Spot \
  --eviction-policy Delete \
  --spot-max-price -1 \
  --node-count 2 \
  --enable-cluster-autoscaler \
  --min-count 0 \
  --max-count 30 \
  --labels workload=batch \
  --node-taints kubernetes.azure.com/scalesetpriority=spot:NoSchedule

# Enable AGIC (App Gateway Ingress Controller)
az aks enable-addons \
  --resource-group rg-aks-prod-eastus \
  --name aks-prod-eastus \
  --addons ingress-appgw \
  --appgw-name agw-aks-prod \
  --appgw-subnet-cidr 10.2.0.0/16
```

## Workload Identity — Managed Identity for Pods

The Azure answer to AWS IRSA (IAM Roles for Service Accounts) is **Workload Identity**. Pods get Entra ID identities via a Kubernetes ServiceAccount → federated identity credential.

Flow:
1. Enable Workload Identity on the cluster (or during create with `--enable-workload-identity`)
2. Create a Managed Identity in Azure
3. Grant it Azure permissions (RBAC role assignments)
4. Create a Kubernetes ServiceAccount annotated with the identity's client ID
5. Configure a federated credential linking the identity to the ServiceAccount
6. Pods using that ServiceAccount get Azure tokens automatically — no secrets

```yaml
# Kubernetes ServiceAccount
apiVersion: v1
kind: ServiceAccount
metadata:
  name: myapp-sa
  namespace: default
  annotations:
    azure.workload.identity/client-id: "xxx-xxx-xxx-xxx"

---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp
spec:
  template:
    metadata:
      labels:
        azure.workload.identity/use: "true"
    spec:
      serviceAccountName: myapp-sa
      containers:
      - name: myapp
        image: myacr.azurecr.io/myapp:v1
```

Pod code uses Azure SDK, SDK reads the projected token, calls Azure APIs. No client_secret, no managed identity endpoint, cleanest pattern available.

## Production Use Case

**Multi-tenant AKS with proper zone and workload isolation:**

```
AKS Cluster: aks-prod-eastus (Kubernetes 1.29, Standard tier, uptime SLA)
├── Networking: Azure CNI, dedicated VNet subnet (10.2.0.0/16)
├── Entra ID auth + Azure RBAC for Kubernetes RBAC
│
├── Node pools:
│   ├── systempool (3× Standard_D2s_v5, zones 1/2/3, cluster autoscale 3-5)
│   │   └── Only kube-system workloads
│   ├── generalpool (3× Standard_D4s_v5, zones 1/2/3, autoscale 3-20)
│   │   └── Most app workloads
│   ├── memorypool (2× Standard_E4s_v5, zones 1/2/3, autoscale 2-10)
│   │   └── Redis, ElasticSearch, other memory-hungry
│   └── spotpool (0-30× D4s_v5 Spot, autoscale 0-30)
│       └── Batch jobs, cost-tolerant workloads
│
├── Addons:
│   ├── AGIC → Application Gateway with WAF
│   ├── Azure Monitor for containers (Container Insights)
│   ├── Azure Key Vault CSI provider (mount secrets as volumes)
│   ├── Workload Identity for pod-level Entra IDs
│   └── Cluster Autoscaler + Vertical Pod Autoscaler
│
├── Ingress: AGIC → Application Gateway (WAF policy attached)
├── Cert-manager for automated TLS
├── ArgoCD for GitOps deployments
└── Prometheus + Grafana via kube-prometheus-stack Helm chart
```

## Console Path

- Portal → **Kubernetes services** (search)
- Cluster detail:
  - **Overview** — status, node count, K8s version, endpoint
  - **Node pools** — manage pools, scaling, upgrades
  - **Configuration** → **Cluster configuration** — cluster autoscaler, workload identity, etc.
  - **Networking** — CNI info, ingress, service mesh
  - **Security** → **Defender for Cloud** for runtime security
  - **Monitoring** → Container Insights, Live metrics, Logs
  - **Kubernetes resources** → Namespaces, Workloads, Services (in-portal view, similar to K8s dashboard)

## Memory Hint

- **AKS control plane = FREE** (vs EKS $73/month) — enables multi-cluster designs
- **Node pools** → System (required, kube-system) + User (your apps) + optional Spot
- **Azure CNI** = real VNet IPs for pods (modern default)
- **Entra ID + Azure RBAC for Kubernetes** = corporate identity → cluster access
- **Workload Identity** = IRSA equivalent, pod-level Entra IDs
- **AGIC** = Application Gateway as native ingress, avoids nginx-controller pods

## Interview Q&A — AKS

**Q: You've worked with EKS. Walk me through migrating to AKS — what changes, what surprises?**

> "The Kubernetes layer is identical — kubectl, YAMLs, Helm charts, operators all work the same. That's the whole point of Kubernetes as a standard. What changes is everything around it: identity, networking, ingress, secrets, monitoring.
>
> Identity: EKS uses IAM + IRSA (IAM Roles for Service Accounts) for pod-level auth. AKS uses Entra ID + Workload Identity. Same pattern, different plumbing. You rewrite your IRSA setup as Workload Identity — annotate ServiceAccounts with Entra ID client IDs, configure federated credentials.
>
> Networking: EKS defaults to AWS VPC CNI, giving pods real VPC IPs. AKS defaults to Azure CNI, giving pods real VNet IPs. Similar model — plan subnet IP capacity accordingly. Kubenet is the older simpler alternative in AKS; nobody uses it for production anymore.
>
> Ingress: EKS commonly uses AWS Load Balancer Controller creating ALBs. AKS has AGIC — Application Gateway Ingress Controller — same idea, App Gateway is the ingress. Or run standard nginx-ingress if you want portability.
>
> Secrets: EKS with External Secrets Operator pulling from Secrets Manager. AKS has native CSI driver for Key Vault — annotate a SecretProviderClass, secrets get mounted as volumes. Simpler than the ESO pattern.
>
> Monitoring: EKS integrates with CloudWatch Container Insights. AKS integrates with Azure Monitor Container Insights. Same category, different UIs.
>
> Cost surprise: AKS control plane is free (Standard tier). EKS is $73/month per cluster. This changes the calculus for multi-cluster architectures — teams that consolidated on EKS for cost can spread out on AKS.
>
> Practical migration path: rebuild the platform layer (identity, ingress, monitoring, secrets) natively for Azure. Application manifests port over largely unchanged. Budget 2-4 weeks for platform, then app-by-app cutover."

**Q: Design a cost-optimized AKS cluster for a workload with steady baseline and unpredictable spikes.**

> "Three-tier node pool strategy plus autoscaling.
>
> Tier one: system pool. Three Standard_D2s_v5 nodes across three AZs. Only kube-system pods here. Cluster autoscaler between 3 and 5 for headroom. Small and always-on.
>
> Tier two: general pool. Standard_D4s_v5 for baseline app capacity. Autoscale 3 to 15 based on demand. This carries the steady traffic. Consider Azure Reserved Instances or Savings Plans for these — 1-year commit gets you 40-50% off, worth it because these nodes run 24/7.
>
> Tier three: spot pool. Standard_D4s_v5 spot instances, autoscale from 0 to 30. This absorbs spikes cheaply — up to 90% discount. Tainted so only workloads that tolerate eviction schedule here.
>
> Workload placement:
> - Critical low-latency services → general pool (nodeSelector or preferred affinity)
> - Batch processing, background workers, ML training → spot pool with tolerations
> - System services → system pool (auto-scheduled by taint)
>
> HPA on each deployment based on CPU or custom metrics. VPA for right-sizing pod requests over time.
>
> Cluster autoscaler runs cluster-wide, scales pools independently. When spikes hit, spot pool grows fast (cheap capacity). If spot evictions happen, HPA can push some pods back to general pool temporarily.
>
> Cost model: baseline general pool is Reserved (steady discount), spot pool for elastic capacity (deep discount), system pool is fixed overhead. Combined, you might see 50-60% savings vs running all on-demand.
>
> Monitoring: Azure Cost Analysis tagged per node pool. Track spot eviction rate — if it's too disruptive, tune eviction policy or diversify VM sizes in the spot pool."

**Q: A pod in AKS can't reach an Azure SQL Database. Walk me through debugging.**

> "Systematic, network-outward.
>
> First layer, pod itself: `kubectl exec -it <pod> -- /bin/sh` then try DNS resolution — `nslookup sql-prod-aakash.database.windows.net`. If DNS fails, CoreDNS or Azure DNS integration issue. If DNS returns the SQL server public IP, that's fine but we probably want private endpoint.
>
> Second layer, network connectivity: from the pod, `nc -zv <sql-server-fqdn> 1433` or curl equivalent. Times out? Firewall or NSG blocking. Connection refused? Server not listening. No response at all? Check both directions of NSG rules on the subnet where nodes live.
>
> Third layer, Azure SQL firewall: SQL Server has its own firewall independent of NSGs. Check if the pod's egress IP is allowed. If AKS uses default outbound, all pods share the cluster's outbound public IP — allowlist that. Better pattern: SQL private endpoint in the AKS subnet, DNS resolves to the private IP, firewall becomes moot.
>
> Fourth layer, auth: pod tries to connect using what credentials? If it's a connection string with SQL auth, verify the username/password against the SQL server directly with sqlcmd. If it's Managed Identity via Workload Identity, verify the service principal exists in the SQL server as an Entra ID user with correct permissions. `SELECT * FROM sys.database_principals` in the target database will confirm.
>
> Fifth layer, application: is the driver / connection string correct? Sometimes it's not the network at all — it's the app trying wrong port, wrong TLS setting, wrong AAD authentication mode.
>
> One trap specific to AKS: if you're using Azure CNI with restricted outbound (e.g., forced tunneling through Azure Firewall), the SQL FQDN might need to be explicitly allowed. Azure Firewall logs will show whether the connection attempted and what happened.
>
> The mental model is the same as any Kubernetes + external DB debug — but Azure adds specific layers (SQL firewall, Entra ID auth, private endpoints) worth knowing individually. Interview signal: walking through this systematically shows you know Azure networking, not just K8s."

---
---

# 15. Networking — VNets, Subnets, NSGs

## Concept

Azure Virtual Networks (VNets) are isolated network environments — your private space in Azure where resources communicate.

## AWS Equivalent

**VNet = AWS VPC**

Same category. Regional. Contains subnets. Rules for traffic control. Similar mental model.

## Key Differences from AWS

**1. VNet is Regional (Same as VPC)**

Unlike GCP where VPCs are global, Azure VNets are regional. Same as AWS VPCs. If you know VPCs, you know this half of VNets.

**2. Subnet Semantics Are Simpler**

AWS distinguishes public vs private subnets by whether they have a route to the Internet Gateway. Azure subnets don't have this hard distinction — a subnet has internet access if:
- Its resources have public IPs, OR
- It routes through NAT Gateway, OR
- Default outbound is enabled (deprecated pattern for new subnets)

More flexible, but slightly less structured than AWS's public/private subnet convention.

**3. NSGs vs Security Groups**

Network Security Group ≈ AWS Security Group + NACL merged.

- **Stateful** like AWS SGs
- Can be attached to **subnet** OR **NIC** (or both — combined)
- Both allow AND deny rules (SGs are allow-only)
- Higher-priority rules override lower-priority

Rule format: priority (100-4096), source, source port, destination, destination port, protocol, action.

Combined attachment: if NSG is on both the subnet and the NIC, traffic must be allowed by BOTH. Restrictive AND, not OR.

**4. Peering + Virtual Network Gateway**

VNet-to-VNet connectivity:
- **VNet Peering** — direct link, low latency, no gateway needed. Cross-region peering supported (extra cost). AWS equivalent: VPC Peering.
- **VNet Gateway** — for hybrid connectivity (VPN or ExpressRoute back to on-prem). AWS equivalent: Site-to-Site VPN / Direct Connect.

**5. Azure Bastion vs Bastion Host EC2**

Azure Bastion is a managed jump box service — provisioned in a VNet, provides browser-based RDP/SSH to VMs without exposing them to internet. AWS you'd typically DIY with a bastion EC2 or use Systems Manager Session Manager. Azure Bastion is cleaner and more secure.

**6. Private Endpoint = PrivateLink**

Azure Private Endpoint gives a resource (SQL, Storage, Cosmos, etc.) a private IP inside your VNet. AWS equivalent: PrivateLink / VPC endpoints. Both essentially remove the need to traverse public internet for accessing PaaS services.

## Real Configuration

```bash
# Create a VNet with an initial subnet
az network vnet create \
  --resource-group rg-prod-network-eastus \
  --name vnet-prod-eastus \
  --address-prefix 10.0.0.0/16 \
  --subnet-name snet-web \
  --subnet-prefix 10.0.1.0/24 \
  --location eastus

# Add more subnets
az network vnet subnet create \
  --resource-group rg-prod-network-eastus \
  --vnet-name vnet-prod-eastus \
  --name snet-app \
  --address-prefix 10.0.2.0/24

az network vnet subnet create \
  --resource-group rg-prod-network-eastus \
  --vnet-name vnet-prod-eastus \
  --name snet-db \
  --address-prefix 10.0.3.0/24

# Create an NSG
az network nsg create \
  --resource-group rg-prod-network-eastus \
  --name nsg-web

# Add rules
az network nsg rule create \
  --resource-group rg-prod-network-eastus \
  --nsg-name nsg-web \
  --name allow-https \
  --priority 100 \
  --source-address-prefixes '*' \
  --destination-port-ranges 443 \
  --access Allow \
  --protocol Tcp \
  --direction Inbound

az network nsg rule create \
  --resource-group rg-prod-network-eastus \
  --nsg-name nsg-web \
  --name allow-ssh-from-bastion \
  --priority 110 \
  --source-address-prefixes 10.0.100.0/24 \
  --destination-port-ranges 22 \
  --access Allow \
  --protocol Tcp \
  --direction Inbound

# Attach NSG to subnet
az network vnet subnet update \
  --resource-group rg-prod-network-eastus \
  --vnet-name vnet-prod-eastus \
  --name snet-web \
  --network-security-group nsg-web

# Peer two VNets
az network vnet peering create \
  --resource-group rg-prod-network-eastus \
  --name peer-prod-to-shared \
  --vnet-name vnet-prod-eastus \
  --remote-vnet /subscriptions/.../vnet-shared-eastus \
  --allow-vnet-access

# Create a NAT Gateway (for outbound internet from private subnets)
az network public-ip create \
  --resource-group rg-prod-network-eastus \
  --name pip-nat-eastus \
  --sku Standard

az network nat gateway create \
  --resource-group rg-prod-network-eastus \
  --name natgw-eastus \
  --public-ip-addresses pip-nat-eastus \
  --idle-timeout 10

az network vnet subnet update \
  --resource-group rg-prod-network-eastus \
  --vnet-name vnet-prod-eastus \
  --name snet-app \
  --nat-gateway natgw-eastus
```

## Production Use Case — 3-Tier VNet Design

```
VNet: vnet-prod-eastus (10.0.0.0/16, region: East US)
│
├── Subnet: snet-web (10.0.1.0/24)        ← "public" tier (App Gateway lives here)
│   ├── NSG: nsg-web
│   │   ├── Allow 443 from Internet
│   │   ├── Allow 80 from Internet (redirect to 443)
│   │   └── Deny everything else
│   └── App Gateway (agw-prod-web)
│
├── Subnet: snet-app (10.0.2.0/24)        ← private tier (VMs, containers)
│   ├── NSG: nsg-app
│   │   ├── Allow 8080 from nsg-web (App Gateway forwards here)
│   │   ├── Allow 443 outbound to Internet (via NAT Gateway)
│   │   └── Deny all other inbound
│   └── VMSS: vmss-prod-web (3 VMs, zones 1/2/3)
│
├── Subnet: snet-db (10.0.3.0/24)         ← isolated tier (databases)
│   ├── NSG: nsg-db
│   │   ├── Allow 1433 from nsg-app (only app tier can reach SQL)
│   │   └── Deny everything else
│   └── Azure SQL Private Endpoint
│
├── Subnet: snet-bastion (10.0.100.0/27)  ← Bastion subnet (fixed name: AzureBastionSubnet)
│   └── Azure Bastion host
│
├── NAT Gateway: natgw-eastus attached to snet-app (outbound only)
└── VNet Peering: to hub VNet for cross-team access

Route tables enforce that traffic between tiers routes correctly.
```

Same three-tier pattern as classic AWS VPC design (public, private-app, private-db), different naming.

## Console Path

- Portal → **Virtual networks**
- VNet detail:
  - **Address space** — CIDR blocks
  - **Subnets** — list, add, remove subnets
  - **Peerings** — VNet-to-VNet connections
  - **DNS servers** — override with custom DNS
  - **Diagram** — visual topology
- **Network security groups** (separate resource type)
- **Route tables** (separate resource type)
- **Network Watcher** — diagnostics, connection troubleshoot, packet capture

## Memory Hint

- **VNet = VPC** (regional, contains subnets)
- **NSG = SG + NACL merged** (stateful, allow AND deny rules)
- **Attach NSG to subnet OR NIC** (or both — additive AND)
- **Private Endpoint = PrivateLink** (private IPs for PaaS services)
- **Azure Bastion = managed jump box** (better than DIY bastion EC2)

## Interview Q&A — Networking

**Q: Design a production 3-tier app network on Azure. Compare choices to AWS.**

> "Reference architecture follows the same three-tier pattern as AWS with Azure-specific tooling.
>
> VNet with a /16 CIDR — plenty of room. Three subnets: web tier /24, app tier /24, db tier /24. Add a Bastion subnet with the mandatory name AzureBastionSubnet, and a NAT Gateway subnet if needed.
>
> Web tier hosts the ingress — Application Gateway with WAF. NSG allows 443 from Internet, everything else denied.
>
> App tier hosts compute — VMSS or AKS node pool. NSG allows only the app port from the App Gateway's NSG (source is the NSG itself, not an IP range). Outbound to Internet via NAT Gateway for OS updates and API calls. No direct inbound from Internet.
>
> DB tier hosts Azure SQL / Cosmos DB endpoints via Private Endpoint — no public exposure. NSG allows the DB port from the app tier NSG only.
>
> Cross-tier traffic enforced by NSG rules using NSG-as-source instead of IP ranges — more resilient because it survives IP changes.
>
> For hybrid connectivity to on-prem: VNet Gateway with ExpressRoute or Site-to-Site VPN.
>
> Comparison to AWS: the design is identical, just different service names. Application Gateway = ALB, NSG = SG (mostly), NAT Gateway = NAT Gateway, Bastion = Bastion. Azure adds Private Endpoint by default for PaaS services which is cleaner than AWS's VPC endpoints for some services.
>
> Governance: Azure Policy enforces the design at scale — 'no subnet can be created without an NSG,' 'no NSG can allow 22 from Internet,' etc. Analogous to AWS Service Control Policies + Config Rules combined."

**Q: A VM in one VNet can't reach a VM in another VNet — same subscription. What could be wrong?**

> "Ordered checklist, from most likely to least.
>
> First: is there a VNet peering between them? By default VNets are isolated. Create bidirectional peering (`--allow-vnet-access` on both sides). Peering is instant, not a route table update.
>
> Second: NSG rules on either end. Even with peering, NSGs still apply. Source VM's outbound NSG must allow the traffic. Destination VM's inbound NSG must allow it. Check both — I've seen people configure just one and wonder why it fails.
>
> Third: Azure Firewall or NVA in the path. If there's a hub-spoke topology, traffic between spokes might route through a hub firewall via User-Defined Routes. Check UDRs on the source subnet — where does traffic to the destination CIDR route? If it's forced through a firewall, check firewall rules.
>
> Fourth: overlapping IP ranges. Peering doesn't work if the two VNets have overlapping CIDR blocks. If VNet A is 10.0.0.0/16 and VNet B is 10.0.0.0/16, they can't peer meaningfully. Fix requires renumbering one of them.
>
> Fifth: DNS. If VMs use FQDNs, is DNS resolving correctly across VNets? Private DNS zones might not be linked to the source VNet. Or default Azure DNS isn't set up to resolve cross-VNet names.
>
> Diagnostic tool: Azure Network Watcher → Connection troubleshoot. Give it source and destination, it tests actual reachability, tells you where traffic drops. Amazing tool for debugging without SSHing anywhere.
>
> AWS equivalent debugging: VPC Reachability Analyzer. Same idea. Same class of tool."

---
---

# 16. Cross-Cloud Interview Master Q&A

## "You're an AWS shop being asked to also support Azure — what changes in your mental model?"

> "Three big mental shifts, then a lot of pattern matching.
>
> Shift one: the resource hierarchy. AWS is Account → Resources with tags providing organization. Azure inserts Subscription → Resource Group → Resource. The Resource Group is the load-bearing new concept — every resource belongs to exactly one RG, and RGs enable atomic cleanup, natural RBAC scoping, and clean environment separation. Once you internalize RGs, a lot of Azure operations click.
>
> Shift two: identity model. AWS IAM is one service handling both identity and resource authorization. Azure splits: Entra ID for identity, RBAC for authorization on resources. Entra ID is a proper enterprise identity provider — it also does Microsoft 365, third-party SaaS SSO. Bigger scope than AWS IAM. RBAC uses pre-built roles (Contributor, Reader, service-specific) assigned to scopes (subscription/RG/resource) that inherit downward. Simpler than writing JSON policies for most cases.
>
> Shift three: managed services are more numerous and more integrated. AWS bundles many capabilities into a few services (ALB does path routing, host routing, WAF integration). Azure splits into more services (Load Balancer for L4, Application Gateway for regional L7, Front Door for global L7, Traffic Manager for DNS routing). More services to learn, but each is more focused. Once you know which one to reach for, they compose well.
>
> Then there's the pattern matching — most services have direct AWS equivalents. EC2 = VM, S3 = Blob Storage, RDS = Azure SQL, Lambda = Functions, EKS = AKS, CloudFront = Front Door, Route 53 = Azure DNS + Traffic Manager. Once you've mapped the vocabulary, day-to-day work is very similar. The Kubernetes layer is identical.
>
> One free win: AKS control plane is free. EKS is $73/month per cluster. This changes multi-cluster architecture economics."

---

## AWS ↔ Azure Terminology Cheat Sheet

| AWS | Azure | Notes |
|---|---|---|
| Account | Subscription | Billing + quota boundary |
| Organization | Entra ID Tenant + Management Groups | Multi-account governance |
| (implicit / tags) | **Resource Group** | Mandatory logical container in Azure |
| Region | Region | Same concept |
| Availability Zone | Availability Zone | Same concept, not universal in Azure |
| IAM User | Entra ID User | |
| IAM Group | Entra ID Group | |
| IAM Role | Service Principal / Managed Identity | Managed Identity is preferred for Azure resources |
| IAM Policy | RBAC Role Assignment | Azure uses pre-built roles + scopes mostly |
| Instance Profile | System-assigned Managed Identity | |
| EC2 | Virtual Machine | |
| EBS | Managed Disk | |
| AMI | VM Image / Shared Image Gallery | |
| Auto Scaling Group | VMSS (VM Scale Set) | VMSS bundles launch template + ASG |
| Launch Template | (embedded in VMSS model) | |
| S3 | Blob Storage | Note: Storage Account wraps containers |
| S3 Bucket | Container (in Storage Account) | |
| Glacier | Archive access tier | |
| EFS | Azure Files | SMB shares |
| RDS | Azure SQL / Database for MySQL / PostgreSQL | |
| Aurora | Azure SQL Business Critical / Hyperscale | Hyperscale is closest |
| DynamoDB | Cosmos DB (NoSQL API) | Cosmos is more feature-rich |
| ElastiCache | Azure Cache for Redis | |
| Redshift | Azure Synapse Analytics | |
| Lambda | Azure Functions | |
| ECS | Azure Container Apps / Container Instances | Container Apps for microservices, ACI for one-off |
| EKS | AKS | Control plane FREE on AKS |
| Fargate | Container Apps / Container Instances / AKS Virtual Nodes | |
| API Gateway | API Management | Azure APIM is more feature-rich but more expensive |
| Route 53 | Azure DNS + Traffic Manager | Split into two services |
| ALB | Application Gateway | |
| NLB | Load Balancer (Standard) | |
| CloudFront | Front Door (or Azure CDN — sunsetting) | Front Door is more capable |
| Global Accelerator | Front Door | (bundled) |
| WAF | Front Door WAF / Application Gateway WAF | |
| VPC | VNet | Regional in both |
| Subnet | Subnet | |
| Security Group | NSG (partly) | NSG allows AND denies |
| NACL | NSG (also this — NSG combines both) | |
| Internet Gateway | (implicit) | Subnets get internet via public IP or NAT |
| NAT Gateway | NAT Gateway | Same |
| VPC Peering | VNet Peering | |
| VPC Endpoint / PrivateLink | Private Endpoint / Private Link | |
| Systems Manager Session Manager | Azure Bastion | Managed jump host |
| CloudFormation | ARM / Bicep | Bicep is the modern DSL |
| CDK | Bicep (roughly) | Both compile to underlying JSON |
| Systems Manager Parameter Store | Azure App Configuration | |
| Secrets Manager | Azure Key Vault | Key Vault also stores certs and keys |
| KMS | Azure Key Vault (keys section) | |
| CloudWatch | Azure Monitor + Log Analytics + App Insights | Three products split by concern |
| CloudTrail | Azure Activity Log + Diagnostic Logs | |
| GuardDuty | Microsoft Defender for Cloud | |
| Config | Azure Policy + Azure Resource Graph | |
| CloudTrail Insights | Microsoft Sentinel | SIEM |
| SNS | Azure Service Bus / Event Grid | Event Grid for events, Service Bus for messages |
| SQS | Service Bus Queues / Storage Queues | |
| Kinesis | Event Hubs | |
| Step Functions | Logic Apps / Durable Functions | Logic Apps for low-code, Durable Functions for code-first |
| CodePipeline | Azure Pipelines / GitHub Actions | |
| CodeBuild | Azure Pipelines / GitHub Actions | |
| ECR | Azure Container Registry (ACR) | |
| CodeCommit | Azure Repos | (Both losing to GitHub) |
| CloudFormation StackSets | Deployment Stacks | |

## Cost Model Differences Worth Knowing

**1. Reserved Capacity is Cheaper in Azure (for equivalent commitment).** Both clouds offer 1-year and 3-year reserved pricing. Azure's savings are often slightly higher for equivalent VM SKUs, especially with Azure Hybrid Benefit (reuse existing on-prem Windows Server / SQL Server licenses).

**2. Azure Free Services (that AWS Charges For):**
- AKS control plane (EKS is $73/mo)
- Front Door base tier includes WAF (AWS WAF is separate charge)
- Azure DevOps free tier is generous for CI/CD (compared to CodeBuild minutes)

**3. AWS Wins on Some Specifics:**
- S3 Standard is cheaper per GB than Blob Storage Hot (marginal but real)
- Lambda free tier is more generous than Functions Consumption free tier
- EBS gp3 is cheaper baseline than Premium SSD

**4. Data Egress — Similar Structure, Both Expensive**
Both charge for outbound data to the internet. Both are free for inbound. Both have "free" data movement within region. Charges kick in for cross-region and cross-cloud data. Same order of magnitude, ~$0.08-0.09/GB.

**5. Watch for Storage Account Redundancy Cost Multipliers**
LRS is 1x. ZRS is ~1.25x. GRS is ~2x. GZRS is ~2.5x. Choose deliberately — a bank might justify GZRS, a dev environment definitely doesn't need it. Same principle in S3 (Standard vs Standard-IA vs Cross-Region Replication).

**6. Reserved Instances vs Savings Plans**
Azure has Reserved Instances (specific VM series in a region for 1/3 years) and Savings Plans (commit to hourly compute spend). AWS has similar. Savings Plans are more flexible; RIs are stricter but cheaper. Same trade-off.

---

## Design Question: 3-Tier Web Application on Azure

**Q: Design a production-grade 3-tier web app on Azure. Highly available, secure, cost-conscious.**

> "Full architecture, walking through region choice first, then each tier.
>
> Region: East US primary, West US secondary for DR. Both have Availability Zone support — critical for HA design. Same region for prod resources unless there's a specific latency need.
>
> Resource groups: one per environment per lifecycle grouping.
> - `rg-prod-network-eastus` — VNet, NSGs, DNS zones (shared, rarely changes)
> - `rg-prod-web-eastus` — App Gateway, Front Door, web-tier resources
> - `rg-prod-app-eastus` — AKS cluster, container images, application secrets
> - `rg-prod-data-eastus` — Azure SQL, Cosmos DB, Blob Storage, Key Vault
>
> Networking layer:
> - VNet: 10.0.0.0/16
> - Subnets: web /24, app /24, db /24, bastion /27, appgw /24
> - NSGs enforce tier isolation with NSG-as-source rules
> - NAT Gateway for outbound from private tiers
> - Private DNS zones for internal service discovery
> - Azure Bastion for admin access
>
> Web tier:
> - Front Door (global L7) as public entry point — DNS points here
> - WAF policy on Front Door: OWASP + custom rules
> - Origin: Application Gateway in web subnet
> - Application Gateway → AKS backend pool via AGIC (Application Gateway Ingress Controller)
> - Managed certificates on both Front Door and App Gateway
>
> App tier:
> - AKS cluster, Standard tier with uptime SLA, Kubernetes 1.29
> - Node pools: system (3 nodes, D2s_v5), general (autoscale 3-15, D4s_v5), spot (autoscale 0-30 for batch)
> - All node pools spread across 3 AZs
> - Entra ID authentication + Azure RBAC for Kubernetes
> - Workload Identity for pod-level auth to Azure resources
> - Container Insights for observability
> - ArgoCD for GitOps deployment
> - Cert-manager for TLS certificates
>
> Data tier:
> - Azure SQL Database Business Critical (Gen5, 4 vCore, zone-redundant) for primary transactional data
> - Read replica for reporting workloads
> - Backup retention: 35 days short-term + 12 months long-term
> - Cosmos DB (NoSQL API) for session state and cart data — global replication to West US, session consistency
> - Blob Storage account (GZRS) for user uploads, lifecycle policy transitions to Cool after 30 days
> - All data services accessed via Private Endpoints — no public exposure
>
> Security layer:
> - Key Vault for secrets, certificates, encryption keys
> - Managed Identity everywhere — no service accounts with passwords
> - Azure Policy enforces compliance (no public storage, no public DBs, tag requirements)
> - Microsoft Defender for Cloud for threat detection
> - Sentinel for SIEM and security analytics
> - Enable diagnostic logs on everything, sent to central Log Analytics workspace
>
> DR strategy:
> - RTO 30 min, RPO 5 min for critical data
> - West US region has warm-standby AKS cluster (small, autoscales up on failover)
> - Azure SQL geo-replication to West US
> - Cosmos DB is already global (multi-region writes)
> - Blob Storage GZRS replicates automatically to paired region
> - Traffic Manager or Front Door priority routing for automatic failover
> - Terraform manifests + Bicep in Git — full infrastructure recreatable in 30 min
>
> Observability:
> - Azure Monitor as the umbrella
> - Log Analytics workspace as central log store
> - Application Insights for app-level telemetry, traces, metrics
> - Container Insights for AKS-specific
> - Alert rules → PagerDuty via webhook
> - Grafana on top of Log Analytics for custom dashboards
>
> CI/CD:
> - GitHub Actions for CI (build, test, push to ACR)
> - ArgoCD in AKS for CD (pull-based GitOps)
> - Separate service principals per pipeline with least-privilege RBAC
> - OIDC federation to Entra ID — no long-lived secrets
>
> Cost optimization notes:
> - VMs behind AKS as Reserved Instances (1-year commit) — 40% off baseline
> - Spot node pool absorbs batch and elastic workloads — 80% off
> - Azure SQL vCore purchasing model with Azure Hybrid Benefit if applicable
> - Storage lifecycle policies save on old data
> - Regularly review Azure Advisor cost recommendations
>
> This architecture achieves ~99.95% availability, defense-in-depth security, sub-100ms global latency via Front Door, automatic failover for regional outages, and clean cost profile.
>
> Direct AWS comparison: this is the same architecture as CloudFront + ALB + EKS + RDS Multi-AZ + DynamoDB Global Tables + S3 CRR. Different service names, identical shape. That transferability is exactly why cloud-agnostic architecture skills matter."

---

# Session Wrap-up

**What this document covers:**
- 15 Azure service categories mapped to AWS equivalents
- Real production configuration examples
- Console navigation for every service
- Key differences that catch AWS engineers off-guard
- Senior-level interview Q&A framed as spoken answers
- Cross-cloud terminology cheat sheet
- Complete 3-tier reference architecture

**Your target:** Multi-cloud fluency for MNC-level DevOps roles. When interviewers ask "have you worked with Azure?" the honest answer becomes "yes, I've built infrastructure on Azure and can compare architectural trade-offs vs AWS" — that's the level this doc gets you to with hands-on practice.

**Next steps to solidify:**
1. Sign up for Azure free tier — 12 months free + $200 credit
2. Recreate your flask-mongo-app deployment on Azure: AKS cluster + Cosmos DB + Blob Storage
3. Set up Bicep or Terraform for the whole thing — IaC exercise
4. Practice the interview questions aloud, especially the cross-cloud comparison ones
5. Build the same 3-tier reference architecture as a portfolio project — deploy on both AWS and Azure to demonstrate multi-cloud skill

**Security reminder from your AWS notes still applies:**
Never store Azure Service Principal secrets, Storage Account keys, SQL admin passwords, or Cosmos DB connection strings in plaintext notes. Use Key Vault. For CI/CD, use OIDC federation instead of long-lived secrets.