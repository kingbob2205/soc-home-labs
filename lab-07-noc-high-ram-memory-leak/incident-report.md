# Incident Report – High memory usage (sessioncache memory leak)

| Field | Details |
|---|---|
| **Summary** | Server memory alert climbing; risk of apps crashing (OOM). |
| **Impact** | No customer outage yet; available RAM down to 2.9Gi of 15Gi. Caught before the OOM killer acted. |
| **Severity** | P2: high risk of outage, service still working. |
| **Detection** | Memory alert (ticket at 20:40). |
| **Findings** | 1. `free -h`: 2.9Gi available of 15Gi.<br>2. `ps aux --sort=-%mem`: `python3 /opt/sessioncache/sessioncache.py` (root) at 18.8% MEM, ~3 GB RSS.<br>3. App log: `total_mb` grew from 50 to 3000 and never dropped. |
| **Root cause** | Memory leak in `sessioncache`: it caches session batches (+50 MB every 3 s) and never frees old ones. |
| **Fix** | Evidence saved to the ticket, then stopped the app: `sudo kill <PID>` (production: `systemctl restart sessioncache`; team warned that sessions may be logged out). |
| **Verified** | `pgrep` shows the process gone; `free -h` available 2.9Gi to 5.8Gi (~3 GiB freed). |
| **Next steps** | 1. Escalate to the devs: add session expiry / evict the oldest entries (fix the leak, not just the cap).<br>2. Until fixed, scheduled restart per runbook + memory limit on the service.<br>3. Alert on available RAM under 20% and on per-process memory growth. |
