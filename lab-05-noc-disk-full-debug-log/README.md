# Lab 05 – NOC: Disk full, uploads failing (runaway debug log)

**Role:** NOC / Support Tier 1 · **Platform:** Linux · **Skills:** `df`, `du`, `ls`, `tail`, `grep`, safe cleanup with `truncate`, ticket write-up

## Scenario
> 🚨 Ticket: "Customers can't upload reports. The upload page shows an error."

The app `reportd` stores uploads and logs in `/srv/appdata` (a 200 MB volume). A debug log was left enabled, grew until the volume hit 100%, and the app could no longer write uploads.

## Lab setup (for reproducing)
```bash
sudo mkdir -p /srv/appdata
sudo mount -t tmpfs -o size=200M tmpfs /srv/appdata        # small lab volume
sudo mkdir -p /srv/appdata/uploads /srv/appdata/logs
for i in 1 2 3 4 5; do sudo head -c 2M /dev/urandom | sudo tee /srv/appdata/uploads/report_$i.pdf >/dev/null; done
yes "DEBUG reportd cache_sync verbose_mode=ON" | sudo dd of=/srv/appdata/logs/app-debug.log bs=1M iflag=fullblock   # fills the volume
# plus a small loop script (/usr/local/bin/reportd) that writes uploads and logs every 5 seconds
```

## Investigation
| # | Command | What it showed | Why it matters |
|---|---|---|---|
| 1 | `df -h` | `/srv/appdata` 200M/200M, **100%**, 0 available | Confirms disk full |
| 2 | `sudo du -sh /srv/appdata/*` | `logs` 190M, `uploads` 10M | Finds the big folder (customer data is small) |
| 3 | `ls -lh /srv/appdata/logs` | `app-debug.log` **190M**, `app.log` 4K | Finds the file: a debug log |
| 4 | `tail -n 5 /srv/appdata/logs/app.log` | `cannot write .../latest_report.tmp: No space left on device` every 5 s | Links the full disk to the customer error |
| 5 | `grep -m1 ERROR /srv/appdata/logs/app.log` | First error at 16:25 | Outage start time for the ticket |

## Fix
```bash
sudo cp -a /srv/appdata/logs/app-debug.log /root/app-debug.log.bak   # backup to a disk WITH free space
sudo ls -lh /root/app-debug.log.bak                                 # confirm 190M
sudo truncate -s 0 /srv/appdata/logs/app-debug.log                   # empty it, app keeps running
```

## Verification
```bash
df -h /srv/appdata                          # 6% used
cat /srv/appdata/uploads/latest_report.tmp  # fresh timestamp each run = app writing again
tail -n 3 /srv/appdata/logs/app.log         # no new errors
```

## Key lessons
- `df` = how full each **disk** is; `du` = **which folder/file** uses the space.
- **`truncate` vs `rm`:** `rm` on a file a process still has open leaves a "ghost file", so the space isn't freed until the app restarts (`sudo lsof +L1` finds them). `truncate -s 0` frees it immediately.
- A backup must go to a **different** disk that has free space.
- Never delete what you don't understand: customer uploads are data, a debug log is noise.
- Disk vs memory: disk = storage, memory = RAM. MB = megabytes (file sizes), Mb = megabits (network speeds).

## Flow
```mermaid
flowchart LR
A[df -h: which disk?] --> B[du -sh: which folder?]
B --> C[ls -lh: which file?]
C --> D[tail: what's the error?]
D --> E[backup to another disk + truncate]
E --> F[df -h + app check: verify]
```

See [incident-report.md](incident-report.md) for the ticket.
