# Incident Report – Server slow, dashboards timing out (high CPU)

| Field | Details |
|---|---|
| **Summary** | The server was very slow and customer dashboards kept timing out. |
| **Impact** | Customers using dashboards, from about 19:57 to 20:30 Israel time. |
| **Severity** | P2: major slowdown, service still partially working. |
| **Detection** | Customer ticket at 19:58; no CPU alert fired. |
| **Findings** | 1. `uptime`: load rising (1.36, then 2.09, then 2.35) on 8 cores.<br>2. `top`: PIDs 98676 and 98678 (root) at 100% CPU each.<br>3. `ps -o cmd,etime`: `/usr/local/bin/logcompress`, running ~22 min.<br>4. `cat` of the script: `while :; do :; done` (endless loop). |
| **Root cause** | The `/usr/local/bin/logcompress` script has an endless loop with no `sleep` or exit condition. Two copies running as root each used 100% of a CPU core, slowing the server. |
| **Fix** | Confirmed it was a stuck maintenance script (not a critical service), then stopped both copies with `sudo kill 98676 98678` (SIGTERM). |
| **Verified** | `ps -p` shows both processes gone; the 1-minute load dropped (2.35 to 1.67); no process at 100% in `top`. |
| **Next steps** | 1. Fix the script (exit condition / `sleep`) and send it to the owner (devs).<br>2. Check what launches it (`sudo crontab -l`, `/etc/cron.d/`) so it doesn't restart.<br>3. Add a CPU alert (e.g. over 80% for 5 min) so the NOC knows before customers do. |
