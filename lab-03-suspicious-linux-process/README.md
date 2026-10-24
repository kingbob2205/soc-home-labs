# Lab 03 – Suspicious Process on a Linux Host

**Scenario:** A Linux host may be compromised. Find what is running that shouldn't be, understand it, preserve evidence, stop it, and write the incident report.

**Skills shown:** Linux process triage · `ps aux` analysis · hidden files in `/tmp` · C2 beaconing pattern · evidence preservation · containment and cleanup · incident timeline

## What I found

A hidden directory `/tmp/.cache-update/` held a script `payload.sh` running in the background (PID 52021, user `box`). It wrote a "beacon" line with a timestamp to `out.log` every 30 seconds, the fixed-interval pattern of malware calling home to a command-and-control (C2) server.

## Investigation steps (how to reproduce)

```bash
# 1. Look for unusual processes
ps aux --sort=start_time | tail -20
ps aux | grep -v grep | grep -i tmp

# 2. Inspect the suspicious process
ls -la /proc/52021/cwd        # where it runs from
cat /proc/52021/cmdline | tr '\0' ' '
ls -la /tmp/.cache-update/    # hidden directory
cat /tmp/.cache-update/payload.sh
tail /tmp/.cache-update/out.log   # beacon every 30 s

# 3. Preserve evidence BEFORE touching anything (cp -a keeps timestamps)
mkdir ~/evidence-case1
cp -a /tmp/.cache-update/* ~/evidence-case1/

# 4. Contain: confirm the PID, then stop it
ps -p 52021 -o pid,user,stat,cmd
kill 52021
ps aux | grep payload        # verify it's gone
tail -1 ~/evidence-case1/out.log

# 5. Clean up
rm -rf /tmp/.cache-update
```

## Timeline (Israel time)

| Time | Event |
|------|-------|
| 19:53:15 | First beacon, activity starts |
| 20:25 | Detected during process review (`ps aux`) |
| ~20:30 | Process stopped (last beacon 20:30:15) |
| | **Dwell time:** about 32 minutes |

## Evidence

- [`evidence/payload.sh`](evidence/payload.sh): the beacon script (harmless lab simulation)
- [`evidence/out.log`](evidence/out.log): 71 beacon lines, one every 30 seconds
- [Incident report](incident-report.md)

## Lessons learned

- `STAT S` in `ps` means *sleeping* (waiting), not dead; a beacon spends most of its time asleep.
- Always confirm the PID with `ps -p <PID>` before `kill`.
- Use `cp -a`, not `cp -r`, so evidence keeps its original timestamps.
- Reports need exact details: full path, PID, user.

## Recommendations

- Mount `/tmp` with `noexec`
- Monitor new files in `/tmp` with auditd or AIDE
- SIEM/EDR rule for fixed-interval (beaconing) activity
