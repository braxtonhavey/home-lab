# 03 – Pi-hole

Network-wide DNS ad blocking. Admin page: `http://192.168.0.252/admin/`

## How Pi-hole Works

Devices ask Pi-hole to look up domain names. Pi-hole answers `0.0.0.0` for domains on its blocklists and forwards everything else upstream. It blocks **domains**, not full URLs, so it can't block ads served from the same domain as the content (e.g., YouTube video ads).

## Deployment

Defined in [docker-compose.yml](../docker/docker-compose.yml). Ports 53/tcp+udp (DNS) and 80/tcp (admin UI). Settings persist in `~/homelab/pihole/etc-pihole`.

## Testing

1. Set **only my PC's** DNS to `192.168.0.252`.
2. `nslookup doubleclick.net 192.168.0.252` → should return `0.0.0.0`.
3. Watch the **Query Log**; blocked queries show in red.
4. Test sites: [d3ward Ad Block Test](https://d3ward.github.io/toolz/adblock), ad-heavy sites (recipe, weather, news).
5. Compare using **Disable blocking** (5 min) on the dashboard.

### Mobile devices

- **iPhone:** Wi-Fi (i) → Configure DNS → Manual. Turn off *Limit IP Address Tracking* / iCloud Private Relay for home Wi-Fi.
- **Android:** requires a static IP to set DNS manually; also set **Private DNS** to Off/Automatic.

## Blocklists

| List                                                         | Purpose              |
| ------------------------------------------------------------ | -------------------- |
| Default (StevenBlack)                                        | Base ads/trackers    |
| [HaGeZi Multi PRO](https://github.com/hagezi/dns-blocklists) | Broader ads/trackers |

Added via **Lists**, then **Tools → Update Gravity**. If needed planning to add one list at a time so breakage is traceable.

### Manual Deny List

| Domain                       | Type | Reason |
| ---------------------------- | ---- | ------ |
| For future denying if needed | -    | -      |

**Limitation:** Pi-hole blocks where a redirect *goes*, not the script that triggers it. Paired with uBlock Origin, Brave, and/or browser pop-up settings.

## Family rollout

1. Router → **Advanced → Network → DHCP Server**.
2. Primary DNS = `192.168.0.252`. Secondary blank. **Never** a public resolver like 8.8.8.8, or devices will bypass Pi-hole.
3. Devices pick it up on reconnect.

## ⚠️ Rollback (if the server is down, the internet "breaks")

1. Log into router at `192.168.0.254`.
2. Advanced → Network → DHCP Server → **clear the Primary DNS field**, save.
3. Reconnect devices / restart them.

Planned future improvement: second Pi-hole on separate server for redundancy.
