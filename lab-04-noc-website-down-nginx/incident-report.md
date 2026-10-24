# Incident report: website down (nginx on the wrong port)

| Field | Details |
|---|---|
| **Summary** | Website unreachable for customers on the web server (nginx); port 80 not responding |
| **Impact** | All customers unable to reach the website (HTTP port 80), 20:40–21:48 Israel time (~68 min) |
| **Severity** | P1: full outage, all customers, no workaround |
| **Detection** | Customer ticket (no monitoring alert fired) |
| **Root cause** | nginx was configured to listen on port 8080 instead of 80 in `/etc/nginx/sites-enabled/default` (lines 22–23), so nothing answered on the standard HTTP port |

## Findings

1. `service nginx status`: nginx running.
2. `curl -I localhost`: connection refused on port 80.
3. `ss -tulpn | grep nginx`: nginx listening on `0.0.0.0:8080` and `[::]:8080` instead of 80.
4. `grep -n listen /etc/nginx/sites-enabled/default`: lines 22–23 set to `listen 8080`.

## Fix

1. Backed up the config: `cp -a /etc/nginx/sites-enabled/default /root/default.bak`
2. Edited lines 22–23 from `listen 8080` to `listen 80` with `nano`.
3. Tested the config: `nginx -t` (syntax ok, test successful).
4. Restarted nginx: `service nginx restart` (`systemctl restart nginx` on systemd hosts).

## Verified

- `curl -I localhost` returned `HTTP/1.1 200 OK`.
- `ss -tulpn` showed nginx listening on `0.0.0.0:80` and `[::]:80`.

## Next steps

1. **Least privilege:** restrict who can edit `/etc/nginx/` and run `sudo` on the web server.
2. **Change alerting:** alert on any change under `/etc/nginx/` (auditd rule or file integrity monitoring such as AIDE).
3. **Uptime monitoring:** add an external HTTP check on port 80 (e.g. every minute) so the team detects an outage before customers do.
4. **Find who changed it:** review sudo and auth logs (`journalctl _COMM=sudo`, `/var/log/auth.log`) for the time of the change.
5. **Change management:** config changes go through review, and `nginx -t` runs before every reload.
