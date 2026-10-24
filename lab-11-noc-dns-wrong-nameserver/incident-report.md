# Incident Report: Camera dashboard fails by name (wrong / unreachable nameserver)

| Field | Details |
|---|---|
| **Summary** | Users can't open `http://camera.lab.local:8080`. Direct IP `http://10.0.2.20:8080` works. |
| **Impact** | Anyone using the hostname lost access; IP access still worked. |
| **Severity** | P2 (service reachable by IP; name path broken for users). |
| **Detection** | User reports; confirmed by IP-vs-name curl. |
| **Findings** | **Saw:** curl by IP returned 200 + dashboard; curl by name failed with "Resolving timed out".<br>**Tried:** `cat /etc/resolv.conf` showed `nameserver 10.0.1.99`. `dig camera.lab.local` timed out to 10.0.1.99#53. `dig @10.0.1.1 camera.lab.local` returned A `10.0.2.20`.<br>**Suspected:** client pointed at a dead nameserver; DNS service on the router is healthy. |
| **Root cause** | Client `/etc/resolv.conf` used unreachable nameserver `10.0.1.99` instead of `10.0.1.1`. Name lookups timed out; IP access skips DNS so it still worked. |
| **Fix** | Set `nameserver 10.0.1.1` in the client's resolv.conf. |
| **Verified** | `dig` returns `10.0.2.20`; curl by name connects to `camera.lab.local (10.0.2.20)` and returns the dashboard. |
| **Next steps** | Make nameserver permanent via DHCP/network config; monitor name resolution (not only IP); document the correct DNS IPs. |
