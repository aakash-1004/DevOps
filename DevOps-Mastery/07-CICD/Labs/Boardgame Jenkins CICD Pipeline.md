---

## tags: [jenkins, kubeadm, kubernetes, rbac, sonarqube, nexus, trivy, prometheus, grafana, blackbox-exporter, cicd, devops, notes] status: completed date: 2026-08-25 to 2026-08-28

# Boardgame Jenkins CI/CD Pipeline — Complete Step-by-Step Notes

Everything that was done, in chronological order, no step omitted regardless of how small.

---

## PHASE 0 — Planning & Infrastructure Decisions

### Server topology decided

7 dedicated EC2 servers, deliberately separated rather than crammed onto one box:

|#|Server|Instance Type|Why this size|
|---|---|---|---|
|1|Jenkins + Docker + Trivy|m7i-flex.large (8GB/2vCPU)|Builds are memory-hungry; originally sized for all 3 tools combined, now dedicated to just Jenkins — generous headroom|
|2|Nexus|c7i-flex.large (4GB/2vCPU)|Nexus is lighter than SonarQube; 4GB is comfortable for a lab artifact repo|
|3|SonarQube + Postgres|m7i-flex.large (8GB/2vCPU)|SonarQube runs embedded Elasticsearch — genuinely RAM-hungry, wants 4GB+ realistically|
|4|K8s master|c7i-flex.large (4GB/2vCPU)|Runs etcd + API server + controller-manager + scheduler together; kubeadm's minimum is 2GB but that's too tight for real use|
|5|K8s worker 1 (slave-1)|t3.small (2GB/2vCPU)|Workers only run kubelet + kube-proxy + CNI + app pods — meets kubeadm minimum comfortably|
|6|K8s worker 2 (slave-2)|t3.small (2GB/2vCPU)|Same as slave-1|
|7|Monitoring (Prometheus + Grafana + Blackbox)|c7i-flex.large (4GB/2vCPU)|Prometheus's time-series DB grows with scrape targets; needs real headroom|

### Why separate servers, not all-in-one

Docker containers give process isolation but NOT resource isolation by default. Without explicit `--memory`/`--cpus` caps, every container shares the same CPU/RAM pool with zero enforced boundaries. SonarQube's embedded Elasticsearch commonly spikes to 2GB+ during scans and can starve Jenkins or Nexus via the OS OOM-killer. Real companies isolate these tools onto separate infrastructure.

### Why kubeadm, not EKS

Deliberate choice — already knew EKS from a previous project, wanted to understand what a managed control plane actually abstracts away (dockershim removal, CNI bootstrapping, kernel prerequisites, RBAC token generation) rather than only knowing the managed-service version.

### Key infrastructure choices made upfront

- All instances use Ubuntu 22.04 (Jammy) AMI
- All share the same security group (`myWebServer`, sg-09c50e5f8249e552e)
- All use the same SSH key pair (`myWebServer`)
- Region: ap-south-1 (Mumbai)
- All instance types verified as Free Tier eligible on this specific AWS account

---

## PHASE 1A — Kubernetes Cluster: EC2 Provisioning

### Launched 3 instances for the K8s cluster

```bash
# Master node
aws ec2 run-instances \
  --region ap-south-1 \
  --image-id ami-0aa761682283b4cc8 \
  --instance-type c7i-flex.large \
  --key-name myWebServer \
  --security-group-ids sg-09c50e5f8249e552e \
  --block-device-mappings '[{"DeviceName":"/dev/sda1","Ebs":{"VolumeSize":20,"VolumeType":"gp3"}}]' \
  --tag-specifications 'ResourceType=instance,Tags=[{Key=Name,Value=k8s-master}]' \
  --query "Instances[0].InstanceId" --output text
# Result: i-04e788d52fb81aef9

# Slave-1
aws ec2 run-instances \
  --region ap-south-1 \
  --image-id ami-0aa761682283b4cc8 \
  --instance-type t3.small \
  --key-name myWebServer \
  --security-group-ids sg-09c50e5f8249e552e \
  --block-device-mappings '[{"DeviceName":"/dev/sda1","Ebs":{"VolumeSize":20,"VolumeType":"gp3"}}]' \
  --tag-specifications 'ResourceType=instance,Tags=[{Key=Name,Value=k8s-slave-1}]' \
  --query "Instances[0].InstanceId" --output text
# Result: i-00dd691370aa08e55

# Slave-2
aws ec2 run-instances \
  --region ap-south-1 \
  --image-id ami-0aa761682283b4cc8 \
  --instance-type t3.small \
  --key-name myWebServer \
  --security-group-ids sg-09c50e5f8249e552e \
  --block-device-mappings '[{"DeviceName":"/dev/sda1","Ebs":{"VolumeSize":20,"VolumeType":"gp3"}}]' \
  --tag-specifications 'ResourceType=instance,Tags=[{Key=Name,Value=k8s-slave-2}]' \
  --query "Instances[0].InstanceId" --output text
# Result: i-03f2096f48784f669
```

Why 20GB root volumes: default is often 8GB, too small for Docker images + system packages + kubeadm components combined.

### Added self-referencing security group rule for inter-node traffic

```bash
aws ec2 authorize-security-group-ingress \
  --region ap-south-1 \
  --group-id sg-09c50e5f8249e552e \
  --protocol all \
  --source-group sg-09c50e5f8249e552e
```

What this does: allows ALL traffic between any two instances sharing this same security group, on any port/protocol. This covers kubelet (10250), etcd (2379-2380), Calico CNI ports, and any other inter-node communication — without manually enumerating every port individually. Does NOT affect inbound rules from the internet (those remain scoped to specific ports like 22, 80, 3000-10000 as already configured).

### Fetched all instance IPs

```bash
aws ec2 describe-instances --region ap-south-1 \
  --instance-ids i-04e788d52fb81aef9 i-00dd691370aa08e55 i-03f2096f48784f669 \
  --query "Reservations[].Instances[].{Name:Tags[?Key=='Name']|[0].Value,State:State.Name,PublicIP:PublicIpAddress,PrivateIP:PrivateIpAddress}" \
  --output table
```

Result:

- k8s-master: private 172.31.33.140, public 13.232.63.245
- k8s-slave-1: private 172.31.6.84, public 13.204.67.146
- k8s-slave-2: private 172.31.7.252, public 13.203.77.37

