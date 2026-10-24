# Incident Report: Camera dashboard unreachable from the client network

| Field | Details |
|---|---|
| **Summary** | Users on the client network (10.0.1.0/24) report that the camera dashboard at `http://10.0.2.20:8080` doesn't load ("Connection refused"). It loads fine locally on the server. |
| **Impact** | All users on 10.0.1.0/24 lost access to the camera dashboard. The server and network were otherwise healthy. |
| **Severity** | P1: the service was fully unavailable to all of its users. |
| **Detection** | User reports, plus the developer's statement: "It works fine, I tested it on the server." |
| **Findings** | **Saw:** `curl -v http://10.0.2.20:8080` from the client failed instantly with "Connection refused" (an answer, not a timeout, so the network path works).<br>**Tried:** `ss -tlnp` on the server showed the app listening on `127.0.0.1:8080` only. `curl 127.0.0.1:8080` on the server worked; `curl 10.0.2.20:8080` was refused.<br>**Suspected:** the app is bound to loopback only. |
| **Root cause** | The dashboard app was listening on `127.0.0.1:8080` (loopback only), so it accepted connections only from the server itself. Connections from the client network to `10.0.2.20:8080` were refused (TCP RST). |
| **Fix** | Stopped the old process and restarted the app bound to all interfaces: `python3 -m http.server 8080 --bind 0.0.0.0 --directory /srv/labweb`. |
| **Verified** | `ss -tlnp` shows `0.0.0.0:8080`. `curl http://10.0.2.20:8080` from the client returns the "Camera dashboard" page. |
| **Next steps** | 1. The fix was a manual restart, so after a reboot or redeploy the app would bind to 127.0.0.1 again. Escalated to the **app owner / dev team** to make the bind address permanent in the app's config or service unit (not a network issue).<br>2. Add an external health check from the client side (not only from the server itself).<br>3. Add "listening on the right address" (`ss -tlnp`) to the deployment checklist. |
