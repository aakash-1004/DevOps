# DevOps Master Revision — Zero to Hero

**Tags:** #devops #revision #interview #masterreview  
**Status:** ✅ Active Revision  
**Interview Relevance:** 🔴 Maximum — Full Stack DevOps Interview Prep  
**Author:** Aakash Rao | Cloud & Automation Engineer

---

> **How to use this note:** Every section follows: Concept → Core Commands → Real-World Context → Senior Interview Q&A. Senior Q&A is framed as answers a 10-year DevOps Tech Lead would give — opinionated, production-aware, and scenario-grounded.

---

# TABLE OF CONTENTS

1. [DevOps Fundamentals & SDLC](https://claude.ai/chat/b1cae23e-7e28-4e55-be9d-e54f24a46644#1-devops-fundamentals--sdlc)
2. [Linux & Bash Scripting](https://claude.ai/chat/b1cae23e-7e28-4e55-be9d-e54f24a46644#2-linux--bash-scripting)
3. [Git & Version Control](https://claude.ai/chat/b1cae23e-7e28-4e55-be9d-e54f24a46644#3-git--version-control)
4. [Docker](https://claude.ai/chat/b1cae23e-7e28-4e55-be9d-e54f24a46644#4-docker)
5. [Kubernetes](https://claude.ai/chat/b1cae23e-7e28-4e55-be9d-e54f24a46644#5-kubernetes)
6. [AWS Core Services](https://claude.ai/chat/b1cae23e-7e28-4e55-be9d-e54f24a46644#6-aws-core-services)
7. [Terraform (IaC)](https://claude.ai/chat/b1cae23e-7e28-4e55-be9d-e54f24a46644#7-terraform-iac)
8. [CI/CD — GitHub Actions & Jenkins](https://claude.ai/chat/b1cae23e-7e28-4e55-be9d-e54f24a46644#8-cicd--github-actions--jenkins)
9. [GitOps & ArgoCD](https://claude.ai/chat/b1cae23e-7e28-4e55-be9d-e54f24a46644#9-gitops--argocd)
10. [Monitoring — Prometheus & Grafana](https://claude.ai/chat/b1cae23e-7e28-4e55-be9d-e54f24a46644#10-monitoring--prometheus--grafana)
11. [Deployment — Nginx, Gunicorn, systemd](https://claude.ai/chat/b1cae23e-7e28-4e55-be9d-e54f24a46644#11-deployment--nginx-gunicorn-systemd)
12. [System Design for DevOps](https://claude.ai/chat/b1cae23e-7e28-4e55-be9d-e54f24a46644#12-system-design-for-devops)

---

---

# 1. DevOps Fundamentals & SDLC

## Concept

DevOps is a culture and set of practices that unify software Development (Dev) and IT Operations (Ops). The goal: **shorten the SDLC while delivering high-quality software continuously.**

Before DevOps, Dev teams threw code "over the wall" to Ops. Ops had no idea what changed. Deployments were quarterly events that caused incidents. DevOps fixes this by making both teams share responsibility across the full lifecycle.

**SDLC Phases:**

```
Plan → Code → Build → Test → Release → Deploy → Operate → Monitor → (back to Plan)
```

**Key DevOps Principles:**

- Everything as Code (IaC, PaC, config-as-code)
- Automate everything repeatable
- Shift Left — catch defects early (testing in CI, not after deployment)
- Continuous Feedback — monitoring feeds back into planning
- Fail Fast — detect issues at the earliest possible stage

**Types of Hypervisors:**

- **Type 1 (Bare-metal):** Runs directly on hardware. Examples: VMware ESXi, Hyper-V, KVM. Used in data centers, production. Better performance.
- **Type 2 (Hosted):** Runs on top of a host OS. Examples: VirtualBox, VMware Workstation. Used for development/learning. Your devops-labs VM uses VirtualBox (Type 2).

---

## Senior Interview Q&A

**Q: What is DevOps and how is it different from traditional IT?**

> "DevOps is fundamentally a cultural shift, not just a toolset. Traditional IT had hard walls between Dev and Ops — developers wrote code, threw it to operations, and operations deployed it without understanding what changed. This created slow releases, blame games, and high failure rates in production.
> 
> DevOps breaks that wall. Developers own code from commit to production, and operations provides the platform but also codes — IaC, automation scripts, pipelines. The result is faster delivery with more stability. When I say faster: Netflix deploys hundreds of times a day. That's only possible with DevOps culture and the tooling that supports it.
> 
> The key metrics that matter: DORA metrics — deployment frequency, lead time for changes, change failure rate, and time to restore service. DevOps organizations score better on all four."

---

**Q: What's the difference between Continuous Integration, Continuous Delivery, and Continuous Deployment?**

> "CI is about automatically building and testing every code commit. The moment a developer pushes, a pipeline runs — compile, unit tests, integration tests. If it fails, the developer knows in minutes, not days.
> 
> Continuous Delivery means the code is always in a deployable state after CI passes. It's ready to go to production, but a human still clicks 'deploy' — there's a manual gate. That gate exists for business reasons, not technical ones.
> 
> Continuous Deployment removes that human gate entirely. Every green build goes to production automatically. You need high test coverage and feature flags to do this safely.
> 
> Most companies I've worked with land on Continuous Delivery — CI all the way, automated staging deployments, manual production approval."

---

**Q: How does a Type 1 vs Type 2 hypervisor affect your infrastructure decisions?**

> "In production, you always use Type 1. KVM on Linux, ESXi for VMware environments — they sit on bare metal, no host OS overhead. You get near-native performance because the hypervisor talks directly to the hardware.
> 
> Type 2 is for your laptop. VirtualBox, VMware Workstation — they run inside your OS, so you're paying a double virtualization tax. Fine for learning, never for production.
> 
> In cloud context: AWS EC2 runs on KVM under the hood (Nitro hypervisor is a specialized KVM variant). So every EC2 instance you spin up is a Type 1 VM. This is why EC2 can offer consistent IOPS and network performance that you can SLA-guarantee."

---

---

# 2. Linux & Bash Scripting

## Concept

Linux is the operating system of DevOps. Every server you manage, every container that runs, every CI runner that builds your code — it's Linux. Not knowing Linux deeply is like being a mechanic who doesn't understand engines.

**The DevOps-critical Linux areas:**

1. Process management (ps, top, kill, systemctl)
2. Networking (ss, netstat, iptables, curl, dig)
3. File & text manipulation (grep, awk, sed, find, xargs)
4. Disk & storage (df, du, lsblk, mount, LVM)
5. User & permission management
6. Bash scripting automation

---

## Core Commands

```bash
# Process management
ps aux | grep <process>       # List all processes, filter by name
top -bn1                      # One-shot top output (useful in scripts)
kill -9 <PID>                 # Force kill a process
systemctl status <service>    # Check service status
systemctl restart <service>   # Restart a service
journalctl -u <service> -f    # Follow live logs for a service

# Networking
ss -tulnp                     # Show all listening ports with process names
netstat -tulnp                # Same as ss, older tool
curl -I https://example.com   # HTTP headers only (check if site is up)
dig google.com                # DNS lookup
ip addr show                  # Show all network interfaces and IPs
iptables -L -n -v             # List all firewall rules with verbosity

# File & text manipulation
grep -E "ERROR|WARN" app.log  # Extended regex: find ERRORs or WARNs
grep -r "keyword" /var/log/   # Recursive search in directory
awk '{print $1, $5}' file     # Print columns 1 and 5
awk -F: '{print $1}' /etc/passwd  # Use : as delimiter, print field 1
sed 's/old/new/g' file        # Replace all occurrences of old with new
sed -i 's/old/new/g' file     # In-place replacement (modifies the file)
find /var/log -name "*.log" -mtime +7 -delete  # Delete logs older than 7 days

# Disk & storage
df -h                         # Disk usage in human-readable format
du -sh /var/log/*             # Size of each item in /var/log
lsblk                         # List block devices (disks, partitions)
mount /dev/xvdf /data         # Mount a disk to a directory

# LVM (Logical Volume Management)
pvs / vgs / lvs               # Show physical volumes, volume groups, logical volumes
lvextend -L +5G /dev/vg/lv   # Extend a logical volume by 5GB
resize2fs /dev/vg/lv          # Resize the filesystem after extending LV

# User & permissions
useradd -m -s /bin/bash user  # Create user with home dir and bash shell
usermod -aG sudo user         # Add user to sudo group
passwd user                   # Set password for user
chmod 755 /path               # rwxr-xr-x permissions
chown user:group /path        # Change ownership
```

---

## Bash Scripting — The Production Pattern

```bash
#!/bin/bash
set -euo pipefail

# -e: exit immediately on error
# -u: treat unset variables as errors
# -o pipefail: catch failures in pipes (not just last command)
```

**Logging pattern:**

```bash
log_info()  { echo "[INFO]  $(date '+%Y-%m-%d %H:%M:%S') - $1" | tee -a "$LOG_FILE"; }
log_error() { echo "[ERROR] $(date '+%Y-%m-%d %H:%M:%S') - $1" | tee -a "$LOG_FILE" >&2; }
```

**Lock file pattern (prevent duplicate cron runs):**

```bash
LOCKFILE="/tmp/script.lock"
cleanup() { rm -f "$LOCKFILE"; }
trap cleanup EXIT                    # Always clean up on exit (even on error)

if [[ -f "$LOCKFILE" ]]; then
  log_error "Already running. Exiting."
  exit 1
fi
touch "$LOCKFILE"
```

**The `|| FAILED=1` pattern (run all checks, fail at end):**

```bash
FAILED=0
check_cpu    || FAILED=1             # Run check; if it fails, set flag — don't exit
check_memory || FAILED=1
check_disk   || FAILED=1

[[ $FAILED -eq 1 ]] && { log_error "One or more checks failed"; exit 1; }
```

**Cron syntax:**

```
* * * * * command
│ │ │ │ └── Day of week (0=Sun, 6=Sat)
│ │ │ └──── Month (1-12)
│ │ └────── Day of month (1-31)
│ └──────── Hour (0-23)
└────────── Minute (0-59)

0 2 * * *  /opt/scripts/backup.sh      # Every day at 2 AM
*/5 * * * * /opt/scripts/health.sh     # Every 5 minutes
```

---

## Senior Interview Q&A

**Q: Production script is running via cron and occasionally hangs. How do you handle it?**

> "Three things. First, lock file — if a previous instance is still running, the new one exits immediately rather than spawning a zombie. Second, timeout wrapper — `timeout 300 ./script.sh` kills it after 5 minutes if it hangs. Third, structured logging to a file with timestamps — so when it does hang, I can see exactly where it stopped.
> 
> Beyond that, I'd check what the script is waiting on. Usually it's a network call with no timeout (curl without `--max-time`), or a database query with no timeout set. Those will hang indefinitely. Always set timeouts on any external call inside a script."

---

**Q: Walk me through how you'd debug a server that's responding slowly.**

> "Systematic approach: start with the top-level symptoms and drill down.
> 
> First, `top` or `htop` — is CPU maxed? If yes, which process? `top` sorted by CPU shows you immediately.
> 
> If CPU is fine, check memory: `free -h`. Is swap being used heavily? Heavy swap means you're OOM-adjacent — the kernel is swapping pages to disk, which destroys performance. Check which process is consuming memory with `ps aux --sort=-%mem | head`.
> 
> If CPU and memory are fine, check disk: `iostat -x 1` — are you hitting 100% disk utilization? Are you hitting IOPS limits on EBS?
> 
> If that's fine, check network: `ss -tulnp` — are you exhausting connection limits? `netstat -s` shows TCP retransmissions.
> 
> Last layer: application-level. Check application logs, error rates, slow query logs in the DB.
> 
> In my experience, 80% of slow server issues are either memory pressure causing swap, or a runaway process eating CPU, or a DB query without an index."

---

**Q: What's the difference between `grep`, `awk`, and `sed` and when do you use each?**

> "They solve different problems.
> 
> `grep` is a filter — you use it when you want to find lines that match a pattern. `grep ERROR app.log`. It doesn't transform, it selects. Use `-E` for extended regex when you need things like `{1,3}` quantifiers or `|` OR conditions.
> 
> `awk` is a column processor. It treats each line as fields separated by a delimiter. Use it when you want to extract or compute on specific columns: `awk '{print $1, $5}'` or `awk -F: '$3 >= 1000 {print $1}' /etc/passwd` to find non-system users. It's also got arithmetic, so you can do `awk '{sum += $2} END {print sum}'` for aggregates.
> 
> `sed` is a stream editor for transformations. You use it when you want to modify content in place — replace strings, delete lines, insert text. `sed 's/old/new/g'` for replacement, `sed -i` to modify the file directly.
> 
> In a pipeline they complement each other: `grep ERROR app.log | awk '{print $5}' | sort | uniq -c | sort -rn` — filter → extract column → count → rank."

---

**Q: Explain iptables. What are chains and tables?**

> "iptables is Linux's packet filtering framework. It sits in the kernel and makes decisions on every packet.
> 
> It has tables for different purposes: `filter` (the default — controls what traffic is allowed), `nat` (network address translation — source/destination NAT), `mangle` (packet marking, QoS), and `raw` (connection tracking bypass).
> 
> Within each table are chains — predefined points in the packet path where rules are evaluated: `INPUT` (packets destined for this host), `OUTPUT` (packets originating from this host), `FORWARD` (packets passing through this host as a router), `PREROUTING` (before routing decision), `POSTROUTING` (after routing decision).
> 
> Every packet traverses the chains in order. First matching rule wins. If no rule matches, the chain policy applies (ACCEPT or DROP).
> 
> In Kubernetes, iptables is what actually implements Service routing. When you create a ClusterIP service, kube-proxy writes iptables rules that NAT the ClusterIP to one of the backend pod IPs. That's why you can't `ping` a ClusterIP — it's a virtual IP that only works via iptables NAT, not ICMP."

---

---

# 3. Git & Version Control

## Concept

Git is distributed version control. Every clone is a full repository. This means you can work offline, branch freely, and merge from any source.

**The mental model:** Git tracks snapshots of your project, not diffs. Every commit is a complete snapshot. The diff you see is computed on the fly between two snapshots.

**Key objects:**

- **Blob** — file content
- **Tree** — directory structure
- **Commit** — snapshot + metadata + parent pointer
- **Tag** — named pointer to a commit

---

## Core Commands

```bash
# Daily workflow
git status                            # What's changed, what's staged
git add -p                            # Interactive staging — review hunks before adding
git commit -m "feat: add health check"
git push origin feature/my-feature

# Branching
git checkout -b feature/new-thing     # Create and switch to new branch
git branch -d feature/done            # Delete local branch
git push origin --delete feature/done # Delete remote branch

# Merging & rebasing
git merge feature/new-thing           # Merge with merge commit (preserves history)
git rebase main                       # Replay commits on top of main (linear history)
git rebase -i HEAD~3                  # Interactive rebase: squash/edit last 3 commits

# Conflict resolution
git status                            # Shows conflicted files
# Edit conflicted files, remove <<<<< ===== >>>>> markers
git add <resolved-file>
git rebase --continue                 # Or: git merge --continue

# Undoing things
git reset --soft HEAD~1               # Undo last commit, keep changes staged
git reset --mixed HEAD~1              # Undo last commit, keep changes unstaged
git reset --hard HEAD~1               # Undo last commit, DISCARD changes (dangerous)
git revert <commit-hash>              # Create new commit that undoes a commit (safe for shared branches)

# Inspection
git log --oneline --graph --all       # Visual branch graph
git diff HEAD~1 HEAD                  # Diff last commit vs current
git stash                             # Temporarily shelve changes
git stash pop                         # Restore stashed changes

# SSH key setup
ssh-keygen -t ed25519 -C "aakash@zehntech.com"
cat ~/.ssh/id_ed25519.pub             # Copy this to GitHub → Settings → SSH Keys
ssh -T git@github.com                 # Test SSH connection
```

---

## Senior Interview Q&A

**Q: Merge vs rebase — when do you use each?**

> "The decision comes down to history vs simplicity.
> 
> Merge preserves the full history — you can see exactly when a feature branch diverged and when it was integrated. It creates a merge commit. This is valuable for audit trails and understanding how features were developed in parallel. Use merge for integrating long-lived feature branches into main, and especially for things like release branches where you want a clear record.
> 
> Rebase replays your commits on top of the target branch, creating a linear history. It looks like the work was done sequentially even if it wasn't. This makes `git log` much cleaner and makes `git bisect` more powerful because history is linear.
> 
> My rule: rebase your personal feature branches before opening a PR. Never rebase shared branches that others have pulled — you're rewriting history and everyone else's branch will diverge. For shared branches, always merge.
> 
> In practice at most teams: developers rebase locally to clean up their commits, then the PR merge uses a squash merge or a regular merge commit into main."

---

**Q: What's `git reset --soft` vs `--hard` and when would you use them in production?**

> "All three reset modes move HEAD to a different commit. The difference is what they do with your working tree and staging area.
> 
> `--soft` moves HEAD only. Your changes are still staged. Use this when you made a commit but want to restructure it — maybe split it into two commits, or fix the commit message, or add a file you forgot.
> 
> `--mixed` (the default) moves HEAD and unstages changes. Your working tree is unchanged. Use this when you want to recommit differently.
> 
> `--hard` moves HEAD, unstages, AND discards working tree changes. This is destructive. Use it when you want to completely abandon work and return to a known state.
> 
> In production context: I'd never `git reset --hard` on a shared branch. If something went to main that shouldn't have, I use `git revert` — it creates a new commit that undoes the bad one, preserving history. `reset --hard` rewrites history, which breaks everyone who has pulled. `revert` is the production-safe tool."

---

**Q: How do you structure a Git branching strategy for a team?**

> "It depends on deployment cadence. Two main models I've used:
> 
> **GitFlow** for teams with scheduled releases: main (production), develop (integration), feature/* (features), release/* (stabilization), hotfix/* (emergency fixes). Good for mobile apps or enterprise software where releases are versioned. Downside: complex, long-lived branches, merge conflicts.
> 
> **Trunk-based development** for teams doing continuous deployment: everyone branches off main, keeps branches short-lived (1-2 days max), merges back frequently. Feature flags hide incomplete features in production. This is what Google, Netflix, and Etsy use. It requires good CI and feature flags to work.
> 
> My recommendation for most teams: trunk-based with short feature branches, squash merges into main, and automated pipelines triggered on main. It scales better and forces you to integrate continuously rather than doing a big bang merge at the end."

---

---

# 4. Docker

## Concept

Docker packages your application and all its dependencies into a portable unit called a container. A container is an isolated process running on the host kernel — it's NOT a VM. Containers share the host kernel but have isolated: filesystem (via Union FS layers), network, process space, and resource limits (via cgroups).

**The key insight:** "It works on my machine" dies because containers make your environment reproducible. The container IS the environment.

**Docker Architecture:**

```
Docker CLI  →  Docker Daemon (dockerd)  →  containerd  →  runc
     (client)        (API server)              (runtime)    (OCI runtime)
```

**Image Layers (Union FS):**

- Each `RUN`, `COPY`, `ADD` instruction creates a new layer
- Layers are cached — if layer N hasn't changed, Docker reuses it
- Build order matters: put frequently-changing layers (COPY . .) last
- Layers are read-only. A container adds a thin writable layer on top

---

## Core Commands

```bash
# Image management
docker build -t myapp:1.0 .                # Build image from Dockerfile in current dir
docker build -t myapp:1.0 -f Dockerfile.prod .  # Use specific Dockerfile
docker images                              # List local images
docker pull nginx:1.25                     # Pull specific version (always pin versions!)
docker push aakash0908/myapp:1.0          # Push to Docker Hub
docker rmi myapp:1.0                       # Remove image
docker image prune                         # Remove dangling images

# Container lifecycle
docker run -d -p 8080:5000 --name myapp myapp:1.0    # Detached, port mapping, named
docker run -it ubuntu:22.04 /bin/bash               # Interactive terminal
docker run --rm -e DB_URL=mongodb://... myapp:1.0   # Auto-remove, env var
docker ps                                            # Running containers
docker ps -a                                         # All containers including stopped
docker stop myapp && docker rm myapp                 # Stop then remove
docker exec -it myapp /bin/bash                      # Shell into running container
docker logs myapp -f                                 # Follow live logs
docker inspect myapp                                 # Full container metadata as JSON

# Networking
docker network ls                          # List networks
docker network create mynet               # Create custom bridge network
docker network inspect mynet             # Show connected containers and IPs
docker run --network mynet myapp:1.0     # Connect container to network

# Volumes
docker volume create mydata              # Create named volume
docker run -v mydata:/app/data myapp     # Mount named volume
docker run -v $(pwd):/app myapp          # Bind mount (dev use — live code reload)
docker volume ls                         # List volumes

# Cleanup
docker system prune -af                  # Remove ALL unused: images, containers, networks
docker volume prune                      # Remove unused volumes
```

---

## Dockerfile — Production Pattern

```dockerfile
# Stage 1: Build
FROM python:3.11-slim AS builder
WORKDIR /app
COPY requirements.txt .
RUN pip install --user --no-cache-dir -r requirements.txt

# Stage 2: Runtime (smaller final image)
FROM python:3.11-slim
WORKDIR /app

# Create non-root user FIRST
RUN useradd -m -u 1001 appuser

# Copy only installed packages from builder
COPY --from=builder /root/.local /root/.local
COPY . .

# Set PATH, switch to non-root user
ENV PATH=/root/.local/bin:$PATH
USER appuser

EXPOSE 5000
CMD ["gunicorn", "--bind", "0.0.0.0:5000", "app:app"]
```

**Why multi-stage?** Your final image doesn't contain build tools (gcc, pip cache, etc.). A Python app that's 800MB with build tools becomes 150MB runtime-only.

---

## Docker Compose

```yaml
version: "3.9"
services:
  web:
    build: .
    ports:
      - "8080:5000"
    environment:
      - MONGO_URL=mongodb://mongo:27017/taskdb
    depends_on:
      - mongo
    networks:
      - appnet
    restart: unless-stopped

  mongo:
    image: mongo:6
    volumes:
      - mongodata:/data/db
    networks:
      - appnet
    restart: unless-stopped

volumes:
  mongodata:

networks:
  appnet:
    driver: bridge
```

```bash
docker compose up -d         # Start all services detached
docker compose down          # Stop and remove containers
docker compose down -v       # Also remove volumes (wipes data)
docker compose logs -f web   # Follow logs for specific service
docker compose exec web sh   # Shell into running service
docker compose ps            # Status of all services
```

---

## Senior Interview Q&A

**Q: How do Docker containers achieve isolation?**

> "Containers use two kernel features: namespaces and cgroups.
> 
> Namespaces provide isolation of system resources. There are six: PID namespace (container processes have their own PID tree, PID 1 inside is not PID 1 on host), network namespace (own network interfaces, routing table, iptables), mount namespace (own filesystem view), UTS namespace (own hostname), IPC namespace (own inter-process communication), user namespace (map container root to unprivileged host user).
> 
> cgroups (control groups) provide resource limits — you can cap a container at 512MB RAM and 0.5 CPU. Without cgroups, a runaway container could OOM the host.
> 
> Together: namespaces make each container think it's alone on the system, cgroups ensure it can't consume more than its share.
> 
> This is fundamentally different from VMs: VMs virtualize hardware, containers virtualize the OS. That's why containers start in milliseconds (no OS boot) but VMs take minutes."

---

**Q: Container keeps restarting. How do you debug it?**

> "Classic scenario. Four things to check in order:
> 
> `docker ps` — shows restart count and status. If status is 'Restarting', it's crashing and Docker is restarting it per the restart policy.
> 
> `docker logs <container>` — look at the last few lines before the crash. Application errors, missing config, permission denied, port already in use — 90% of crash reasons show up here.
> 
> `docker inspect <container>` — look at the ExitCode in State. Code 1 is generic error, code 137 is OOM kill (cgroup killed it for exceeding memory limit), code 139 is segfault, code 127 is command not found.
> 
> If the container crashes before you can exec into it: override the entrypoint to get a shell: `docker run -it --entrypoint /bin/sh myapp:1.0`. This lets you inspect the filesystem, check if env vars are set, check if config files exist.
> 
> In production on Kubernetes: `kubectl describe pod` shows Events (OOMKilled, CrashLoopBackOff reasons), `kubectl logs <pod> --previous` shows logs from the crashed instance."

---

**Q: What's the difference between CMD and ENTRYPOINT?**

> "Both define what runs when a container starts, but they interact differently with arguments.
> 
> ENTRYPOINT sets the executable — it's the main process, and it can't be easily overridden at runtime (you need `--entrypoint` flag). CMD provides default arguments to ENTRYPOINT, or if ENTRYPOINT isn't set, CMD is the command to run.
> 
> The common pattern I use: ENTRYPOINT is the executable, CMD is the default arguments. This means someone can do `docker run myapp --debug` and it appends `--debug` to the entrypoint command without replacing it.
> 
> If you only use CMD: `docker run myapp /bin/sh` replaces CMD entirely, which is sometimes what you want for debugging.
> 
> In production: always use the exec form (`["executable", "arg"]`) not shell form (`executable arg`). Shell form wraps the command in `/bin/sh -c`, which means your process is not PID 1 — it's the shell. That means Docker stop signals don't reach your app, and you get ugly shutdown behavior."

---

**Q: How do you reduce Docker image size?**

> "Six strategies I apply to every production image:
> 
> 1. Multi-stage builds — build in one stage, copy only artifacts to runtime stage. No build tools in final image.
> 2. Use slim or alpine base images — `python:3.11-slim` vs `python:3.11`. 150MB vs 900MB.
> 3. Clean up in the same RUN layer — `apt-get install -y curl && apt-get clean && rm -rf /var/lib/apt/lists/*`. If you clean in a separate RUN, the install layer is still in the image.
> 4. Use `.dockerignore` — exclude `.git`, `node_modules`, `__pycache__`, test files.
> 5. Combine RUN commands — every RUN creates a layer. Chain with && to keep it one layer.
> 6. Don't copy unnecessary files — only COPY what the runtime needs."

---

**Q: Docker networking — how does a container reach another container?**

> "Docker has four network modes:
> 
> **bridge** (default): Docker creates a virtual bridge (`docker0`). Each container gets a virtual ethernet on that bridge. Containers on the same bridge can reach each other by IP. For DNS resolution by name, you need a user-defined bridge network — not the default bridge. That's why Compose creates its own network: so services can reach each other by service name.
> 
> **host**: Container shares the host's network namespace. No port mapping needed — a container listening on port 5000 IS listening on host port 5000. Best performance, worst isolation. Use for high-throughput networking where NAT overhead matters.
> 
> **none**: No network. Complete isolation. Used for batch jobs or security-sensitive workloads.
> 
> **overlay**: For Docker Swarm — spans multiple hosts. Built on VXLAN. In Kubernetes, this is replaced by CNI plugins (Calico, Flannel, Cilium).
> 
> User-defined bridge is what I use in Compose: containers get DNS resolution by service name, isolated network per app, no cross-contamination."

---

---

# 5. Kubernetes

## Concept

Kubernetes is a container orchestration platform. It solves the problem that Docker alone doesn't: running containers at scale across multiple machines with health management, scaling, load balancing, rolling updates, and self-healing.

**The core principle:** You declare desired state. Kubernetes continuously reconciles actual state to match desired state. This is the reconciliation loop.

**Architecture:**

```
Control Plane:
  kube-apiserver      → All communication goes through here (the brain)
  etcd                → Distributed key-value store: source of truth for all cluster state
  kube-scheduler      → Decides which node a pod runs on
  kube-controller-manager → Runs controllers (ReplicaSet, Node, Endpoint, etc.)
  cloud-controller-manager → Cloud-provider specific (AWS, GCP, Azure)

Worker Nodes:
  kubelet             → Agent on each node: ensures containers are running
  kube-proxy          → Implements Service networking (iptables/IPVS rules)
  Container runtime   → containerd (or CRI-O) — actually runs containers
```

**Pod — the atomic unit:** A Pod is one or more containers sharing a network namespace (same IP) and storage. All containers in a Pod communicate via localhost.

---

## Core Commands

```bash
# Cluster info
kubectl cluster-info                  # API server and CoreDNS endpoints
kubectl get nodes                     # List nodes and their status
kubectl top nodes                     # CPU/memory usage per node
kubectl describe node <node>         # Detailed node info (capacity, allocations)

# Pod management
kubectl get pods -n <namespace>      # List pods in namespace
kubectl get pods -A                  # All pods across all namespaces
kubectl describe pod <pod>           # Detailed pod info + Events (crucial for debugging)
kubectl logs <pod>                   # Pod logs
kubectl logs <pod> -c <container>    # Logs from specific container in pod
kubectl logs <pod> --previous        # Logs from crashed previous instance
kubectl exec -it <pod> -- /bin/bash  # Shell into pod
kubectl delete pod <pod>             # Delete pod (Deployment will recreate it)

# Apply/Delete resources
kubectl apply -f deployment.yaml     # Create or update resource from file
kubectl delete -f deployment.yaml    # Delete resources defined in file
kubectl apply -f ./k8s/             # Apply all YAML files in a directory

# Deployments
kubectl get deployments             # List deployments
kubectl rollout status deployment/<name>   # Check rollout progress
kubectl rollout history deployment/<name>  # Show revision history
kubectl rollout undo deployment/<name>     # Rollback to previous version
kubectl set image deployment/<name> <container>=<image:tag>  # Update image

# Scaling
kubectl scale deployment <name> --replicas=5   # Manual scale
kubectl autoscale deployment <name> --min=2 --max=10 --cpu-percent=70  # HPA

# Services
kubectl get svc                      # List services
kubectl port-forward svc/<name> 8080:80  # Forward local port to service

# ConfigMaps & Secrets
kubectl create configmap myconfig --from-file=./config/   # From files
kubectl create secret generic mysecret --from-literal=DB_PASS=secret123
kubectl get secret mysecret -o jsonpath='{.data.DB_PASS}' | base64 -d  # Decode

# RBAC
kubectl get roles,rolebindings -n <ns>        # List RBAC in namespace
kubectl auth can-i get pods --as=dev-user     # Test permissions

# Debugging
kubectl get events --sort-by=.metadata.creationTimestamp  # All events sorted
kubectl run debug --image=busybox --rm -it -- sh          # Throw away debug pod
```

---

## Key YAML Manifests

**Deployment:**

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: flask-app
  namespace: production
spec:
  replicas: 3
  selector:
    matchLabels:
      app: flask-app
  template:
    metadata:
      labels:
        app: flask-app
    spec:
      containers:
        - name: flask
          image: aakash0908/flask-app:1.2
          ports:
            - containerPort: 5000
          envFrom:
            - configMapRef:
                name: flask-config
            - secretRef:
                name: flask-secrets
          resources:
            requests:
              cpu: "100m"
              memory: "128Mi"
            limits:
              cpu: "500m"
              memory: "256Mi"
          livenessProbe:
            httpGet:
              path: /health
              port: 5000
            initialDelaySeconds: 10
            periodSeconds: 5
          readinessProbe:
            httpGet:
              path: /ready
              port: 5000
            initialDelaySeconds: 5
            periodSeconds: 3
```

**Service:**

```yaml
apiVersion: v1
kind: Service
metadata:
  name: flask-svc
  namespace: production
spec:
  selector:
    app: flask-app
  ports:
    - port: 80
      targetPort: 5000
  type: ClusterIP       # ClusterIP | NodePort | LoadBalancer
```

**HPA:**

```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: flask-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: flask-app
  minReplicas: 2
  maxReplicas: 10
  metrics:
    - type: Resource
      resource:
        name: cpu
        target:
          type: Utilization
          averageUtilization: 70
```

---

## Helm

```bash
helm repo add stable https://charts.helm.sh/stable   # Add a chart repo
helm repo update                                      # Refresh repo index
helm search repo nginx                                # Search for charts

helm install myrelease ./mychart                     # Install from local chart
helm install myrelease ./mychart -f custom-values.yaml  # Override values
helm upgrade myrelease ./mychart -f custom-values.yaml  # Upgrade
helm rollback myrelease 1                            # Rollback to revision 1
helm history myrelease                               # Revision history
helm uninstall myrelease                             # Uninstall (remove all resources)

helm template ./mychart                             # Render YAML without installing (debugging)
helm lint ./mychart                                 # Validate chart syntax
```

**Helm chart structure:**

```
mychart/
├── Chart.yaml           # Chart metadata (name, version, appVersion)
├── values.yaml          # Default values
├── templates/
│   ├── deployment.yaml  # Uses {{ .Values.image.tag }}
│   ├── service.yaml
│   ├── _helpers.tpl     # Template helpers/partials
│   └── NOTES.txt        # Post-install instructions
```

---

## Senior Interview Q&A

**Q: What happens when you `kubectl apply -f deployment.yaml`?**

> "Full lifecycle:
> 
> 1. kubectl serializes the YAML and sends a PUT/PATCH HTTP request to the kube-apiserver.
> 2. kube-apiserver authenticates (your kubeconfig credentials), authorizes (RBAC check), then validates the schema.
> 3. The API server writes the desired state to etcd.
> 4. The Deployment controller (inside kube-controller-manager) is watching for Deployment changes via the watch API. It detects the new/updated Deployment.
> 5. The Deployment controller creates or updates a ReplicaSet to match the desired replica count.
> 6. The ReplicaSet controller creates Pod objects.
> 7. The scheduler watches for unscheduled pods (pods with no `nodeName`). It runs its scheduling algorithm: filters nodes that meet resource requests, then ranks remaining nodes. It assigns a node by setting `nodeName` on the Pod.
> 8. The kubelet on the selected node is watching for pods assigned to its node. It sees the new pod, calls containerd to pull the image and start the container.
> 9. kubelet reports back container status to the API server.
> 10. kube-proxy on all nodes detects service endpoint changes and updates iptables rules accordingly.
> 
> All of this happens in seconds. It's a beautiful system of independent controllers all watching etcd and reacting."

---

**Q: Pod is in CrashLoopBackOff. Walk me through your diagnosis.**

> "CrashLoopBackOff means the container started, ran, crashed, and Kubernetes has restarted it enough times that it's now adding exponential backoff delay before trying again.
> 
> Step 1: `kubectl describe pod <pod> -n <namespace>`. Look at the Events section — this is gold. You'll often see the exact error: OOMKilled, image pull error, readiness probe failed, config mount error.
> 
> Step 2: `kubectl logs <pod> --previous`. The `--previous` flag gets logs from the crashed instance, not the current (potentially-starting) one. This shows the actual crash reason.
> 
> Common causes and fixes:
> 
> - **OOMKilled** → container hit its memory limit, kernel killed it. Fix: increase memory limit or fix memory leak.
> - **Missing env var** → app fails to start because required config isn't there. Check your ConfigMap/Secret references.
> - **Command not found** → entrypoint binary doesn't exist. Check your Dockerfile CMD/ENTRYPOINT.
> - **Permission denied** → app can't read a mounted secret or config file. Check file permissions, check if running as non-root.
> - **Readiness probe failure** → probe path doesn't exist or app isn't ready in time. Adjust initialDelaySeconds.
> 
> If logs don't help: `kubectl exec -it <pod> --image-override busybox -- sh` to get into the container environment and inspect manually."

---

**Q: Explain the difference between liveness and readiness probes.**

> "They solve different problems.
> 
> **Readiness probe:** Is this pod ready to receive traffic? If readiness fails, Kubernetes removes the pod's endpoint from the Service — it stops sending it new requests, but doesn't kill the container. This is used for: graceful startup (don't send traffic until the app has loaded its cache), graceful degradation (temporarily remove a pod from rotation if its backend is down, add it back when the backend recovers).
> 
> **Liveness probe:** Is this pod alive? If liveness fails repeatedly, Kubernetes kills and restarts the container. This handles deadlocks, infinite loops — situations where the process is running but effectively stuck and will never recover.
> 
> Common mistake I've seen: people set liveness probes too aggressively. If `initialDelaySeconds` is too low and your app takes 20 seconds to start, liveness will kill it before it ever becomes healthy, creating a death loop.
> 
> **Startup probe** (the third one, often forgotten): Is this pod still starting? It's used for slow-starting apps to prevent liveness from killing them during initialization. Once startup probe succeeds, liveness and readiness take over."

---

**Q: How does Kubernetes networking work? How does a request reach a pod?**

> "Let's trace a request from outside the cluster to a pod.
> 
> External request → LoadBalancer (cloud LB, e.g., AWS NLB) → NodePort on any cluster node → ClusterIP Service → Pod IP.
> 
> ClusterIP is a virtual IP. It doesn't exist on any interface. kube-proxy on every node writes iptables DNAT rules: 'traffic to ClusterIP:port, NAT to one of these pod IPs'. The kernel intercepts the packet before routing and rewrites the destination to a real pod IP. The reply goes back NAT'd to the original source.
> 
> Pod-to-pod networking: every pod gets a unique IP from the pod CIDR. The CNI plugin (Flannel, Calico, Cilium) ensures pods on different nodes can reach each other — typically via overlay networking (VXLAN encapsulation) or BGP route advertisement.
> 
> Calico uses BGP: each node advertises 'I have pods in subnet X' to other nodes. No encapsulation overhead — packets route natively. Flannel uses VXLAN: encapsulates pod packets in UDP datagrams to cross node boundaries.
> 
> This is why every pod needs a unique IP cluster-wide: because there's no NAT between pods, and routing requires unique addresses."

---

**Q: How does Helm differ from just `kubectl apply`?**

> "kubectl apply gives you CRUD on Kubernetes resources. Helm gives you release management.
> 
> With kubectl apply: you have a bunch of YAML files. There's no concept of grouping, versioning, or rollback. If you `kubectl delete` a deployment, it's gone. There's no 'which version was deployed and what did it look like' history.
> 
> Helm treats a set of Kubernetes resources as a single release. It stores release history in Secrets in the cluster. `helm rollback myrelease 1` doesn't just undo one resource — it rolls back the entire set of resources to the exact state of revision 1. That includes Deployments, Services, ConfigMaps, RBAC — everything in the chart.
> 
> Helm also handles templating: the same chart deploys to dev with 1 replica and `debug` image tag, and to production with 10 replicas and `1.5.2` image tag — just different values files.
> 
> When NOT to use Helm: for one-off debugging or simple single-resource deploys. The overhead isn't worth it. For anything production and multi-resource, Helm is the right tool."

---

---

# 6. AWS Core Services

## Concept

AWS is the dominant cloud provider. DevOps engineers primarily interact with: compute (EC2, ECS, Lambda), storage (S3, EBS, EFS), networking (VPC, Route53, CloudFront), and IAM.

**The mental model:** Every AWS resource lives in a region. Regions contain Availability Zones (physically separate data centers). High availability = spreading across AZs.

---

## IAM — Identity & Access Management

**Core concepts:**

- **User**: Human or machine identity with credentials
- **Role**: Assumable identity — for EC2 instances, Lambda functions, CI/CD tools
- **Policy**: JSON document defining permissions (Allow/Deny + Actions + Resources)
- **Group**: Collection of users sharing policies

**Least privilege principle:** Start with zero permissions. Grant only exactly what's needed.

```bash
# Check what your current identity can do
aws sts get-caller-identity              # Who am I?
aws iam list-attached-user-policies --user-name devops-admin  # What policies do I have?

# S3
aws s3 ls                               # List all buckets
aws s3 ls s3://mybucket/                # List bucket contents
aws s3 cp file.txt s3://mybucket/       # Upload file
aws s3 sync ./dist s3://mybucket/       # Sync local dir to S3
aws s3 rb s3://mybucket --force         # Delete bucket and all contents

# EC2
aws ec2 describe-instances              # List all instances
aws ec2 run-instances \
  --image-id ami-0abcdef1234567890 \
  --instance-type t3.micro \
  --key-name mykey \
  --security-group-ids sg-xxxxx \
  --subnet-id subnet-xxxxx              # Launch EC2 instance

aws ec2 describe-instances \
  --filters "Name=instance-state-name,Values=running" \
  --query 'Reservations[].Instances[].[InstanceId,PublicIpAddress,Tags[?Key==`Name`].Value]' \
  --output table                        # Running instances with IPs and names
```

---

## VPC — Virtual Private Cloud

**Concepts:**

- **VPC**: Your private network in AWS (you define IP range, e.g., 10.0.0.0/16)
- **Subnet**: Subdivision of VPC. Public subnet = has route to Internet Gateway. Private subnet = no direct internet access.
- **Internet Gateway (IGW)**: Allows public subnets to communicate with the internet
- **NAT Gateway**: Allows private subnet instances to initiate outbound internet connections (for updates, etc.) without being reachable from outside
- **Security Group**: Stateful firewall at the instance level. Allow rules only.
- **NACL**: Stateless firewall at the subnet level. Allow and Deny rules.
- **Route Table**: Rules controlling where subnet traffic is directed

**3-tier architecture:**

```
Internet → IGW → Public Subnet (Load Balancer)
                      ↓
               Private Subnet (App Servers — EC2/ECS)
                      ↓
               Private Subnet (Database — RDS)
```

---

## S3 — Simple Storage Service

**S3 is object storage** — not a filesystem. Objects have keys (paths) and values (content). Max object size: 5TB.

**S3 storage classes (cost optimization):**

- **Standard**: Frequently accessed data. High cost, low latency.
- **Intelligent-Tiering**: Auto-moves between frequent/infrequent based on access patterns.
- **Standard-IA**: Infrequently accessed. Lower storage cost, retrieval fee.
- **Glacier**: Archives. Minutes to hours retrieval.
- **Glacier Deep Archive**: Coldest. 12-hour retrieval. Cheapest.

**Common DevOps uses:** Terraform remote state, CI/CD artifact storage, log archiving, static website hosting, Docker image layers (via ECR).

---

## Senior Interview Q&A

**Q: What's the difference between IAM Role and IAM User? When do you use each?**

> "IAM User is a permanent identity with long-lived credentials — access key + secret key. You use it for humans (developers, DevOps engineers) who log in to the console or use CLI locally.
> 
> IAM Role is an identity that's assumed, not permanently held. It has no long-lived credentials — when assumed, it generates short-lived STS tokens (typically 1 hour). You use roles for:
> 
> - EC2 instances (instance profile) — the instance assumes a role so your application code can call AWS APIs without storing keys on the instance
> - Lambda functions
> - ECS task execution
> - Cross-account access
> - GitHub Actions via OIDC (instead of storing AWS keys as GitHub Secrets)
> 
> The golden rule: **never store AWS access keys on an EC2 instance or in code**. Always use IAM roles for machine-to-AWS authentication. If you see `aws_access_key_id` in a config file on a server, that's a security finding."

---

**Q: You need to give your EC2 app write access to S3. How do you do it?**

> "Create an IAM Role with an S3 policy. Attach the role to the EC2 instance as an instance profile.
> 
> The policy would be: Action: `s3:PutObject`, `s3:GetObject`, Resource: `arn:aws:s3:::mybucket/*`. No `s3:*` — least privilege.
> 
> The application uses the AWS SDK without any explicit credentials. The SDK automatically calls the EC2 instance metadata service (`169.254.169.254/latest/meta-data/iam/security-credentials/`) to get temporary credentials from the role. It refreshes them before expiry.
> 
> Never add credentials to the application code or environment variables. The instance role handles it transparently."

---

**Q: Walk me through VPC design for a production application.**

> "I'd use a standard 3-tier architecture across two AZs minimum for HA.
> 
> VPC CIDR: `10.0.0.0/16` — gives 65,536 addresses.
> 
> Subnets across two AZs:
> 
> - Public: `10.0.1.0/24` (AZ-a), `10.0.2.0/24` (AZ-b) — for ALB, NAT Gateway
> - Private App: `10.0.10.0/24` (AZ-a), `10.0.11.0/24` (AZ-b) — for EC2/ECS
> - Private DB: `10.0.20.0/24` (AZ-a), `10.0.21.0/24` (AZ-b) — for RDS
> 
> Public subnets have a route to IGW. Private subnets route to NAT Gateway in the respective public subnet for outbound internet (package installs, API calls).
> 
> Security groups: ALB accepts 443 from 0.0.0.0/0. App servers accept traffic only from ALB security group. DB accepts traffic only from App security group. Layered security — no direct public access to app servers or DB."

---

---

# 7. Terraform (IaC)

## Concept

Terraform is Infrastructure as Code — you define your infrastructure in HCL (HashiCorp Configuration Language), and Terraform creates/updates/destroys it to match. The critical concept is **state**: Terraform tracks what it has created in a state file (`terraform.tfstate`). This is how it knows what to update vs. create vs. destroy.

**Workflow:**

```
Write HCL → terraform init → terraform plan → terraform apply → terraform destroy
```

---

## Core Syntax & Commands

```bash
terraform init           # Initialize: download providers, set up backend
terraform plan           # Preview changes (what will be created/updated/destroyed)
terraform apply          # Apply changes (prompts for confirmation)
terraform apply -auto-approve  # Skip confirmation (use in CI/CD)
terraform destroy        # Destroy all managed infrastructure
terraform fmt            # Format HCL files to canonical style
terraform validate       # Validate HCL syntax
terraform output         # Show output values
terraform state list     # List all resources in state
terraform state show aws_instance.web  # Show state for specific resource
terraform import aws_s3_bucket.my_bucket mybucket-name  # Import existing resource
```

**Provider:**

```hcl
terraform {
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"    # Allow 5.x.x but not 6.x.x
    }
  }
  backend "s3" {
    bucket  = "tf-state-aakash"
    key     = "prod/terraform.tfstate"
    region  = "ap-south-1"
    encrypt = true
  }
}

provider "aws" {
  region = var.aws_region
}
```

**Resource, Variable, Output:**

```hcl
# variables.tf
variable "instance_type" {
  type    = string
  default = "t3.micro"
}

# main.tf
resource "aws_instance" "web" {
  ami           = data.aws_ami.ubuntu.id
  instance_type = var.instance_type
  subnet_id     = aws_subnet.public.id

  tags = {
    Name        = "web-server"
    Environment = var.env
  }

  user_data = <<-EOF
    #!/bin/bash
    apt-get update -y
    apt-get install -y docker.io
    systemctl start docker
    systemctl enable docker
  EOF
}

# outputs.tf
output "web_public_ip" {
  value       = aws_instance.web.public_ip
  description = "Public IP of the web server"
}
```

**Remote State (S3 Backend) — Critical for team use:**

```hcl
backend "s3" {
  bucket         = "my-terraform-state"
  key            = "project/env/terraform.tfstate"
  region         = "ap-south-1"
  encrypt        = true
  dynamodb_table = "terraform-state-lock"  # Prevents concurrent applies
}
```

---

## Senior Interview Q&A

**Q: What is Terraform state and why is it important?**

> "Terraform state is the source of truth for what Terraform has deployed. It maps your HCL resource definitions to real infrastructure IDs — 'this `aws_instance.web` is EC2 instance `i-0abc12345`. '
> 
> Without state, Terraform would have to query every AWS resource on every plan to figure out what exists. That would be slow and AWS API rate-limited. State makes plans fast.
> 
> State also tracks dependencies — so if you delete a VPC resource, Terraform knows to delete its subnets first.
> 
> The critical concern: **state contains secrets in plaintext**. Database passwords, private keys — they're all in the state file. This is why you never commit state to Git. Store it in S3 with encryption enabled, and use DynamoDB for state locking (prevents two people running `terraform apply` simultaneously, which would corrupt state).
> 
> In practice at teams: S3 backend + DynamoDB lock is the minimum setup. Some teams use Terraform Cloud for state management with built-in locking and audit trails."

---

**Q: What's `terraform plan` doing under the hood?**

> "Plan compares three things: your HCL code (desired state), the Terraform state file (last known real state), and the actual infrastructure (refreshed from the provider API).
> 
> It runs a refresh first — calls AWS APIs to check if state-tracked resources still exist and match. If you manually deleted something in the console, plan detects the drift.
> 
> Then it diffs desired state vs. current state and produces a set of actions: create (resource in code, not in state), update (resource exists but attributes differ), destroy (resource in state, not in code), replace (forced replacement — like changing an AMI which requires EC2 to be destroyed and recreated — shown as `-/+`).
> 
> The `~` symbol means in-place update. The `-/+` means destroy-then-create. Always review `-/+` carefully — that means downtime or data loss risk.
> 
> I always run plan in CI before apply, and the plan output is attached to the PR for review. Nobody applies without a reviewed plan."

---

**Q: How do you handle sensitive values in Terraform?**

> "Three approaches, in order of preference:
> 
> 1. **Don't put secrets in Terraform at all.** For RDS passwords, use AWS Secrets Manager and pass the ARN to the resource. `password = data.aws_secretsmanager_secret_version.db.secret_string`. The secret never touches Terraform state.
>     
> 2. **Use `sensitive = true` on variables.** This prevents the value from appearing in `terraform plan` output. But note: it's still in the state file — just not shown in logs.
>     
> 3. **Pass secrets at runtime via environment variables.** `export TF_VAR_db_password=$SECRET`. The variable is in Terraform but never committed to source control.
>     
> 
> What I never do: put secrets in `terraform.tfvars` and commit that file to Git. I've seen teams do this and it's a major security incident waiting to happen. `.gitignore` must include `*.tfvars` files containing secrets."

---

---

# 8. CI/CD — GitHub Actions & Jenkins

## Concept

CI/CD is the backbone of DevOps. CI (Continuous Integration) automates build + test on every commit. CD (Continuous Delivery/Deployment) automates delivery to environments.

**Why it matters:** Without CI/CD, deployments are manual, risky, and infrequent. With CI/CD, deployments are automatic, tested, and happen multiple times a day.

---

## GitHub Actions

**Trigger on push to main, run tests, build Docker image, push to Docker Hub:**

```yaml
name: CI/CD Pipeline

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

env:
  DOCKER_IMAGE: aakash0908/flask-app

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-python@v5
        with:
          python-version: '3.11'
      - run: pip install -r requirements.txt
      - run: pytest --tb=short

  build-and-push:
    needs: test                         # Only runs if test passes — pipeline gate
    runs-on: ubuntu-latest
    if: github.ref == 'refs/heads/main' # Only on main branch, not PRs
    steps:
      - uses: actions/checkout@v4
      - uses: docker/login-action@v3
        with:
          username: ${{ secrets.DOCKERHUB_USERNAME }}
          password: ${{ secrets.DOCKERHUB_TOKEN }}
      - uses: docker/build-push-action@v5
        with:
          push: true
          tags: ${{ env.DOCKER_IMAGE }}:${{ github.sha }},${{ env.DOCKER_IMAGE }}:latest

  deploy:
    needs: build-and-push
    runs-on: ubuntu-latest
    steps:
      - uses: appleboy/ssh-action@v1.0.3
        with:
          host: ${{ secrets.EC2_HOST }}
          username: ubuntu
          key: ${{ secrets.EC2_SSH_KEY }}
          script: |
            docker pull ${{ env.DOCKER_IMAGE }}:latest
            docker stop flask-app || true
            docker rm flask-app || true
            docker run -d --name flask-app -p 80:5000 \
              --env-file /home/ubuntu/.env \
              ${{ env.DOCKER_IMAGE }}:latest
```

---

## Jenkins

**Declarative Pipeline:**

```groovy
pipeline {
    agent any

    environment {
        DOCKER_IMAGE = "aakash0908/flask-app:${BUILD_NUMBER}"
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/aakash-1004/flask-mongo-app.git'
            }
        }

        stage('Test') {
            steps {
                sh 'pip install -r requirements.txt'
                sh 'pytest --tb=short'
            }
        }

        stage('Build & Push') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'dockerhub-creds',
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )]) {
                    sh '''
                        echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin
                        docker build -t $DOCKER_IMAGE .
                        docker push $DOCKER_IMAGE
                    '''
                }
            }
        }

        stage('Deploy') {
            steps {
                sshagent(['ec2-ssh-key']) {
                    sh '''
                        ssh -o StrictHostKeyChecking=no ubuntu@${EC2_HOST} "
                            docker pull ${DOCKER_IMAGE} &&
                            docker stop flask-app || true &&
                            docker rm flask-app || true &&
                            docker run -d --name flask-app -p 80:5000 ${DOCKER_IMAGE}
                        "
                    '''
                }
            }
        }
    }

    post {
        failure { echo "Pipeline FAILED" }
        success { echo "Pipeline SUCCEEDED" }
    }
}
```

---

## Senior Interview Q&A

**Q: How do you handle secrets in a CI/CD pipeline?**

> "Several layers.
> 
> For GitHub Actions: GitHub Secrets (Settings → Secrets and variables → Actions). They're encrypted, not visible in logs (shown as `***`), and not passed to forked PRs by default — that last point is security-critical.
> 
> For Jenkins: Credentials store (Manage Jenkins → Credentials). Use `withCredentials` block in pipeline. Never echo secrets or set -x in the block.
> 
> For both: never hardcode secrets in the pipeline YAML/Jenkinsfile. Those files are in source control. Rotate secrets regularly. Use different secrets per environment.
> 
> Best practice I push for: replace static credentials with OIDC-based authentication where possible. GitHub Actions can assume an AWS IAM role via OIDC — no AWS keys stored as GitHub Secrets at all. The GitHub Actions runtime presents a JWT to AWS, AWS validates it and issues short-lived credentials. Zero long-lived secrets."

---

**Q: Your deployment pipeline takes 40 minutes. How do you optimize it?**

> "Audit each stage for parallelism and caching opportunities.
> 
> First: parallelize independent stages. Tests, linting, security scanning — run them concurrently. In GitHub Actions that's separate jobs with `needs: []`. In Jenkins it's `parallel {}` blocks. This alone can cut time in half.
> 
> Second: cache dependencies. In GitHub Actions: `actions/cache` for pip, npm, Maven. If dependencies haven't changed, restore from cache instead of reinstalling. Dependency install going from 5 minutes to 30 seconds is common.
> 
> Third: optimize Docker builds. Multi-stage builds, `.dockerignore` to exclude unnecessary files, and layer ordering so the expensive `pip install` layer is only rebuilt when `requirements.txt` changes.
> 
> Fourth: use self-hosted runners for builds. GitHub-hosted runners cold-start every job. A self-hosted runner on a powerful machine eliminates startup time and network latency for pulling Docker images.
> 
> Fifth: split test suites. If you have 5000 unit tests, split them into parallel shards. GitHub Actions matrix strategy is perfect for this.
> 
> I've taken a 40-minute pipeline to 8 minutes with parallelism + caching alone."

---

**Q: Difference between GitHub Actions and Jenkins — when would you choose each?**

> "GitHub Actions is the right choice for most teams today. It's cloud-native, zero infrastructure to maintain, deep GitHub integration — PRs get automatic status checks, you can gate merges on pipeline green. For teams already on GitHub, the path of least resistance.
> 
> Jenkins is the right choice when you need: on-premises builds (sensitive code can't go to cloud runners), complex enterprise integrations (Jira, Artifactory, LDAP), highly customized agents (specific hardware, GPUs, network access), or a mature organization that already has Jenkins infrastructure.
> 
> Jenkins overhead is real: you maintain the master node, plugins, security updates, agent provisioning. For a 10-person team, that's a part-time job. For a 1000-person enterprise with specialized needs, it pays for itself.
> 
> The modern compromise: GitHub Actions for most pipelines, self-hosted runners on your own infrastructure when you need control, still zero Jenkins maintenance."

---

---

# 9. GitOps & ArgoCD

## Concept

GitOps is an operating model where Git is the **single source of truth** for both application code and infrastructure state. All changes flow through Git (PR, review, merge). The actual deployment is automated by an agent running in the cluster that continuously reconciles cluster state to match what's in Git.

**The GitOps principle:**

- Desired state = what's in Git
- Actual state = what's in the cluster
- ArgoCD continuously detects drift (actual ≠ desired) and auto-syncs

**Why this matters:**

- Full audit trail: every change is a Git commit (who, what, when, why)
- Easy rollback: `git revert` + sync
- Self-healing: if someone does `kubectl edit` manually, ArgoCD reverts it
- No kubectl access needed for deployments — Git is the deploy interface

---

## Core ArgoCD Commands

```bash
# Installation on k3s
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml \
  --server-side --force-conflicts   # Needed for large CRDs

# Get admin password
kubectl get secret argocd-initial-admin-secret -n argocd \
  -o jsonpath='{.data.password}' | base64 -d

# Port forward to access UI
kubectl port-forward svc/argocd-server -n argocd 8080:443

# ArgoCD CLI
argocd login localhost:8080
argocd app list
argocd app sync myapp
argocd app status myapp
argocd app history myapp
argocd app rollback myapp 5
```

**ArgoCD Application manifest:**

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: fullstack-app
  namespace: argocd
spec:
  project: default
  source:
    repoURL: https://github.com/aakash-1004/fullstack-chart
    targetRevision: main
    path: fullstack-chart                # Path to Helm chart in repo
    helm:
      valueFiles:
        - values-prod.yaml
  destination:
    server: https://kubernetes.default.svc
    namespace: production
  syncPolicy:
    automated:
      prune: true      # Delete resources removed from Git
      selfHeal: true   # Revert manual changes in cluster
    syncOptions:
      - CreateNamespace=true
```

---

## Senior Interview Q&A

**Q: What is GitOps and how does it differ from traditional CI/CD?**

> "In traditional CI/CD, the pipeline has write access to the cluster — it runs `kubectl apply` or `helm upgrade` as part of the deploy stage. The pipeline is the deployment mechanism.
> 
> In GitOps, the pipeline never touches the cluster directly. Instead, it updates the Git repo (a 'config repo' or updates the image tag in a Helm values file). An agent running inside the cluster (ArgoCD, Flux) watches Git and reconciles the cluster state to match.
> 
> The inversion is significant: instead of push-based deployment (pipeline pushes to cluster), you have pull-based deployment (cluster agent pulls from Git).
> 
> Operational benefits: you can revoke all external CI/CD cluster access — nothing from outside needs to deploy. The cluster manages itself. Full audit trail in Git. Disaster recovery becomes `git clone + argocd sync` — you can recreate a cluster from Git state.
> 
> The trade-off: it adds complexity. You need to manage the separation between 'app code repo' and 'config repo', and the CI pipeline needs to update image tags in the config repo as part of its flow."

---

**Q: ArgoCD shows an application is OutOfSync. What does that mean and how do you handle it?**

> "OutOfSync means what ArgoCD sees in the cluster (actual state) doesn't match what's in Git (desired state).
> 
> This can happen three ways:
> 
> 1. Someone changed something directly in the cluster via `kubectl` or Helm — a manual change that bypassed Git.
> 2. You pushed a change to the Git config repo and ArgoCD hasn't synced yet (or auto-sync is off).
> 3. A resource was created outside ArgoCD's management and conflicts with a managed resource.
> 
> If auto-sync and selfHeal are enabled, ArgoCD will reconcile automatically within seconds.
> 
> If manual: click Sync in the UI or `argocd app sync myapp`. It applies the Git state to the cluster.
> 
> Investigation: `argocd app diff myapp` shows exactly what's different. This is your first step — understand the diff before you sync, to avoid overwriting something important.
> 
> In production: always have `selfHeal: true`. Any manual kubectl edit should be treated as a drift event, logged, and reverted. If you need to make an emergency change, make it in Git — even if it takes 5 minutes longer, you maintain auditability."

---

---

# 10. Monitoring — Prometheus & Grafana

## Concept

Observability has three pillars: metrics (Prometheus), logs (ELK/Loki), traces (Jaeger/Tempo). Prometheus + Grafana is the de facto Kubernetes monitoring stack.

**How Prometheus works:**

1. Prometheus **scrapes** metrics from targets (HTTP GET to `/metrics` endpoint)
2. Targets are discovered via static config, service discovery, or ServiceMonitor CRDs
3. Metrics are stored in a time-series database
4. PromQL is used to query and compute on those metrics
5. Alertmanager handles alert routing (PagerDuty, Slack, email)

**Metric types:**

- **Counter**: Only goes up. Total requests, total errors. Use `rate()` to get per-second rate.
- **Gauge**: Can go up or down. Memory usage, active connections, queue size.
- **Histogram**: Samples observations into buckets. Used for request latency distributions. Exposes `_bucket`, `_count`, `_sum`.
- **Summary**: Client-side percentiles. Similar to histogram but less flexible for aggregation.

---

## Key Commands & PromQL

```bash
# Helm installation (kube-prometheus-stack)
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update
helm install prometheus prometheus-community/kube-prometheus-stack \
  -n monitoring --create-namespace

# Access UIs
kubectl port-forward svc/prometheus-operated -n monitoring 9090:9090
kubectl port-forward svc/prometheus-grafana -n monitoring 3000:80
# Grafana default: admin / prom-operator

# Useful PromQL queries
rate(http_requests_total[5m])                          # Request rate over 5 minutes
rate(http_requests_total{status=~"5.."}[5m])           # 5xx error rate
histogram_quantile(0.99, rate(http_request_duration_seconds_bucket[5m]))  # p99 latency
container_memory_usage_bytes{namespace="production"}   # Memory per container
kube_pod_container_status_restarts_total               # Pod restart count (alert on this!)

# For JS7/JMX metrics (your specific setup):
java_lang_Memory_HeapMemoryUsage_used                 # JVM heap usage
js7_order_count_total                                  # JS7 order count
```

**ServiceMonitor (for scraping custom apps):**

```yaml
apiVersion: monitoring.coreos.com/v1
kind: ServiceMonitor
metadata:
  name: flask-monitor
  namespace: monitoring
  labels:
    release: prometheus     # Must match Helm release label selector
spec:
  namespaceSelector:
    matchNames:
      - production
  selector:
    matchLabels:
      app: flask-app
  endpoints:
    - port: http
      path: /metrics
      interval: 15s
```

---

## Senior Interview Q&A

**Q: What's the difference between a Counter and a Gauge? Give examples.**

> "A Counter only ever increases — it counts events. HTTP request count, error count, bytes sent. It resets to zero only on process restart. You never query a raw counter value in PromQL — you use `rate()` to get the per-second rate of increase, or `increase()` for total increase over a range.
> 
> A Gauge can go up or down — it represents a current state. CPU utilization, memory usage, active connections, number of pods running, queue depth. You query gauges directly.
> 
> Why the distinction matters: if your metric counts total requests since startup (counter), and you want requests per minute, you need `rate(http_requests_total[1m]) * 60`. If you mistakenly modeled this as a gauge and reset it each minute, you lose the ability to aggregate across multiple replicas and you have inconsistent behavior across restarts.
> 
> Common mistake: modeling something as a gauge when it should be a counter. If it's a thing that's happened (event), it's a counter. If it's a thing that is (state), it's a gauge."

---

**Q: How do you set up alerting? Walk through the full chain.**

> "The chain: metric condition → Prometheus alert rule → Alertmanager → notification channel.
> 
> In the PrometheusRule CRD:
> 
> ```yaml
> - alert: HighErrorRate
>   expr: rate(http_requests_total{status=~'5..'}[5m]) > 0.1
>   for: 5m            # Must be true for 5 minutes before firing
>   labels:
>     severity: critical
>   annotations:
>     summary: 'High 5xx error rate on {{ $labels.service }}'
> ```
> 
> Prometheus evaluates this every `evaluationInterval` (default: 15s). If the expression is true for the `for` duration, the alert fires to Alertmanager.
> 
> Alertmanager handles routing: which alert goes to which team (based on labels), grouping (batch 50 alerts into one Slack message instead of 50), silencing (suppress alerts during planned maintenance), and inhibition (if the cluster is down, suppress all service alerts).
> 
> The `for` duration is critical. Without it, a single spike fires the alert. 5 minutes of sustained high error rate is a real problem; a 10-second spike might be a batch job."

---

**Q: Your Prometheus targets show a service as DOWN. How do you debug?**

> "Check the Prometheus targets page (`/targets` in the UI) — it shows the exact error: connection refused, 404, no route to host.
> 
> Connection refused: the app is not running on the scraped port. Check if the pod is up and the port is correct in the ServiceMonitor.
> 
> No route to host / DNS resolution failure: the service name in the scrape config doesn't resolve. Check the ServiceMonitor selector labels match the Service labels.
> 
> 403 Forbidden: the app has auth on `/metrics`. Either remove auth on that path or configure Prometheus with the right auth headers.
> 
> 404 Not Found: the app doesn't expose `/metrics` at that path. Either it's not instrumented (needs the prometheus client library added) or the path is different.
> 
> For Flask apps: you need `prometheus_flask_exporter` and `PrometheusMetrics(app)` initialized. Then the `/metrics` endpoint exists automatically.
> 
> The most common issue I see: ServiceMonitor `release` label doesn't match the Prometheus Helm release name. kube-prometheus-stack looks for ServiceMonitors with `release: prometheus` by default — if yours has `release: myrelease`, it's ignored silently."

---

---

# 11. Deployment — Nginx, Gunicorn, systemd

## Concept

Before Kubernetes, and still in many production setups, apps are deployed directly on Linux servers. Understanding traditional deployment is essential for debugging, legacy environments, and when you don't need Kubernetes complexity.

**The stack:**

```
Internet → Nginx (reverse proxy, SSL termination, static files) 
         → Gunicorn (Python WSGI server, manages worker processes)
         → Flask app code
```

**Why not run Flask directly?** Flask's dev server is single-threaded and not production-safe. Gunicorn manages multiple worker processes to handle concurrent requests. Nginx sits in front because it's much better at SSL termination, static file serving, rate limiting, and handling slow clients.

---

## Core Commands

```bash
# Nginx
sudo apt-get install nginx
sudo systemctl start nginx
sudo systemctl enable nginx             # Start on boot
sudo nginx -t                           # Test config syntax
sudo systemctl reload nginx             # Reload config without downtime

# Nginx config location
/etc/nginx/nginx.conf                   # Main config
/etc/nginx/sites-available/myapp        # Per-site config
/etc/nginx/sites-enabled/               # Symlink enabled configs

# Gunicorn
pip install gunicorn
gunicorn --workers 4 --bind 0.0.0.0:5000 app:app          # 4 workers
gunicorn --workers 4 --bind unix:/tmp/gunicorn.sock app:app # Unix socket

# systemd service file: /etc/systemd/system/myapp.service
sudo systemctl daemon-reload            # After creating/editing service file
sudo systemctl enable myapp             # Start on boot
sudo systemctl start myapp
sudo journalctl -u myapp -f             # Follow service logs
```

**Nginx reverse proxy config:**

```nginx
server {
    listen 80;
    server_name yourdomain.com;

    location / {
        proxy_pass http://127.0.0.1:5000;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    }

    location /static {
        alias /var/www/myapp/static;    # Serve static files directly (faster)
    }
}
```

**systemd service:**

```ini
[Unit]
Description=Flask App
After=network.target

[Service]
User=ubuntu
WorkingDirectory=/home/ubuntu/myapp
Environment="PATH=/home/ubuntu/myapp/venv/bin"
EnvironmentFile=/home/ubuntu/myapp/.env
ExecStart=/home/ubuntu/myapp/venv/bin/gunicorn --workers 4 --bind 0.0.0.0:5000 app:app
Restart=always

[Install]
WantedBy=multi-user.target
```

---

## Senior Interview Q&A

**Q: Why do you put Nginx in front of Gunicorn? Why not expose Gunicorn directly?**

> "Four reasons.
> 
> First, SSL termination. Nginx handles HTTPS with minimal config. Offloading SSL from Gunicorn means your Python workers only handle HTTP internally — no crypto overhead.
> 
> Second, static file serving. Nginx serves static files (JS, CSS, images) in C at wire speed — far faster than Python. Gunicorn workers are expensive (memory, CPU). You don't want them serving a 50KB CSS file that Nginx can handle in microseconds.
> 
> Third, slow client protection. A slow HTTP client (mobile on 3G) can tie up a Gunicorn worker for seconds waiting for the request to arrive. Nginx buffers the full request before handing it to Gunicorn, so your workers are never waiting on slow clients.
> 
> Fourth, connection concurrency. Nginx handles thousands of simultaneous connections in a single process (event-driven). Gunicorn workers are one-connection-at-a-time per worker. Nginx sits at the edge, Gunicorn handles the compute.
> 
> Modern alternative: in cloud environments with a load balancer, SSL is often terminated at the LB. But the static file and slow client arguments still apply."

---

---

# 12. System Design for DevOps

## The Core Patterns

**High Availability:**

- Eliminate single points of failure: everything critical has at least 2 instances across 2 AZs
- Load balancing: distribute traffic across instances
- Health checks: route traffic only to healthy instances
- Auto-scaling: handle load spikes automatically

**Deployment Strategies:**

- **Recreate**: Stop all old, start all new. Downtime. Never in production.
- **Rolling update**: Replace pods one by one. Default in Kubernetes Deployments. Zero downtime, brief period of mixed versions.
- **Blue/Green**: Two identical environments. Switch traffic instantly. Easy rollback (switch back). Cost: double infrastructure.
- **Canary**: Route small % of traffic to new version. Validate before full rollout. Best risk management, most complex.

**The 12-Factor App:**

1. Codebase — one repo per app
2. Dependencies — declare explicitly (requirements.txt, package.json)
3. Config — store in environment (never in code)
4. Backing services — treat as attached resources (DB URL in config, not hardcoded)
5. Build/Release/Run — strict separation
6. Processes — stateless (no local filesystem state)
7. Port binding — self-contained HTTP server
8. Concurrency — scale out via processes
9. Disposability — fast startup, graceful shutdown
10. Dev/Prod parity — keep environments as similar as possible
11. Logs — treat as event streams (stdout, not files)
12. Admin processes — run as one-off processes

---

## Senior Interview Q&A

**Q: Design a CI/CD pipeline for a microservices application. Walk me through the architecture.**

> "For a microservices app, I'd design around these principles: each service has its own pipeline, independent deployability, and central observability.
> 
> Source: each microservice in its own directory in a monorepo, or separate repos. PRs trigger the respective service pipeline.
> 
> CI pipeline per service: code checkout → dependency install → unit tests → integration tests (against test DB/services) → static analysis/security scan → Docker build → image pushed to ECR/Docker Hub with `git sha` tag.
> 
> CD pipeline: pipeline updates the Helm values file in a 'config repo' with the new image tag, commits and pushes. ArgoCD detects the Git change, syncs the Kubernetes cluster.
> 
> The separation is critical: the app code repo and the deployment config repo are separate. CI owns app code. ArgoCD owns cluster state. This means a cluster outage doesn't block CI, and a failed deploy doesn't affect other services.
> 
> For staging: ArgoCD auto-syncs. For production: ArgoCD waits for manual sync approval (a PR approval in the config repo triggers sync).
> 
> Observability layer: Prometheus scrapes all services, centralized Grafana, alerts route to PagerDuty for on-call. Logs flow to ELK or CloudWatch.
> 
> One thing I always add: a smoke test job after deploy — runs a few key API calls against the deployed service to verify it's actually serving traffic correctly before marking the deploy complete."

---

**Q: Your application is running slow. How do you systematically debug it?**

> "The USE method for infrastructure + RED method for services.
> 
> **USE (for every resource):** Utilization, Saturation, Errors. Is CPU at 90% (high utilization)? Is the request queue growing (saturation)? Are there errors in the logs?
> 
> **RED (for every service):** Rate (requests per second), Errors (error rate), Duration (latency). These three metrics tell you the health of any service.
> 
> Drill down approach:
> 
> - Metrics first: Grafana dashboard. Where is CPU/memory high? Which service has high error rate or high latency?
> - Identify the bottleneck service. In a request trace: user → API gateway → service A → service B → DB. Which hop is slow?
> - If it's DB: slow query log. Missing index? N+1 query? Table scan?
> - If it's the app: profiling. Python cProfile, Java async-profiler. Where is the CPU time being spent?
> - If it's external: network latency to a third-party API? Add caching.
> - If it's Kubernetes: is the pod being throttled (CPU throttle)? Is it hitting memory limits and triggering GC constantly?
> 
> The mistake people make: jumping to solutions before finding the actual bottleneck. Profile first, optimize second."

---

---

# QUICK REFERENCE — INTERVIEW CHEAT SHEET

## "Tell me about yourself" — DevOps Frame

> "I'm a Cloud and Automation Engineer with two years of production experience. I support JS7 JobScheduler systems, handle Keycloak/OIDC authentication issues, and do Linux administration and troubleshooting for enterprise clients. On the DevOps side, I've built end-to-end pipelines — Flask microservices containerized with Docker, deployed on Kubernetes with k3s, infrastructure provisioned with Terraform, CI/CD via GitHub Actions, GitOps with ArgoCD, and monitoring with Prometheus and Grafana. I'm also progressing toward AWS Solutions Architect Associate certification. I'm looking for a DevOps engineering role where I can own the full delivery pipeline."

---

## Must-Know Numbers

|Item|Value to Know|
|---|---|
|EC2 free tier|t3.micro, 750 hours/month|
|S3 max object size|5TB|
|S3 multipart upload required after|5GB|
|Default kubeconfig location|`~/.kube/config`|
|Prometheus default scrape interval|15s|
|Docker bridge network default subnet|172.17.0.0/16|
|etcd port|2379 (client), 2380 (peer)|
|kube-apiserver port|6443|
|kubelet port|10250|
|Grafana default port|3000|

---

## Your Projects — Quick Reference for Interviews

|Project|What it demonstrates|
|---|---|
|`flask-mongo-app`|Docker, GitHub Actions CI/CD, EC2 deployment|
|`fullstack-docker`|Multi-container, Docker Compose, networking|
|`k3s-fullstack`|Kubernetes, ConfigMap, Secrets, Service|
|`fullstack-chart`|Helm, ArgoCD, GitOps, full k8s stack|
|`terraform-aws`|Terraform, VPC, EC2, ECS Fargate, ALB|
|`taskmanager`|Flask REST API, MongoDB, Prometheus metrics|

---

## Session Summary

**Covered:** Full DevOps stack revision — Fundamentals, Linux/Bash, Git, Docker, Kubernetes, AWS, Terraform, CI/CD, GitOps, Monitoring, Deployment, System Design **Interview Q&A added:** Senior Tech Lead level — production scenarios, architecture decisions, debugging approaches **Next:** Practice speaking these answers out loud. Record yourself. The knowledge is there — fluency under pressure is the gap to close. **Progress Note:** Comprehensive revision complete — all major DevOps topics consolidated with senior-level interview Q&A.

> ⚠️ **Security reminder:** Never store AWS keys, GitHub PATs, Docker tokens, or DB passwords in Obsidian notes or plain text files. Always use your credentials manager or AWS Secrets Manager.