# Lab — Server Health Check Script

**Tags:** #linux #bash #lab #day1
**Status:** ✅ Completed & Working
**File:** `~/aakash/devops/health_check.sh`

---

## What It Does

- Checks CPU usage against a threshold (default: 80%)
- Checks memory usage against a threshold (default: 80%)
- Checks all disk partitions against a threshold (default: 90%)
- Checks if a target process (`dockerd`) is running
- Logs everything with timestamps to `/tmp/health_check.log`
- Uses lock file to prevent duplicate runs
- Cleans up lock file on any exit via `trap`

---

## Key Pattern — Run All Checks, Collect Failures

```bash
local FAILED=0
check_cpu     || FAILED=1
check_memory  || FAILED=1
check_disk    || FAILED=1
check_process || FAILED=1

if (( FAILED == 1 )); then
    log_error "Health check FAILED"
    exit 1
fi
```

**Why:** `set -e` would exit on the first failed check. `|| FAILED=1` overrides that — if the check fails, don't exit, just record it. All checks run, failures are collected, reported at the end.

---

## Bug Found During Lab

**Problem:** `PROCESS_NAME="docker"` → check always failed even though Docker was running.

**Root cause:** `pgrep -x` matches the exact process name. The Docker daemon process is `dockerd`, not `docker`.

**Fix:** `PROCESS_NAME="dockerd"`

**Lesson:** Always verify the real process name before hardcoding:
```bash
ps aux | grep docker
# or
pgrep -x dockerd
```

This type of bug is common in production — always verify assumptions.

---

## Test Cases Run

| Test | How | Expected Result |
|------|-----|----------------|
| Normal run | `./health_check.sh` | All checks pass |
| Process not running | `PROCESS_NAME="fakeprocess"` | Process check fails |
| Lock file active | `touch /tmp/health_check.lock` then run | Exits with "Previous run still active" |
| Low disk threshold | `DISK_THRESHOLD=1` | Disk check fails |

---

## Wikilinks
- [[1.5-Bash-Strict-Mode]]
- [[Logging-Trap-Lock-Files.md]]
- [[1.7-Log-parsing]]]
- [[1.6-Cron-Jobs]]