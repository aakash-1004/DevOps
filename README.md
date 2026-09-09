# DevOps-Mastery

Personal Obsidian vault of hands-on DevOps / Cloud engineering notes — Linux, Docker, Kubernetes, AWS, Azure, GCP, Terraform, CI/CD, monitoring — built while transitioning into a Cloud/DevOps Engineer role.

## Structure

```
DevOps-Mastery/
├── 00-Fundamentals/     # DevOps basics, SDLC, VMs, cloud provisioning
├── 00-Index/             # Dashboard.md — progress tracker + links + quick commands
├── 01-Linux-Bash/        # Core Linux, shell scripting, networking, perf troubleshooting, cron, log parsing
├── 02-Python-DevOps/     # JSON/YAML parsing, requests, subprocess
├── 03-Git/               # Git workflow
├── 04-Docker/            # Architecture, images/containers, Dockerfile, networking, storage, compose, interview Q&A
├── 05-Kubernetes/        # Core concepts, deployments, manifests lab, Helm, EKS/Fargate/IRSA/ALB project
├── 06-AWS/               # Core services, deployment notes, labs
├── 07-CICD/              # GitHub Actions, ArgoCD, Jenkins
├── 08-Terraform/         # Core concepts, labs
├── 09-Monitoring/        # Prometheus + Grafana
├── 10-Deployment/        # Frontend/backend concepts, framework deployment, Nginx vs Apache, process management
├── 11-Azure/             # Azure notes
├── 12-GCP/               # GCP notes
└── Devops master revision.md   # Full revision doc
```

## Start here

Open `DevOps-Mastery/00-Index/Dashboard.md` — it's the hub: module progress tracker, the golden-thread project (Taskmanager: Flask + MongoDB → Docker → K8s → Terraform → CI/CD → Monitoring), a quick-command cheat sheet, and wikilinks into every note.

1. Clone: `git clone https://github.com/aakash-1004/DevOps.git`
2. In Obsidian: **Open folder as vault** → select the `DevOps` folder itself (not `DevOps-Mastery`) — the `.obsidian` config lives at repo root, so opening `DevOps-Mastery` skips it.
3. The **AnuPpuccin** theme is bundled and already set as active, so it loads automatically — no manual theme install needed.
4. Open `00-Index/Dashboard.md` and pin it (right-click the tab → Pin) as your permanent home tab — every module links out from there.
5. Backlinks and Outgoing Links panes are already core-enabled, so use them to jump module-to-module instead of the file explorer.
6. Fix the `*.md.md` files first (item above) before you rely on search/graph view — right now they show up as broken/duplicate-looking entries.
7. If you're going to keep maintaining this: install the **Dataview** community plugin and turn Dashboard's module table into a live query instead of a hand-updated table — saves you re-editing it every time you finish a module.
