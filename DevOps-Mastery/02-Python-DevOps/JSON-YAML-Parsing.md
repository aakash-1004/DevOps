# JSON & YAML Parsing

**Tags:** #python #devops #json #yaml #day1
**Status:** ✅ Understood
**Interview Relevance:** 🔴 High — YAML is everywhere in K8s, Terraform, CI/CD

---

## JSON — Built-in, No Install Needed

```python
import json

# Parse JSON string → Python dict
data = '{"name": "nginx", "replicas": 3}'
parsed = json.loads(data)
print(parsed["name"])      # nginx
print(parsed["replicas"])  # 3

# Read JSON file
with open("config.json") as f:
    config = json.load(f)

# Write/print JSON
data = {"service": "nginx", "port": 80}
print(json.dumps(data, indent=2))
```

**`json.loads()` vs `json.load()`:**
- `json.loads()` → parse a **string**
- `json.load()` → parse a **file object**

---

## YAML — Needs pyyaml

```bash
pip install pyyaml
```

```python
import yaml

manifest = """
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx
spec:
  replicas: 3
  image: nginx:latest
"""

parsed = yaml.safe_load(manifest)
print(parsed["kind"])               # Deployment
print(parsed["spec"]["replicas"])   # 3
print(parsed["metadata"]["name"])   # nginx
```

**Always use `yaml.safe_load()` not `yaml.load()`**
`yaml.load()` can execute arbitrary code embedded in YAML — it's a security vulnerability. `safe_load()` only parses data structures.

---

## Navigating Nested Structures

```python
# Nested dict access
data["spec"]["replicas"]

# List access
slides = data["slideshow"]["slides"]
for slide in slides:
    print(slide["title"])

# Safe access with .get() — returns None if key missing
name = data.get("metadata", {}).get("name", "unknown")
```

---

## Real DevOps Connection

This is how tools like Helm, ArgoCD, and kubectl work internally — they parse YAML manifests into Python/Go data structures and manipulate them.

Same pattern for hitting the Kubernetes API:
```python
response = requests.get("https://k8s-api/api/v1/namespaces/default/pods",
                        headers={"Authorization": f"Bearer {token}"})
pods = response.json()["items"]
for pod in pods:
    print(pod["metadata"]["name"])
```

---

## Interview — Ready to Speak

**Q: "How do you parse a Kubernetes manifest in Python?"**

> "I use `yaml.safe_load()` from the pyyaml library — it parses the YAML into a Python dictionary. Then I navigate it like any nested dict: `manifest['spec']['replicas']` to get replicas, `manifest['metadata']['name']` for the name. I always use `safe_load` not `load` because the unsafe version can execute arbitrary code embedded in the YAML."

---

## Wikilinks
- [[Subprocess.md]]
- [[Requests.md]]
- [[Python-DevOps-Scripts.md]]