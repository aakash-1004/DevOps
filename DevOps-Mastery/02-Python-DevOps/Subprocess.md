# Python — subprocess

**Tags:** #python #devops #automation #day1
**Status:** ✅ Understood
**Interview Relevance:** 🔴 High — core tool for Python automation scripts

---

## What Is It?

`subprocess` lets Python run shell commands and capture their output. This is how Python scripts interact with the OS — running `df`, `free`, `systemctl`, `kubectl`, `docker`, etc.

---

## Basic Usage

```python
import subprocess

result = subprocess.run(
    ["df", "-h"],
    capture_output=True,
    text=True
)

print(result.stdout)       # command output as string
print(result.returncode)   # 0 = success, non-zero = failure
```

**Key parameters:**

| Parameter | What It Does |
|-----------|-------------|
| `capture_output=True` | Captures stdout and stderr |
| `text=True` | Returns strings instead of bytes |
| `check=True` | Raises exception if command fails |

---

## With Error Handling

```python
try:
    result = subprocess.run(
        ["systemctl", "status", "docker"],
        capture_output=True,
        text=True,
        check=True
    )
    print(result.stdout)
except subprocess.CalledProcessError as e:
    print(f"Command failed: {e.returncode}")
    print(e.stderr)
```

`check=True` is the Python equivalent of `set -e` in bash — raises `CalledProcessError` if the command returns non-zero.

---

## Real Example — Memory Usage

```python
import subprocess

def get_memory_usage():
    result = subprocess.run(["free", "-m"], capture_output=True, text=True, check=True)
    fields = result.stdout.strip().split("\n")[1].split()
    total = int(fields[1])
    used = int(fields[2])
    return round(used / total * 100)

print(f"Memory usage: {get_memory_usage()}%")
```

**Parsing logic:**
- `.split("\n")` → splits output into lines
- `[1]` → Mem: row (index 0 is the header)
- `.split()` → splits by whitespace into fields
- `fields[1]` = total, `fields[2]` = used

---

## Common Gotcha

Never name your Python file the same as a standard library module.

```
subprocess.py   ← WRONG — shadows the built-in module
os.py           ← WRONG
json.py         ← WRONG
test_subprocess.py  ← CORRECT
```

---

## Interview — Ready to Speak

**Q: "How would you run a shell command from Python and handle failures?"**

> "I use `subprocess.run()` with `capture_output=True` and `text=True` to get the output as a string. For error handling I add `check=True` which raises a `CalledProcessError` if the command returns non-zero — I wrap it in a try/except to handle the failure gracefully. I avoid `subprocess.shell=True` because it's a security risk — it passes the command through the shell which can be exploited with shell injection."

---

## Wikilinks
- [[JSON-YAML-Parsing.md]]
- [[Requests.md]]
- [[Python-DevOps-Scripts.md]]
