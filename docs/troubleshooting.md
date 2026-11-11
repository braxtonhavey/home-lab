# Troubleshooting Log

Problems I've hit, what caused them, and how I fixed them.

---

### Pi-hole won't start: `failed to bind host port 0.0.0.0:53/tcp: address already in use`

**Date:** 2026-11
**Cause:** Ubuntu's `systemd-resolved` still listening on port 53; the edit to `resolved.conf` hadn't taken effect.
**Diagnose:** `sudo ss -tulpn | grep ':53 '` showed `systemd-resolve`.
**Fix:** Drop-in file `/etc/systemd/resolved.conf.d/nostub.conf` with `DNSStubListener=no`, re-linked `/etc/resolv.conf`, restarted `systemd-resolved`. See [02 – Docker VM](02-docker-vm.md#freeing-port-53-for-pi-hole).

---

### Pi-hole `(unhealthy)`, empty PORTS column, admin page won't load

**Date:** 2026-11
**Cause:** Container was created during the earlier port 53 failure and never published its ports.
**Fix:** `docker compose down && docker compose up -d`. Data folders survive `down`.

---

### Template

```
### <Symptom / error message>
**Date:**
**Cause:**
**Diagnose:**
**Fix:**
```
