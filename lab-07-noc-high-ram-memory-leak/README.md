# Lab 07 – NOC: High RAM, memory leak

**Role:** NOC / Support Tier 1 · **Platform:** Linux · **Skills:** `free -h`, `ps aux --sort=-%mem`, `pgrep`, reading app logs for growth, safe stop/restart, escalation

## Scenario
> 🚨 Ticket: "The server's memory alert is climbing, and the team is worried apps will start crashing."

A Python "session cache" service kept caching sessions and never freed old ones (a memory leak), growing by 50 MB every 3 seconds to 3 GB.

## Lab setup (for reproducing)
```python
# /opt/sessioncache/sessioncache.py  (run with: sudo setsid nohup python3 /opt/sessioncache/sessioncache.py &)
import time, datetime
cache = []
while True:
    if len(cache) < 60:                              # lab safety cap ~3 GB
        cache.append(bytearray(50 * 1024 * 1024))    # +50 MB, never freed = leak
    open("/var/log/sessioncache/sessioncache.log", "a").write(
        f"{datetime.datetime.now():%F %T} INFO cached sessions batch={len(cache)} total_mb={len(cache)*50}\n")
    time.sleep(3)
```

## Investigation
| # | Command | What it showed | Why it matters |
|---|---|---|---|
| 1 | `free -h` | 15Gi total, **2.9Gi available** (free 797Mi) | RAM tight. Read **available**, not "free" (Linux uses spare RAM as cache) |
| 2 | `ps aux --sort=-%mem \| head -5` | root, `python3 /opt/sessioncache/sessioncache.py`, **18.8% MEM, ~3 GB RSS** | Finds the culprit (`-` = biggest first) |
| 3 | `head -3` / `tail -3` of `/var/log/sessioncache/sessioncache.log` | `total_mb=50`, rising to `total_mb=3000`, never going down | Proves a **leak** (growing), not just a big app |
| 4 | `pgrep -af sessioncache` | 2 PIDs: the `sudo` launcher + the `python3` app | Pick the app's PID |

## Fix
```bash
sudo kill <PID of python3 line>          # lab (no systemd)
# production: sudo systemctl restart sessioncache   (frees RAM and the service comes back)
```

## Verification
```bash
pgrep -af sessioncache                   # nothing = stopped
free -h                                  # available 2.9Gi -> 5.8Gi (~3 GiB freed)
```

## Key lessons
- **RAM** = short-term working memory; **disk** = storage (Lab 05). Out of RAM means the **OOM killer** kills processes (`dmesg | grep -i oom`).
- A **leak** = memory that keeps growing over time: compare two points in time, not one snapshot.
- `--sort=-%mem`: the `-` comes right before `%mem` (after `=`). Without it, the sort runs smallest-first.
- **"Why not just restart?"** Collect evidence first (a restart wipes it), restart the **service** not the server, then **escalate** to the devs. A restart is a band-aid; the real fix is in the code (expire old sessions).

## Flow
```mermaid
flowchart LR
A[free -h: how much available?] --> B[ps --sort=-%mem: who?]
B --> C[log / 2nd snapshot: growing = leak?]
C --> D[Save evidence to ticket]
D --> E[Restart service / kill]
E --> F[free -h: verify]
F --> G[Escalate to devs with findings]
```

See [incident-report.md](incident-report.md) for the ticket.