Why both IPs matter: SSH in using public IPs; kubeadm cluster communication uses private IPs (traffic stays inside AWS's internal network, faster and more secure).

---

## PHASE 1B — Kubernetes Cluster: Node Setup (All 3 Nodes)

### Script created and run on all 3 nodes identically

```bash
cat > k8s-node-setup.sh << 'EOF'
#!/bin/bash
set -e

echo "=== 1. Update system packages ==="
sudo apt-get update

echo "=== 2. Install Docker ==="
sudo apt install docker.io -y
sudo chmod 666 /var/run/docker.sock

echo "=== 3. Install cri-dockerd (CRI shim, required since K8s 1.24+ removed dockershim) ==="
VER=$(curl -s https://api.github.com/repos/Mirantis/cri-dockerd/releases/latest | grep '"tag_name"' | cut -d '"' -f4 | sed 's/^v//')
wget https://github.com/Mirantis/cri-dockerd/releases/download/v${VER}/cri-dockerd_${VER}.3-0.ubuntu-jammy_amd64.deb
sudo dpkg -i cri-dockerd_${VER}.3-0.ubuntu-jammy_amd64.deb

echo "=== 4. Install Kubernetes apt dependencies ==="
sudo apt-get install -y apt-transport-https ca-certificates curl gnupg
sudo mkdir -p -m 755 /etc/apt/keyrings

echo "=== 5. Add Kubernetes repo and GPG key ==="
curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.28/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg
echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.28/deb/ /' | sudo tee /etc/apt/sources.list.d/kubernetes.list

echo "=== 6. Refresh package list ==="
sudo apt update

echo "=== 7. Install kubeadm, kubelet, kubectl (version-pinned) ==="
sudo apt install -y kubeadm=1.28.1-1.1 kubelet=1.28.1-1.1 kubectl=1.28.1-1.1
sudo apt-mark hold kubeadm kubelet kubectl

echo "✅ Node setup complete."
EOF
chmod +x k8s-node-setup.sh
./k8s-node-setup.sh
```

Why cri-dockerd is needed: Kubernetes removed `dockershim` (its built-in Docker integration) starting v1.24. Docker Engine alone is NOT CRI-compliant — kubelet can't talk to it directly anymore. `cri-dockerd` bridges that gap, implementing the CRI standard for Docker. Without it, `kubeadm init` fails its preflight check ("no valid CRI socket found"). The --cri-socket flag must be passed to every kubeadm init/join command.

Why apt-mark hold: prevents a routine `apt upgrade` from silently bumping kubeadm/kubelet/kubectl to a newer, potentially incompatible version — these three must stay version-matched with each other and with what the cluster was initialized with.

### Issue hit: MobaXterm session mix-up

When running on "slave-2," `hostname -I` actually showed slave-1's IP (172.31.6.84) — the MobaXterm saved session was duplicated from slave-1 without updating the IP. Fixed by creating a brand-new SSH session and verifying with `hostname -I` + `curl ifconfig.me` before running any commands. Lesson: always verify which server you're actually on before running join/init commands.

---

## PHASE 1C — Kernel Prerequisites (All 3 Nodes)

### Initially skipped these steps — then hit the exact failure they prevent

```
error execution phase preflight: [preflight] Some fatal errors occurred:
    [ERROR FileContent--proc-sys-net-bridge-bridge-nf-call-iptables]: /proc/sys/net/bridge/bridge-nf-call-iptables does not exist
```

### Applied on all 3 nodes

```bash
sudo tee /etc/modules-load.d/k8s.conf <<EOF
overlay
br_netfilter
EOF

sudo modprobe overlay
sudo modprobe br_netfilter

sudo tee /etc/sysctl.d/k8s.conf <<EOF
net.bridge.bridge-nf-call-iptables  = 1
net.bridge.bridge-nf-call-ip6tables = 1
net.ipv4.ip_forward                 = 1
EOF

sudo sysctl --system
```

Why overlay: the overlay filesystem driver — container runtimes (containerd/Docker) use this for efficient, layered container image storage.

Why br_netfilter: allows iptables to see and filter bridged network traffic. Kubernetes' pod networking relies on bridge networking + iptables rules for service routing; without this module, pod-to-pod traffic across the bridge wouldn't be visible to netfilter, breaking Service routing entirely.

Why ip_forward: enables the node to forward IP packets between interfaces — essential since a K8s node routes traffic between pods, and between pods and the outside world.

Why these must be on ALL 3 nodes, not just master: every node runs pod networking, not just the control plane.

---

## PHASE 1D — Kubernetes Master Initialization

### kubeadm init on master only

```bash
sudo kubeadm init --pod-network-cidr=10.244.0.0/16 --cri-socket unix:///var/run/cri-dockerd.sock
```

Why --pod-network-cidr: defines the IP range pods will use cluster-wide. 10.244.0.0/16 is the conventional default for Flannel-style CNI plugins; Calico also accepts it.

Why --cri-socket: explicitly tells kubeadm where to find the CRI endpoint — needed because Docker's default socket (/var/run/docker.sock) is NOT a CRI socket; cri-dockerd exposes its own at /var/run/cri-dockerd.sock.

### Configure kubectl access

```bash
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

Why: kubeadm generates a cluster-admin kubeconfig at /etc/kubernetes/admin.conf (root-owned). Copying it to your home directory with correct ownership lets your regular user run kubectl without sudo.

### Deploy Calico CNI

```bash
kubectl apply -f https://docs.projectcalico.org/manifests/calico.yaml
```

Why Calico specifically: it's the CNI plugin this course uses. Handles pod IP assignment, inter-node pod routing, and network policy enforcement. Without a CNI plugin, nodes stay NotReady and pods can't communicate.

### Deploy nginx Ingress Controller — hit a version-mismatch bug

Original command from the course docs:

```bash
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v0.49.0/deploy/static/provider/baremetal/deploy.yaml
```

This caused a CrashLoopBackOff — diagnosed via:

```bash
kubectl logs -n ingress-nginx deployment/ingress-nginx-controller --tail=50
```

Root cause found in logs:

```
Failed to watch *v1beta1.Ingress: failed to list *v1beta1.Ingress: the server could not find the requested resource
```

The controller version (v0.49.0, from ~2021) was hardcoded to use the `networking.k8s.io/v1beta1` Ingress API, which was fully removed in Kubernetes 1.22 — our cluster runs 1.28, six versions past removal.

Fixed with a current version:

```bash
kubectl delete -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v0.49.0/deploy/static/provider/baremetal/deploy.yaml
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.9.6/deploy/static/provider/baremetal/deploy.yaml
```

---

## PHASE 1E — Worker Nodes Join

### Generate join command on master

```bash
kubeadm token create --print-join-command
```

### Run on each worker (slave-1 and slave-2), with --cri-socket appended

```bash
sudo kubeadm join 172.31.33.140:6443 --token <token> --discovery-token-ca-cert-hash sha256:<hash> --cri-socket unix:///var/run/cri-dockerd.sock
```

Why --cri-socket is needed here too: same reason as master init — workers also need to know where the CRI endpoint is, since Docker itself doesn't expose one.

### Verified on master

```bash
kubectl get nodes
```

Result: all 3 nodes Ready (master = control-plane role, workers = <none> role).

### Note: kubectl only works on the master

Running kubectl on a worker gives "connection refused" — workers don't have the kubeconfig file pointing at the API server. kubectl is a client tool for the API server, which only runs on the control plane. Normal, expected architecture.

---

## PHASE 1F — Security Auditing

### kubeaudit (cluster configuration scanner)

```bash
VER=$(curl -s https://api.github.com/repos/Shopify/kubeaudit/releases/latest | grep '"tag_name"' | cut -d '"' -f4)
curl -L -o kubeaudit.tar.gz "https://github.com/Shopify/kubeaudit/releases/download/${VER}/kubeaudit_${VER#v}_linux_amd64.tar.gz"
tar -xvzf kubeaudit.tar.gz
sudo mv kubeaudit /usr/local/bin/
kubeaudit all
```

Findings grouped by actionability:

- Fixable: missing NetworkPolicies, missing resource limits, missing seccomp/AppArmor, runAsNonRoot not set
- Expected/not fixable: PrivilegedTrue + hostNetwork on Calico and kube-proxy (they genuinely need this to function — manipulating host iptables/network interfaces is their job)

kubeaudit's own banner flagged itself as deprecated (planned deprecation Oct 2024), recommending kube-bench instead.

### kube-bench (CIS Kubernetes Benchmark)

```bash
VER=$(curl -s https://api.github.com/repos/aquasecurity/kube-bench/releases/latest | grep '"tag_name"' | cut -d '"' -f4 | sed 's/^v//')
curl -L -o kube-bench.tar.gz "https://github.com/aquasecurity/kube-bench/releases/download/v${VER}/kube-bench_${VER}_linux_amd64.tar.gz"
tar -xvf kube-bench.tar.gz
sudo mv kube-bench /usr/local/bin/
```

First run had a subtle bug: running with `sudo` meant root's environment was used, which has no access to ~/.kube/config — kube-bench couldn't detect the real cluster version and silently fell back to testing against Kubernetes 1.18 controls (from 2020), producing a false PodSecurityPolicy finding for a feature removed in 1.25.

Corrected invocation:

```bash
sudo -E kube-bench run --targets master --version 1.28 --config-dir "$(pwd)/cfg" --config "$(pwd)/cfg/config.yaml"
```

Real results (against correct 1.28 benchmark): 38 PASS, 9 FAIL, 12 WARN.

Key FAILs identified:

- No audit logging configured at all (1.2.16-1.2.19)
- --profiling not disabled on API server, controller-manager, scheduler (1.2.15, 1.3.2, 1.4.1)
- --kubelet-certificate-authority not set (1.2.5)
- etcd data directory ownership (1.1.12)

---

## PHASE 2 — Jenkins Server Setup

### Jenkins instance was already provisioned earlier (m7i-flex.large)

Started it after overnight stop:

```bash
aws ec2 start-instances --region ap-south-1 --instance-ids i-0c120d94e908a53f2
```

### Full setup script (final corrected version, incorporating all bugs fixed during the session)

```bash
#!/bin/bash
set -e

# 0. Clean stale repo registrations from any prior attempts
sudo rm -f /etc/apt/sources.list.d/jenkins.list
sudo rm -f /usr/share/keyrings/jenkins-keyring.asc

# 1. Update system
sudo apt-get update
sudo apt-get upgrade -y

# 2. Remove docker.io if present (avoid conflict with official Docker repo)
sudo systemctl stop docker 2>/dev/null || true
sudo apt-get purge -y docker.io docker-doc docker-compose podman-docker containerd runc 2>/dev/null || true
sudo apt-get autoremove -y --purge

# 3. Install Docker from official repo (not docker.io)
sudo apt-get install -y ca-certificates curl
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu \
  $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt-get update
sudo apt-get install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin

# 4. Install Java 21 (Jenkins current LTS requires 21, NOT 17)
sudo apt install -y fontconfig openjdk-21-jre

# 5. Add Jenkins repo key (2026 key — rotated Dec 2025) and repo
sudo wget -O /usr/share/keyrings/jenkins-keyring.asc https://pkg.jenkins.io/debian-stable/jenkins.io-2026.key
echo "deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] https://pkg.jenkins.io/debian-stable binary/" | sudo tee /etc/apt/sources.list.d/jenkins.list > /dev/null

# 6. Install Jenkins
sudo apt-get update
sudo apt-get install -y jenkins

# 7. Grant jenkins user Docker access (persistent, scoped — NOT chmod 666)
sudo usermod -aG docker jenkins

# 8. Start and enable Jenkins
sudo systemctl start jenkins
sudo systemctl enable jenkins
sudo systemctl restart jenkins

# 9. Install Trivy
sudo apt-get install -y wget apt-transport-https gnupg lsb-release
wget -qO - https://aquasecurity.github.io/trivy-repo/deb/public.key | sudo gpg --dearmor -o /usr/share/keyrings/trivy.gpg
echo "deb [signed-by=/usr/share/keyrings/trivy.gpg] https://aquasecurity.github.io/trivy-repo/deb $(lsb_release -sc) main" | sudo tee -a /etc/apt/sources.list.d/trivy.list
sudo apt-get update
sudo apt-get install -y trivy

echo "✅ Jenkins + Docker (official) + Java 21 + Trivy setup complete."
echo "Jenkins initial admin password:"
sudo cat /var/lib/jenkins/secrets/initialAdminPassword
```

### Bugs hit and fixed during Jenkins setup

1. **Stale Jenkins GPG key (jenkins.io-2023.key):** Jenkins rotated their signing key in Dec 2025. The old key URL from tutorials fails with `NO_PUBKEY 7198F4B714ABFC68`. Fixed by using the 2026 key URL.
    
2. **Java 17 vs Java 21 requirement:** Jenkins installed fine with Java 17, apt reported success, but the service immediately crash-looped (`status=1/FAILURE`, CPU: 79ms — barely ran at all). Root cause only visible in journalctl — Jenkins now requires Java 21 minimum. Silently fatal, no clear error in `systemctl status` output alone.
    
3. **Docker socket permissions:** tutorials commonly use `chmod 666 /var/run/docker.sock` which is world-writable (any user on the box gets Docker = effectively root access), and doesn't survive Docker daemon restarts. Proper fix: `usermod -aG docker jenkins` — scoped to just the jenkins user, persistent across restarts.
    
4. **Docker.io vs Docker official repo:** `docker.io` is Ubuntu's repackaged, often-outdated build. The official repo provides `docker-ce` (current), plus `docker-buildx-plugin` and `docker-compose-plugin` which `docker.io` doesn't include at all.
    
5. **Stale repo file from prior failed attempt:** a leftover `jenkins.list` (with the old broken key reference) from an earlier attempt caused `apt-get update` to fail at step 1, before the script's own key fix at step 5 ever got a chance to run. Fixed by adding a cleanup step (step 0) that removes any stale repo/key files before starting.
    

### Install kubectl on Jenkins server

```bash
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl
kubectl version --client
```

Why needed: the `withKubeConfig` step in the Jenkinsfile needs the actual `kubectl` binary present on the Jenkins server — the plugin just injects credentials, it doesn't bundle kubectl itself.

### Jenkins initial configuration (via browser)

Accessed at `http://<jenkins-public-ip>:8080`

- Logged in with initial admin password from `sudo cat /var/lib/jenkins/secrets/initialAdminPassword`
- Installed suggested plugins
- Created admin user

### Jenkins Global Tool Configuration (Manage Jenkins → Tools)

- JDK: name `jdk17`, auto-install
- Maven: name `maven3`, auto-install
- SonarQube Scanner: name `sonar-scanner`, auto-install
- Docker: name `docker`, Install automatically UNCHECKED, Installation root: `/usr` (points at the system's already-installed docker-ce, not a stale auto-downloaded binary)

Why Docker tool path matters: with "Install automatically" checked, Jenkins auto-downloads its own ancient Docker CLI from a legacy URL (`get.docker.com/builds/...`, API v1.29) — incompatible with the modern daemon (needs API 1.40+). Setting Installation root to `/usr` makes Jenkins use `/usr/bin/docker` (the real, modern docker-ce binary) instead.

### Jenkins Credentials (Manage Jenkins → Credentials → Global)

Created these credentials (all Kind: Username with password, except where noted):

- `git-cred` — GitHub username (aakash-1004) + Personal Access Token (scope: repo)
- `docker-cred` — Docker Hub username (aakash0908) + Docker Hub Access Token
- `sonar-token` — Kind: Secret text — SonarQube token (generated from SonarQube UI → Administration → Security → Users → Tokens)
- `k8s-cred` — Kind: Secret text — Kubernetes ServiceAccount token (from the RBAC secret we create later)

### Jenkins System Configuration (Manage Jenkins → System)

SonarQube servers:

- Name: `sonar`
- URL: `http://<sonarqube-ip>:9000`
- Authentication token: `sonar-token` credential

Extended E-mail Notification:

- SMTP server: `smtp.gmail.com`
- SMTP port: `465` (NOT 25 — port 25 doesn't support SSL, and AWS blocks outbound port 25 by default)
- Use SSL: checked
- SMTP Authentication: username `aakashrao0908@gmail.com`, password: Google App Password (not the real Gmail account password)

### Jenkins Managed Files (Manage Jenkins → Managed files)

Added: Global Maven Settings (`global-settings`) Contains a `<servers>` block with Nexus credentials, whose `<id>` matches the `<distributionManagement>` repository IDs in `pom.xml`.

---

## PHASE 3 — Nexus Server Setup

### Launched instance

```bash
aws ec2 run-instances \
  --region ap-south-1 \
  --image-id ami-0aa761682283b4cc8 \
  --instance-type c7i-flex.large \
  --key-name myWebServer \
  --security-group-ids sg-09c50e5f8249e552e \
  --block-device-mappings '[{"DeviceName":"/dev/sda1","Ebs":{"VolumeSize":20,"VolumeType":"gp3"}}]' \
  --tag-specifications 'ResourceType=instance,Tags=[{Key=Name,Value=nexus-server}]' \
  --query "Instances[0].InstanceId" --output text
# Result: i-0ca0c755453e7ce6e
```

### Setup script

```bash
#!/bin/bash
set -e

sudo apt-get update
sudo apt-get upgrade -y

# Install Docker (official repo)
sudo apt-get install -y ca-certificates curl
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu \
  $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt-get update
sudo apt-get install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
sudo usermod -aG docker $USER

# Run Nexus with persistent volume
sudo docker run -d -p 8081:8081 --name nexus --restart unless-stopped -v nexus-data:/nexus-data sonatype/nexus3
```

Why `-v nexus-data:/nexus-data`: persistent volume — without it, all uploaded artifacts and Nexus config vanish if the container is ever recreated.

Why `--restart unless-stopped`: auto-restarts the container after host reboots — without it, EC2 stop/start would bring the host back but Nexus itself would stay down.

### Get initial admin password

```bash
sudo docker exec nexus cat /nexus-data/admin.password
```

Takes 1-2 minutes after first container start before this file exists (Nexus is slow to initialize).

### Nexus initial configuration (via browser at :8081)

- Logged in with admin + the initial password
- Set new admin password during setup wizard
- Created maven-releases and maven-snapshots repositories (if not already present by default)

---

## PHASE 4 — SonarQube + Postgres Server Setup

### Launched instance

```bash
aws ec2 run-instances \
  --region ap-south-1 \
  --image-id ami-0aa761682283b4cc8 \
  --instance-type m7i-flex.large \
  --key-name myWebServer \
  --security-group-ids sg-09c50e5f8249e552e \
  --block-device-mappings '[{"DeviceName":"/dev/sda1","Ebs":{"VolumeSize":20,"VolumeType":"gp3"}}]' \
  --tag-specifications 'ResourceType=instance,Tags=[{Key=Name,Value=sonarqube-server}]' \
  --query "Instances[0].InstanceId" --output text
# Result: i-0ecf03e9ec37e1224
```

### Setup script

```bash
#!/bin/bash
set -e

sudo apt-get update
sudo apt-get upgrade -y

# Set required kernel parameter for SonarQube's embedded Elasticsearch
sudo sysctl -w vm.max_map_count=262144
echo "vm.max_map_count=262144" | sudo tee -a /etc/sysctl.d/99-sonarqube.conf
sudo sysctl --system

# Install Docker (official repo — same steps as other servers)
sudo apt-get install -y ca-certificates curl
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu \
  $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt-get update
sudo apt-get install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
sudo usermod -aG docker $USER

# Create dedicated Docker network for SonarQube + Postgres
sudo docker network create sonarnet

# Run Postgres (external DB — SonarQube's embedded H2 is unsupported for real use)
sudo docker run -d --name sonarqube_db --network sonarnet --restart unless-stopped \
  -e POSTGRES_USER=sonar \
  -e POSTGRES_PASSWORD=sonar \
  -e POSTGRES_DB=sonarqube \
  -v postgresql_data:/var/lib/postgresql/data \
  postgres:15

# Run SonarQube, pointed at Postgres
sudo docker run -d --name sonarqube --network sonarnet --restart unless-stopped -p 9000:9000 \
  -e SONAR_JDBC_URL=jdbc:postgresql://sonarqube_db:5432/sonarqube \
  -e SONAR_JDBC_USERNAME=sonar \
  -e SONAR_JDBC_PASSWORD=sonar \
  -v sonarqube_data:/opt/sonarqube/data \
  -v sonarqube_extensions:/opt/sonarqube/extensions \
  -v sonarqube_logs:/opt/sonarqube/logs \
  sonarqube:lts-community
```

Why vm.max_map_count: SonarQube bundles Elasticsearch, which has a hard documented requirement for this kernel parameter to be >= 262144. Docker containers inherit this from the HOST kernel, not settable inside the container. Without it, SonarQube starts then immediately crashes with an Elasticsearch bootstrap-check failure.

Why external Postgres instead of embedded H2: SonarSource's own docs mark H2 as "for evaluation only" and unsupported for production. Deliberate, documented production-hardening choice, diverging from the tutorial on purpose.

Why docker network create sonarnet: creates an isolated Docker network so SonarQube and Postgres can communicate by container name (`sonarqube_db:5432`) without exposing Postgres to the host network or needing manual IP management.

Why postgres:15 (pinned version, not :latest): avoids a moving-target version that could unexpectedly break compatibility.

### SonarQube initial configuration (via browser at :9000)

- Default login: admin / admin (forced password change on first login)
- Verified it's using Postgres (Administration → System → database info shows PostgreSQL, not H2)
- Generated an authentication token (Administration → Security → Users → Tokens) — this is what gets stored in Jenkins as `sonar-token`

---

## PHASE 5 — Source Code: Private Repo Setup

### Cloned reference app, stripped history, pushed to own private repo

```bash
cd ~/aakash/devops/devops-project
git clone https://github.com/jaiswaladi246/Boardgame.git boardgame-jenkins-cicd-pipeline
cd boardgame-jenkins-cicd-pipeline
rm -rf .git
git init
git add .
git commit -m "Initial commit: Boardgame app source (reference Jenkinsfile included, will be rewritten)"
git branch -M main
git remote add origin https://github.com/aakash-1004/boardgame-jenkins-cicd-pipeline.git
git push -u origin main
```

Why rm -rf .git + git init: removes the original repo's entire commit history and link back to the creator's repo — makes this a genuinely fresh project, not a visible fork.

### Modified pom.xml

Changed Maven repository URLs in `<distributionManagement>` to point at our own Nexus server's maven-releases and maven-snapshots repositories.

---

## PHASE 6 — RBAC: Jenkins → Kubernetes Deployment Access

All run on the K8s master.

### Create namespace

```bash
kubectl create namespace webapps
```

### Create ServiceAccount

```bash
cat > serviceaccount.yaml << 'EOF'
apiVersion: v1
kind: ServiceAccount
metadata:
  name: jenkins
  namespace: webapps
EOF
kubectl apply -f serviceaccount.yaml
```

### Create Role (namespace-scoped, not ClusterRole)

```bash
cat > role.yaml << 'EOF'
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: app-role
  namespace: webapps
rules:
  - apiGroups:
      - ""
      - apps
      - autoscaling
      - batch
      - extensions
      - policy
      - rbac.authorization.k8s.io
    resources:
      - pods
      - deployments
      - services
      - replicasets
      - configmaps
      - secrets
      - ingresses
    verbs:
      - get
      - list
      - watch
      - create
      - update
      - patch
      - delete
EOF
kubectl apply -f role.yaml
```

Why Role (namespace-scoped) not ClusterRole: least-privilege — Jenkins only needs to manage resources inside `webapps`, not the entire cluster.

### Create RoleBinding

```bash
cat > rolebinding.yaml << 'EOF'
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: app-rolebinding
  namespace: webapps
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: Role
  name: app-role
subjects:
  - kind: ServiceAccount
    name: jenkins
    namespace: webapps
EOF
kubectl apply -f rolebinding.yaml
```

### Create token Secret (required since K8s 1.24)

```bash
cat > secret.yaml << 'EOF'
apiVersion: v1
kind: Secret
type: kubernetes.io/service-account-token
metadata:
  name: mysecretname
  namespace: webapps
  annotations:
    kubernetes.io/service-account.name: jenkins
EOF
kubectl apply -f secret.yaml
```

Why this must be explicitly created: Kubernetes used to auto-generate a token Secret for every ServiceAccount. This behavior was removed in 1.24 as a security hardening measure — now you must explicitly opt in.

### Retrieve token and add to Jenkins

```bash
kubectl get secret mysecretname -n webapps -o jsonpath='{.data.token}' | base64 --decode
```

Copied this JWT-style token string into Jenkins as credential `k8s-cred` (Kind: Secret text).

### Verified network connectivity (Jenkins → K8s API)

From the Jenkins EC2 box:

```bash
curl -k https://172.31.33.140:6443
```

Result: `403 Forbidden` from `system:anonymous` — confirms network path works (a real timeout would mean security group blocking), and RBAC correctly rejects unauthenticated access. The 403 is expected, not an error.

---

## PHASE 7 — Jenkinsfile: Writing, Debugging, and Correction

### Initial Jenkinsfile written by hand (from the video), with bugs caught in review

Bugs found and fixed:

1. `scripts{}` → `script{}` — appeared 3 times, not a valid Jenkins DSL step name
2. `docker built` → `docker build` — not a valid Docker subcommand
3. `mvn deply` → `mvn deploy` — not a valid Maven lifecycle phase
4. `credentialId` → `credentialsId` — wrong parameter name on waitForQualityGate
5. `abortPipeline: false` → `true` — a non-enforcing quality gate defeats its purpose
6. Both Trivy scans wrote to the same filename (`trivy-fs-report.html`) — the image scan silently overwrote the filesystem scan's report; renamed second to `trivy-image-report.html`
7. `:latest` tag → `${BUILD_NUMBER}` — for traceability between deployed image and the Jenkins run that produced it
8. Stage name typo: "Dcoker Image Scan" → "Docker Image Scan"

### deployment-service.yaml — 3 bugs caught before it could fail silently

1. Wrong image entirely: `adijaiswal/boardshack:latest` → `aakash0908/boardgame:BUILD_TAG_PLACEHOLDER` (pointed at tutorial creator's image, not ours)
2. Missing `namespace: webapps` on both Deployment and Service — would silently bypass all RBAC work by deploying to `default` namespace
3. `type: LoadBalancer` — kept deliberately; EXTERNAL-IP stays `<pending>` forever on bare-metal kubeadm (no cloud controller manager to provision a real LB) — accessible via auto-assigned NodePort instead

### BUILD_TAG_PLACEHOLDER mechanism

The Jenkinsfile's deploy stage runs:

```groovy
sh "sed -i 's/BUILD_TAG_PLACEHOLDER/${BUILD_NUMBER}/g' deployment-service.yaml"
```

This rewrites the placeholder in the YAML to the actual current build number (e.g., `aakash0908/boardgame:2`) at deploy time, matching the exact image this specific pipeline run just built and pushed. The sed operates on the Jenkins workspace copy only — doesn't commit back to Git, so each run starts fresh from the placeholder.

### Final working Jenkinsfile (13 stages + post block)

```groovy
pipeline {
    agent any
    tools {
        jdk 'jdk17'
        maven 'maven3'
    }
    environment {
        SCANNER_HOME = tool 'sonar-scanner'
    }
    stages {
        stage('Git Checkout') {
            steps {
                git branch: 'main', credentialsId: 'git-cred', url: 'https://github.com/aakash-1004/boardgame-jenkins-cicd-pipeline.git'
            }
        }
        stage('Compile') {
            steps {
                sh "mvn compile"
            }
        }
        stage('Test') {
            steps {
                sh "mvn test"
            }
        }
        stage('File System Scan') {
            steps {
                sh "trivy fs --format table -o trivy-fs-report.html ."
            }
        }
        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('sonar') {
                    sh ''' $SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=boardgame -Dsonar.projectKey=boardgame \
                           -Dsonar.java.binaries=. \
                           -Dsonar.exclusions=**/trivy-*-report.html '''
                }
            }
        }
        stage('Quality Gate') {
            steps {
                script {
                    waitForQualityGate abortPipeline: true, credentialsId: 'sonar-token'
                }
            }
        }
        stage('Build') {
            steps {
                sh "mvn package"
            }
        }
        stage('Publish To Nexus') {
            steps {
                withMaven(globalMavenSettingsConfig: 'global-settings', jdk: 'jdk17', maven: 'maven3', traceability: true) {
                    sh "mvn deploy"
                }
            }
        }
        stage('Build & Tag Docker Image') {
            steps {
                script {
                    withDockerRegistry(credentialsId: 'docker-cred', toolName: 'docker') {
                        sh "docker build -t aakash0908/boardgame:${BUILD_NUMBER} ."
                    }
                }
            }
        }
        stage('Docker Image Scan') {
            steps {
                sh "trivy image --format table -o trivy-image-report.html aakash0908/boardgame:${BUILD_NUMBER}"
            }
        }
        stage('Push Docker Image') {
            steps {
                script {
                    withDockerRegistry(credentialsId: 'docker-cred', toolName: 'docker') {
                        sh "docker push aakash0908/boardgame:${BUILD_NUMBER}"
                    }
                }
            }
        }
        stage('Deploy to Kubernetes') {
            steps {
                sh "sed -i 's/BUILD_TAG_PLACEHOLDER/${BUILD_NUMBER}/g' deployment-service.yaml"
                withKubeConfig(caCertificate: '', clusterName: 'kubernetes', contextName: '', credentialsId: 'k8s-cred', namespace: 'webapps', restrictKubeConfigAccess: false, serverUrl: 'https://172.31.33.140:6443') {
                    sh "kubectl apply -f deployment-service.yaml"
                }
            }
        }
        stage('Verify the Deployment') {
            steps {
                withKubeConfig(caCertificate: '', clusterName: 'kubernetes', contextName: '', credentialsId: 'k8s-cred', namespace: 'webapps', restrictKubeConfigAccess: false, serverUrl: 'https://172.31.33.140:6443') {
                    sh "kubectl get pods -n webapps"
                    sh "kubectl get svc -n webapps"
                }
            }
        }
    }
    post {
        always {
            script {
                def jobName = env.JOB_NAME
                def buildNumber = env.BUILD_NUMBER
                def pipelineStatus = currentBuild.result ?: 'UNKNOWN'
                def bannerColor = pipelineStatus.toUpperCase() == 'SUCCESS' ? 'green' : 'red'
                def body = """
                    <html>
                    <body>
                    <div style="border: 4px solid ${bannerColor}; padding: 10px;">
                    <h2>${jobName} - Build ${buildNumber}</h2>
                    <div style="background-color: ${bannerColor}; padding: 10px;">
                    <h3 style="color: white;">Pipeline Status: ${pipelineStatus.toUpperCase()}</h3>
                    </div>
                    <p>Check the <a href="${BUILD_URL}">console output</a>.</p>
                    </div>
                    </body>
                    </html>
                """
                emailext (
                    subject: "${jobName} - Build ${buildNumber} - ${pipelineStatus.toUpperCase()}",
                    body: body,
                    to: 'aakashrao1004@gmail.com',
                    from: 'aakashrao0908@gmail.com',
                    replyTo: 'aakashrao0908@gmail.com',
                    mimeType: 'text/html',
                    attachmentsPattern: 'trivy-*-report.html'
                )
            }
        }
    }
}
```

### sonar.exclusions — a bug traced to a single missing character

First Quality Gate failure: 79.62% code duplication. Investigation via SonarQube's file-level breakdown showed all actual Java source at 0.0% duplication — the false positive came from Trivy's own HTML scan reports (templated, repetitive boilerplate) sitting in the analysis scope. Added `-Dsonar.exclusions=**/trivy-*-report.html` to fix this. First attempt silently failed because the pattern was truncated to `.htm` (missing final `l`) — confirmed by reading the scanner's own echoed config line in the console log: `Excluded sources: **/trivy-*-report.htm`, `0 files ignored`.

### Pipeline-from-SCM conversion

Changed job configuration from "Pipeline script" (Jenkinsfile typed in Jenkins UI) to "Pipeline script from SCM":

- SCM: Git
- Repository URL: `https://github.com/aakash-1004/boardgame-jenkins-cicd-pipeline.git`
- Credentials: `git-cred`
- Branch: `*/main`
- Script Path: `Jenkinsfile`

Why this matters: the Jenkinsfile now lives in Git, version-controlled alongside the application code — has real commit history, can be code-reviewed via PRs, survives Jenkins server rebuilds.

### Email notification debugging

1. Tested via Jenkins System → Extended E-mail Notification → "Test configuration" button — test email arrived successfully to `aakashrao1004@gmail.com`
2. Pipeline emails never arrived — diagnosed via Jenkins Script Console test, which revealed: SMTP port was set to 25, while "Use SSL" was checked — port 25 doesn't support SSL, AND AWS blocks outbound port 25 by default on all EC2 instances as an anti-spam measure
3. Fixed by changing SMTP port to 465 — emails started arriving immediately

---

## PHASE 8 — Pipeline Runs and Real Debugging

### First real run — 8 stages passed, failed at Docker build

Successful: Git Checkout, Compile, Test (6 tests, 0 failures), File System Scan, SonarQube Analysis, Quality Gate, Build, Publish to Nexus (real 48MB jar uploaded to Nexus successfully). Failed at Docker build: `docker login` error — "client version 1.29 is too old. Minimum supported API version is 1.40."

Root cause: `withDockerRegistry`'s `toolName: 'docker'` auto-downloaded an ancient Docker CLI from a legacy distribution URL, completely separate from the modern docker-ce already installed. Fixed via Jenkins Global Tool Configuration — pointed the `docker` tool at `/usr` (Installation root), unchecked "Install automatically."

### Second run — Quality Gate failure (genuine enforcement)

SonarQube gate failed on 79.62% code duplication (Trivy report files analyzed as source). Pipeline correctly aborted all downstream stages. Fixed via sonar.exclusions as described above.

### Final successful run

All 13 stages green. Nexus upload confirmed. Docker image pushed to Docker Hub with build-number tag. Kubernetes deployment applied via RBAC-scoped token. Pods verified Running. App accessible at `http://<worker-node-ip>:31540`. Email notification sent.

---

## PHASE 9 — Monitoring Stack

### Launched monitoring server

```bash
aws ec2 run-instances \
  --region ap-south-1 \
  --image-id ami-0aa761682283b4cc8 \
  --instance-type c7i-flex.large \
  --key-name myWebServer \
  --security-group-ids sg-09c50e5f8249e552e \
  --block-device-mappings '[{"DeviceName":"/dev/sda1","Ebs":{"VolumeSize":20,"VolumeType":"gp3"}}]' \
  --tag-specifications 'ResourceType=instance,Tags=[{Key=Name,Value=monitoring-server}]' \
  --query "Instances[0].InstanceId" --output text
# Result: i-0964449c2cec9ab33
```

### Prometheus installation (step by step, raw binary — not apt)

1. Created dedicated system user:

```bash
sudo useradd --system --no-create-home --shell /usr/sbin/nologin prometheus
```

Why --shell /usr/sbin/nologin: this account can never be logged into — no password exists, no shell access possible. Exists purely as an OS-level identity for file ownership and process attribution.

2. Downloaded and extracted:

```bash
cd ~
wget https://github.com/prometheus/prometheus/releases/download/v3.14.0/prometheus-3.14.0.linux-amd64.tar.gz
tar -xvzf prometheus-3.14.0.linux-amd64.tar.gz
```

3. Moved binaries into place:

```bash
sudo mv prometheus-3.14.0.linux-amd64/prometheus /usr/local/bin/
sudo mv prometheus-3.14.0.linux-amd64/promtool /usr/local/bin/
```

4. Created config and data directories:

```bash
sudo mkdir -p /etc/prometheus
sudo mkdir -p /var/lib/prometheus
```

/etc/prometheus: config files (standard Linux convention for config). /var/lib/prometheus: time-series database storage (standard for application data that changes over time). Empty until Prometheus actually starts.

5. Moved bundled config file:

```bash
cd prometheus-3.14.0.linux-amd64
sudo mv prometheus.yml /etc/prometheus/
```

6. Set ownership:

```bash
sudo chown -R prometheus:prometheus /etc/prometheus /var/lib/prometheus /usr/local/bin/prometheus /usr/local/bin/promtool
```

7. Edited config file:

```bash
sudo nano /etc/prometheus/prometheus.yml
```

Content (initial, later expanded):

```yaml
global:
  scrape_interval: 15s
scrape_configs:
  - job_name: 'prometheus'
    static_configs:
      - targets: ['localhost:9090']
```

8. Created systemd service file:

```bash
sudo nano /etc/systemd/system/prometheus.service
```

```ini
[Unit]
Description=Prometheus
Wants=network-online.target
After=network-online.target

[Service]
User=prometheus
Group=prometheus
Type=simple
ExecStart=/usr/local/bin/prometheus --config.file=/etc/prometheus/prometheus.yml --storage.tsdb.path=/var/lib/prometheus/

[Install]
WantedBy=multi-user.target
```

Why a manual service file is needed: Prometheus was installed from a raw GitHub binary tarball, not an apt package — no .deb package means no pre-written systemd service included. apt-installed tools (Jenkins, Docker, Grafana) get this automatically; manually-downloaded binaries require you to write it yourself.

9. Started the service:

```bash
sudo systemctl daemon-reload
sudo systemctl start prometheus
sudo systemctl enable prometheus
sudo systemctl status prometheus --no-pager
```

### YAML config bug hit twice

First time: missing closing bracket `]` on a targets line — Prometheus failed to start with INVALIDARGUMENT. Second time: indentation mismatch on two new job blocks (8 spaces where 4 were expected) — same failure.

Both diagnosed via `cat -A /etc/prometheus/prometheus.yml` (shows invisible whitespace characters) + `sudo journalctl -u prometheus --no-pager -n 30` (shows the full, untruncated error that systemctl status cuts off).

### Node Exporter installation (on monitoring server, then on Jenkins server)

Same manual-binary pattern as Prometheus:

```bash
sudo useradd --system --no-create-home --shell /usr/sbin/nologin node_exporter
wget https://github.com/prometheus/node_exporter/releases/download/v1.12.1/node_exporter-1.12.1.linux-amd64.tar.gz
tar -xvzf node_exporter-1.12.1.linux-amd64.tar.gz
sudo mv node_exporter-1.12.1.linux-amd64/node_exporter /usr/local/bin/
sudo chown node_exporter:node_exporter /usr/local/bin/node_exporter
```

Service file (`/etc/systemd/system/node_exporter.service`):

```ini
[Unit]
Description=Node Exporter
Wants=network-online.target
After=network-online.target

[Service]
User=node_exporter
Group=node_exporter
Type=simple
ExecStart=/usr/local/bin/node_exporter

[Install]
WantedBy=multi-user.target
```

Simpler than Prometheus's service — no config file or storage path flags needed. Node Exporter reads live OS stats on demand (from /proc, /sys) and serves them on port 9100 whenever scraped.

### Grafana installation (via apt — auto-creates systemd service)

```bash
sudo apt-get install -y apt-transport-https software-properties-common wget
sudo mkdir -p /etc/apt/keyrings/
wget -q -O - https://apt.grafana.com/gpg.key | gpg --dearmor | sudo tee /etc/apt/keyrings/grafana.gpg > /dev/null
echo "deb [signed-by=/etc/apt/keyrings/grafana.gpg] https://apt.grafana.com stable main" | sudo tee /etc/apt/sources.list.d/grafana.list
sudo apt-get update
sudo apt-get install -y grafana
sudo systemctl start grafana-server
sudo systemctl enable grafana-server
```

Why Grafana is different from Prometheus/Node Exporter: ships as a proper .deb package via apt — auto-creates its own system user and systemd service during install. Service name is `grafana-server`, not `grafana`.

Accessed at `http://<monitoring-ip>:3000`. Default login: admin / admin (forced password change on first login).

Connected to Prometheus data source: Connections → Data sources → Add → Prometheus → URL: `http://localhost:9090` → Save & Test.

### Blackbox Exporter installation

Same manual-binary pattern:

```bash
sudo useradd --system --no-create-home --shell /usr/sbin/nologin blackbox_exporter
wget https://github.com/prometheus/blackbox_exporter/releases/download/v0.28.0/blackbox_exporter-0.28.0.linux-amd64.tar.gz
tar -xvzf blackbox_exporter-0.28.0.linux-amd64.tar.gz
sudo mv blackbox_exporter-0.28.0.linux-amd64/blackbox_exporter /usr/local/bin/
sudo mkdir -p /etc/blackbox_exporter
sudo mv blackbox_exporter-0.28.0.linux-amd64/blackbox.yml /etc/blackbox_exporter/
sudo chown blackbox_exporter:blackbox_exporter /usr/local/bin/blackbox_exporter
sudo chown -R blackbox_exporter:blackbox_exporter /etc/blackbox_exporter
```

Service file (`/etc/systemd/system/blackbox_exporter.service`):

```ini
[Unit]
Description=Blackbox Exporter
Wants=network-online.target
After=network-online.target

[Service]
User=blackbox_exporter
Group=blackbox_exporter
Type=simple
ExecStart=/usr/local/bin/blackbox_exporter --config.file=/etc/blackbox_exporter/blackbox.yml

[Install]
WantedBy=multi-user.target
```

Blackbox needs a config file (unlike Node Exporter) because it needs to know what kinds of probes to support (HTTP, TCP, ICMP) — the config defines probe "modules." Default bundled config includes `http_2xx` (the one we actually use), plus TCP/SSH/gRPC/ICMP modules we don't need.

### Final prometheus.yml (all scrape targets)

```yaml
global:
  scrape_interval: 15s

scrape_configs:
  - job_name: 'prometheus'
    static_configs:
      - targets: ['localhost:9090']

  - job_name: 'node_exporter'
    static_configs:
      - targets: ['localhost:9100']

  - job_name: 'blackbox_http'
    metrics_path: /probe
    params:
      module: [http_2xx]
    static_configs:
      - targets:
          - http://<jenkins-ip>:8080/login
          - http://<sonarqube-ip>:9000
          - http://<app-nodeport-ip>:31540
    relabel_configs:
      - source_labels: [__address__]
        target_label: __param_target
      - source_labels: [__param_target]
        target_label: instance
      - target_label: __address__
        replacement: localhost:9115

  - job_name: 'jenkins_node_exporter'
    static_configs:
      - targets: ['<jenkins-ip>:9100']

  - job_name: 'jenkins_app_metrics'
    metrics_path: /prometheus
    static_configs:
      - targets: ['<jenkins-ip>:8080']
```

Why Blackbox's config shape is different (the relabel trick): Prometheus normally scrapes targets directly. For Blackbox, we rewrite the request so Prometheus scrapes Blackbox itself (localhost:9115/probe?target=...), telling Blackbox which real target to probe via a query parameter. This relabeling pattern is standard for every Blackbox+Prometheus integration.

Why Jenkins uses /login for Blackbox probing: Jenkins requires authentication for most endpoints — an unauthenticated http_2xx probe gets a 403 (correctly), which Blackbox marks as failure. /login responds with a 200 without authentication, genuinely proving "Jenkins is alive" without needing to solve auth in the probe.

Why jenkins_app_metrics uses metrics_path: /prometheus: the Jenkins Prometheus plugin exposes metrics at this non-standard path, not the default /metrics — must be specified explicitly.

### Grafana dashboards imported

- Blackbox Exporter dashboard — imported from Grafana.com by dashboard ID
- Node Exporter Full (ID 1860) — imported similarly, visualizes Jenkins server's OS-level metrics

---

## PHASE 10 — Repo Polish

### Added RBAC manifests to the repo

Created `k8s/rbac/` directory containing serviceaccount.yaml, role.yaml, rolebinding.yaml, and secret-template.yaml (template only — does NOT contain the live token, since the real value is auto-populated server-side by Kubernetes).

### Updated README.md

Replaced the original placeholder with a comprehensive project README covering architecture diagram, pipeline stages, key engineering decisions, and the full tech stack.

### Added .dockerignore

```
.git
.gitignore
target
*.md
k8s
Jenkinsfile
```

Why, even though the Dockerfile uses selective COPY (not blanket COPY .): reduces build context transfer size (Docker sends everything not excluded to the daemon before reading any Dockerfile instructions), and acts as defense-in-depth against a future edit accidentally introducing a blanket COPY.

### Updated .gitignore

Added: `trivy-*-report.html` — prevents scan report artifacts from accidentally being committed if a scan is ever run locally in this directory.

---

## Instance IDs — Full Reference

|Server|Instance ID|Type|
|---|---|---|
|Jenkins|i-0c120d94e908a53f2|m7i-flex.large|
|Nexus|i-0ca0c755453e7ce6e|c7i-flex.large|
|SonarQube|i-0ecf03e9ec37e1224|m7i-flex.large|
|K8s master|i-04e788d52fb81aef9|c7i-flex.large|
|K8s slave-1|i-00dd691370aa08e55|t3.small|
|K8s slave-2|i-03f2096f48784f669|t3.small|
|Monitoring|i-0964449c2cec9ab33|c7i-flex.large|

All in ap-south-1, all using security group sg-09c50e5f8249e552e (myWebServer), all using key pair myWebServer.

### Stop all instances (to pause billing)

```bash
aws ec2 stop-instances --region ap-south-1 --instance-ids \
  i-0c120d94e908a53f2 i-0ca0c755453e7ce6e i-0ecf03e9ec37e1224 \
  i-04e788d52fb81aef9 i-00dd691370aa08e55 i-03f2096f48784f669 \
  i-0964449c2cec9ab33
```

Note on resume: public IPs will change on next start (no Elastic IPs allocated). Private IPs persist. Prometheus config (prometheus.yml) references public IPs for Blackbox/Jenkins targets and will need updating after restart. The Jenkinsfile's serverUrl uses the master's private IP (172.31.33.140) which is safe across stop/start.