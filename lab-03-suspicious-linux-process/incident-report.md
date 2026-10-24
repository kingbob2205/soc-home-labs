# Incident Report – Suspicious Process on Linux Host

1. **Detection:** Hidden script `/tmp/.cache-update/payload.sh` found running as PID 52021 under user `box`.
2. **Behavior:** Infinite loop writing "beacon" + current time to `/tmp/.cache-update/out.log` every 30 seconds. Consistent with C2 beaconing.
3. **Timeline (IDT):**
   - 19:53:15 – first beacon (activity start)
   - 20:25 – detected via process review (`ps aux`)
   - ~20:30 – process terminated (last beacon 20:30:15)
   - Dwell time: ~32 minutes
4. **Actions taken:** Evidence preserved to `~/evidence-case1`, process stopped with `kill`, removal verified via `ps` and log, directory deleted.
5. **Recommendations:**
   - Mount `/tmp` with `noexec`
   - Monitor new files in `/tmp` with AIDE/auditd
   - SIEM/EDR rule for fixed-interval activity (beaconing)

Lesson learned: use `cp -a` (not `cp -r`) to preserve original timestamps when collecting evidence.
