# 02 – Docker VM

Ubuntu Server 26.04.1 LTS (`live-server-amd64`) running in Proxmox VM 100. Hosts all containers.

## Ubuntu install choices

| Screen         | Choice                                                                                                |
| -------------- | ----------------------------------------------------------------------------------------------------- |
| Network        | Manual IPv4: subnet `192.168.0.0/24`, address `192.168.0.252`, gateway `192.168.0.254`, DNS `1.1.1.1` |
| Storage        | Entire 64 GB disk; expanded `ubuntu-lv` to max size                                                   |
| Profile        | Hostname `docker`, non-default admin username, unique password                                        |
| Ubuntu Pro     | Skipped                                                                                               |
| SSH            | Installed OpenSSH server                                                                              |
| Featured snaps | **None** (snap Docker conflicts with the official install)                                            |

## Post-install

```bash
sudo apt update && sudo apt full-upgrade -y
sudo apt install qemu-guest-agent -y
sudo reboot
```

Removed the ISO from the CD/DVD drive in Proxmox afterward.

## Temporary media disk → `/mnt/media`

```bash
lsblk                         # sdb ≈ 100G (sda is the OS disk)
sudo mkfs.ext4 /dev/sdb
sudo mkdir -p /mnt/media
sudo blkid /dev/sdb           # copy UUID
```

`/etc/fstab`:
```
UUID=<uuid>  /mnt/media  ext4  defaults,nofail  0  2
```

```bash
sudo systemctl daemon-reload && sudo mount -a
sudo mkdir -p /mnt/media/Movies /mnt/media/Shows
sudo chown -R 1000:1000 /mnt/media
```

`nofail` lets the VM boot even if the disk is missing.

## Docker Engine

Installed from Docker's official apt repository ([docs](https://docs.docker.com/engine/install/ubuntu/)), then:

```bash
sudo usermod -aG docker $USER   # log out/in afterward
```

## Freeing port 53 for Pi-hole

Ubuntu's `systemd-resolved` listens on port 53. Disabled its stub listener with a drop-in file (more reliable than editing `resolved.conf`):

`/etc/systemd/resolved.conf.d/nostub.conf`
```ini
[Resolve]
DNSStubListener=no
```

```bash
sudo ln -sf /run/systemd/resolve/resolv.conf /etc/resolv.conf
sudo systemctl restart systemd-resolved
sudo ss -tulpn | grep ':53 '   # should print nothing
```

## Compose project

All services live in `~/homelab/docker-compose.yml` with data in subfolders. See [docker-compose.yml](../docker/docker-compose.yml).

```bash
cd ~/homelab
docker compose up -d      # apply changes
docker compose down       # remove containers (data folders kept)
docker ps                 # status
docker logs <name> --tail 50
```
