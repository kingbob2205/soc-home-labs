# Incident Report: Camera dashboard hangs (firewall DROP on port 8080)


| Field | Details |
|---|---|
| **Summary** | Users on the client network (10.0.1.0/24) report that the camera dashboard at `http://10.0.2.20:8080` loads forever and then times out. It worked an hour earlier. |
| **Impact** | All users on 10.0.1.0/24 lost access to the dashboard. Other traffic to the server (ping) was unaffected. |
| **Severity** | P1: the service was fully unavailable to all of its users. |
| **Detection** | User reports. |
| **Findings** | **Saw:** `curl -v --max-time 5` from the client timed out after 5002 ms (no answer, not refused).<br>**Tried:** `ping 10.0.2.20` gave 3/3 replies, so routing and the host are fine. `tcpdump -n -i eth0 port 8080` on the server showed the client's SYN `[S]` arriving 4 times (same seq, retransmits) with no SYN-ACK or RST going back. The app was known to be listening on `0.0.0.0:8080`.<br>**Suspected:** a host firewall on the server silently dropping port 8080. |
| **Root cause** | `iptables -L INPUT -n --line-numbers` showed rule 1: `DROP tcp 0.0.0.0/0 0.0.0.0/0 tcp dpt:8080`. Every TCP packet to port 8080 was silently discarded, which caused the timeouts. ICMP was not affected, so ping still worked. |
| **Fix** | Deleted the rule: `iptables -D INPUT 1`. |
| **Verified** | `iptables -L INPUT` shows no DROP rule. The client's `curl` returns the "Camera dashboard" page. `tcpdump` shows a full exchange: `[S]`, `[S.]`, `[.]`, data `[P.]` (HTTP 200 OK) and `[F.]` close, 12 packets in both directions. |
| **Next steps** | 1. Find who added the rule and why (change history, shell history, config management); it may have been intended for a different port or host.<br>2. Note that `iptables -D` is not persistent: make sure the saved firewall config (iptables-save / nftables / ufw) doesn't restore the rule on reboot.<br>3. Firewall changes should go through change management with a ticket.<br>4. Add an external port check for 8080 from the client network to monitoring. |
