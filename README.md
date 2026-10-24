# SOC Home Labs

Hands-on blue-team labs I run in my own Linux lab while preparing for a **SOC / NOC Tier 1** role. Each lab is written like a real ticket: scenario, how I detected it, the commands I used, a timeline, the response, and an incident report.

Based partly on the [30-Days SOC Challenge](https://github.com/0xrajneesh/30-Days-SOC-Challenge-Beginner).

| # | Lab | Skills | Status |
|---|-----|--------|--------|
| 03 | [Suspicious Linux process (hidden beacon)](lab-03-suspicious-linux-process/) | `ps` triage, process analysis, evidence preservation, timeline, containment | ✅ Done |
| 04 | [NOC: website down, nginx on the wrong port](lab-04-noc-website-down-nginx/) | service triage, `curl`, `ss`, config review, `nginx -t`, ticket write-up | ✅ Done |
| 05 | [NOC: disk full, uploads failing](lab-05-noc-disk-full-debug-log/) | `df`, `du`, `ls`, `tail`, `grep`, `truncate` vs `rm`, ticket write-up | ✅ Done |
| 06 | [NOC: high CPU, runaway script](lab-06-noc-high-cpu-runaway-script/) | `uptime` load, `top`, `ps`, script review, `kill` vs `kill -9`, ticket write-up | ✅ Done |
| 07 | [NOC: high RAM, memory leak](lab-07-noc-high-ram-memory-leak/) | `free -h`, `ps --sort=-%mem`, `pgrep`, leak vs big app, restart vs escalate, ticket write-up | ✅ Done |
| 08 | [NOC: missing default gateway](lab-08-noc-missing-default-gateway/) | netns topology, `ip -br addr`, `ping`, `ip route`, `ip route get`, TTL hops, ticket write-up | ✅ Done |
| 09 | [NOC: service bound to localhost](lab-09-noc-service-bound-to-localhost/) | `curl -v`, refused vs timeout, `ss -tlnp`, 127.0.0.1 vs 0.0.0.0, ticket write-up | ✅ Done |
| 10 | [NOC: firewall DROP, site hangs](lab-10-noc-firewall-drop-timeout/) | timeout vs refused, `ping`, `tcpdump` flags, `iptables -L/-D`, ticket write-up | ✅ Done |
| 11 | [NOC: DNS wrong nameserver](lab-11-noc-dns-wrong-nameserver/) | IP vs name, resolv.conf, dig vs dig @NS, A record vs nameserver, ticket | ✅ Done |
| 13 | Persistence through cron | cron review, persistence hunting | ⏳ Next |
| – | Windows Event IDs (4624 / 4625 / 4688 / 4720 / 4672) | Windows logging, authentication analysis | ⏳ Planned |

Main project: [soc-ssh-detection-lab](https://github.com/kingbob2205/soc-ssh-detection-lab), SSH brute-force detection with Splunk and automated response with Fail2ban.
