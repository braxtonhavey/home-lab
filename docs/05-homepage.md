# 05 – Homepage

Dashboard linking to every lab service. `http://192.168.0.252:3000`

## Deployment

- Defined in [docker-compose.yml](../docker/docker-compose.yml).
- Config files: `~/homelab/homepage/config/` (YAML; changes apply on page refresh).
- `HOMEPAGE_ALLOWED_HOSTS` set to the address I browse to (required to avoid a host validation error).
- Docker socket mounted at `/var/run/docker.sock:ro` for the container status/resource widgets.

## Layout

| Group          | Service          | Link                        | Widget                       |
| -------------- | ---------------- | --------------------------- | ---------------------------- |
| Infrastructure | Proxmox          | https://192.168.0.253:8006  | Yes                          |
| Infrastructure | Uptime Kuma      | http://192.168.0.252:3001   | Yes                          |
| Network        | Pi-hole          | http://192.168.0.252/admin/ | Yes                          |
| Media          | Jellyfin         | http://192.168.0.252:8096   | Yes                          |
| Games          | Minecraft Server | 192.168.0.252:25565         | Yes (resource usage monitor) |

## Widget credentials

Currently, widgets do not use credentials; Data comes from the Docker socket and status checks.
## Security Notes

### Known risk: Docker socket access

The compose file mounts the Docker socket with `:ro`, but **`:ro` does not make the Docker API read-only.** It only prevents the socket file itself from being replaced. Anything that can talk to the socket can send any Docker API request: start containers, mount the host filesystem, run privileged containers. Access to the Docker socket is effectively **root on the VM**.

| Risk             | If Homepage were compromised, an attacker could take over the Docker VM                                                                                                                                                          |
| ---------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Likelihood       | Low: Homepage is LAN-only, not exposed through playit.gg                                                                                                                                                                         |
| Accepted because | Home lab, trusted LAN, convenience of container widgets                                                                                                                                                                          |
| Future fix       | Put **docker-socket-proxy** (e.g. `tecnativa/docker-socket-proxy`) between Homepage and the socket, allowing only read endpoints (`CONTAINERS=1`, everything else denied), and point Homepage at the proxy instead of the socket |

