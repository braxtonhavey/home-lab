# 01 – Proxmox VE

Proxmox VE is the hypervisor: it takes over the whole server and runs everything else as virtual machines. Managed from a browser at `https://192.168.0.253:8006`.

## Install

1. BIOS: enabled **Intel VT-x** and **VT-d** (needed for VMs and later GPU passthrough); set **Restore on AC power loss = On**.
2. Wrote the Proxmox VE ISO to USB and installed to the NVMe SSD (wiped the existing drive).
3. Network during install: IP `192.168.0.253`, gateway `192.168.0.254`.

## Repositories and updates

- **Node → Updates → Repositories → Add → No-Subscription.**
- Disabled both `enterprise.proxmox.com` repos (`pve-enterprise`, `ceph`), which return 401 errors without a paid key.
- **Updates → Refresh → Upgrade**, rebooted after kernel updates.

### Subscription pop-up (optional, cosmetic)

```bash
sed -Ezi.bak "s/(function\(orig_cmd\) \{)/\1\n\torig_cmd\(\);\n\treturn;/g" /usr/share/javascript/proxmox-widget-toolkit/proxmoxlib.js && systemctl restart pveproxy.service
```

- Gets undone when `proxmox-widget-toolkit` updates. Re-run when the pop-up returns.
- Undo: `apt reinstall proxmox-widget-toolkit && systemctl restart pveproxy`.

## Default storage and networking

| Name | What it is | Used for |
|---|---|---|
| `local` | Folder on the SSD (`/var/lib/vz`) | ISOs, container templates, backups |
| `local-lvm` | Thin-provisioned LVM pool on the SSD | VM disks |
| `vmbr0` | Virtual switch bridged to the ethernet port | Puts VMs on the home LAN |

**Note:** backups to `local` live on the same SSD as the VMs. Will add an external disk (NAS) as backup storage for real protection.

## VM 100 – `docker`

Ubuntu ISO uploaded to `local` with **SHA-256 checksum verification** against Ubuntu's `SHA256SUMS`.

| Tab     | Setting                           | Value                          | Why                                                                          |
| ------- | --------------------------------- | ------------------------------ | ---------------------------------------------------------------------------- |
| General | Start at boot                     | On                             | Comes back after power loss                                                  |
| General | HA / Resource pool                | Off / blank                    | Single node, no cluster                                                      |
| System  | Machine                           | `q35`                          | Native PCIe; better for GPU passthrough                                      |
| System  | BIOS                              | `OVMF (UEFI)`                  | Modern firmware; GPUs expect UEFI                                            |
| System  | EFI storage                       | `local-lvm`                    | Tiny disk holding UEFI settings                                              |
| System  | Pre-enroll keys                   | Off                            | Secure Boot would block the unsigned NVIDIA driver                           |
| System  | SCSI controller                   | VirtIO SCSI single             | Fastest; one controller per disk enables IO thread                           |
| System  | Qemu Agent                        | On                             | Shows IP, clean shutdowns, consistent backups                                |
| System  | TPM                               | Off                            | Not needed for Ubuntu                                                        |
| Disks   | Bus                               | SCSI 0, `local-lvm`, 64 GB     | Paravirtualized, supports discard                                            |
| Disks   | Cache / Async IO                  | No cache / io_uring            | Defaults; guest does its own caching                                         |
| Disks   | Discard, SSD emulation, IO thread | On                             | Return freed space, SSD tuning, separate IO thread                           |
| Disks   | Backup                            | On                             | Include OS disk in backups                                                   |
| CPU     | Cores / Type                      | 4 / `host`                     | Exposes real CPU features; no extra flags needed                             |
| Memory  | RAM                               | 16384 MiB, ballooning + KSM on | Ballooning reports real usage                                                |
| Network | Bridge / Model                    | `vmbr0` / VirtIO               | Firewall box checked (Proxmox firewall itself not enabled), not disconnected |

### Additional disk: temporary media (scsi1)

- **Hardware → Add → Hard Disk:** SCSI 1, `local-lvm`, **100 GB**, discard/IO thread/SSD emulation on, **Backup off** (keeps media out of backups).
- Will be replaced by an NFS share from the NAS, once ordered.

> **Lesson learned:** `ls /dev/disk/by-id/` on the host lists the boot SSD and its LVM volumes. Never pass any of those to a VM. A new physical HDD will show up as `ata-...`.
