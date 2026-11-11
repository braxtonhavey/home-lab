# 06 – Uptime Kuma *(in progress)*

Self-hosted monitoring that alerts me when a service goes down, ideally before my family notices.

`http://192.168.0.252:3001`

## Deployment

- Defined in [docker-compose.yml](../docker/docker-compose.yml).
- Data in `~homelab/uptime-kuma`
- [ ] Admin account created

## Monitors

| Name             | Type     | Target                                           | Status |
| ---------------- | -------- | ------------------------------------------------ | ------ |
| Proxmox          | HTTP(s)  | `https://192.168.0.253:8006` (ignore TLS errors) | ☐      |
| Pi-hole DNS      | DNS      | Resolve `google.com` via `192.168.0.252`         | ☐      |
| Pi-hole admin    | HTTP     | `http://192.168.0.252/admin/`                    | ☐      |
| Jellyfin         | HTTP     | `http://192.168.0.252:8096/health`               | ☐      |
| Router           | Ping     | `192.168.0.254`                                  | ☐      |
| Internet         | Ping     | `1.1.1.1`                                        | ☐      |
| Minecraft Server | TCP Port | 192.168.0.252:25565                              | ☐      |

## Notifications

- [ ] Notification method chosen and tested

## Notes

> For now, Uptime Kuma runs on the same VM it monitors, so it can't alert if the whole VM or host is down. In the future, a second instance on separate hardware will be implemented.
