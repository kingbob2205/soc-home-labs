# Incident Report – Report uploads failing (disk full)

| Field | Details |
|---|---|
| **Summary** | Customers got an error when trying to upload reports. |
| **Impact** | All customers couldn't upload reports from 16:25 to 17:04 Israel time (~40 minutes). |
| **Severity** | P1: core function broken for all customers, no workaround. |
| **Detection** | Customer ticket at 16:06; no monitoring alert fired. |
| **Findings** | 1. `df -h`: `/srv/appdata` at 100% disk usage, 0 available.<br>2. `du -sh /srv/appdata/*`: `logs` 190M, `uploads` 10M.<br>3. `ls -lh /srv/appdata/logs`: `app-debug.log` 190M, `app.log` 4K.<br>4. `tail -n 5 app.log`: `reportd` failing every 5 seconds with "No space left on device". |
| **Root cause** | Debug logging was left enabled on `reportd` (likely forgotten after troubleshooting). `app-debug.log` grew to 190M and filled `/srv/appdata` to 100%, so `reportd` could not write uploads. |
| **Fix** | 1. Backed up the log to the root disk: `sudo cp -a /srv/appdata/logs/app-debug.log /root/app-debug.log.bak`<br>2. Confirmed the backup (190M): `sudo ls -lh /root/app-debug.log.bak`<br>3. Emptied the log without stopping the app (truncate instead of rm, because the file was still open): `sudo truncate -s 0 /srv/appdata/logs/app-debug.log` |
| **Verified** | `df -h` shows 6% used; `latest_report.tmp` shows a fresh timestamp each time; no new errors in `app.log` since 16:28. |
| **Next steps** | Turn off debug logging on `reportd`; configure `logrotate` (rotate, compress, delete old logs); add disk-usage alerts at 80%/90% so the NOC knows before customers do. |
