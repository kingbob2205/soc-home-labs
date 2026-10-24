# Incident Report – Server 10.0.2.20 cannot reach client network

**Summary:** Server 10.0.2.20 cannot communicate with client network 10.0.1.0/24 (reported by monitoring).

**Impact:** All hosts on 10.0.1.0/24 could not reach server 10.0.2.20, from detection until fix. Local traffic within 10.0.2.0/24 was unaffected.

**Severity:** P1 – every host on 10.0.1.0/24 lost access to the server, no workaround.

**Detection:** Reported by the monitoring team.

**Findings (saw / tried / suspect):**
- `ip -br addr` → eth0 UP, 10.0.2.20/24 (link and IP OK)
- `ping -c 3 10.0.2.1` (gateway) → 0% loss (local network OK)
- `ping -c 3 10.0.1.10` → "Network is unreachable", returned by the server itself
- `ip route` / `ip route get 10.0.1.10` → only the local route, **no default route**
- **Suspected:** missing default gateway on the server

**Root cause:** No default gateway configured on server 10.0.2.20 (confirmed via `ip route`). Why it was missing is unknown – likely a config change or a manual route lost on reboot.

**Fix:** `ip route add default via 10.0.2.1`

**Verified:** `ip route` shows `default via 10.0.2.1`; `ping 10.0.1.10` 0% loss (ttl=126, one hop).

**Next steps:**
- Network/sysadmin team: make the gateway **permanent** in the server's network config, or fix DHCP to provide it (the `ip route add` fix is lost on reboot).
- Review change history to find **why** the route disappeared.
- Add a monitoring check that alerts if the server loses its default route or can't reach its gateway.
