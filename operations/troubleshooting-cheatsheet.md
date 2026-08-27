# Troubleshooting Cheatsheet

## Service Does Not Open by Hostname

```bash
nslookup <host> <pihole-ip>
curl -vk https://<host>
```

If DNS fails:

- Pi-hole
- wildcard config
- FTL reload
- client DNS

If DNS works but HTTPS fails:

- Traefik
- middleware
- certificate
- backend

## Docker Service Down

```bash
docker ps -a
docker logs <container>
docker compose ps
docker compose logs
```

Check mounts:

```bash
docker inspect <container>
findmnt
```

## Host Disk Full

```bash
df -h
df -i
sudo du -xhd1 / | sort -h
docker system df
```

## Proxmox Thin Pool High

```bash
pvesm status
lvs
```

Inside affected guest:

```bash
df -h
sudo fstrim -av
```

Then re-check:

```bash
lvs
```

## VM Shows `io-error`

First suspect storage availability / capacity.

Check:

```bash
pvesm status
lvs
qm config <vmid>
```

Avoid assuming the VM itself is corrupted.

## CIFS Mount Missing

```bash
findmnt
mount
dmesg | tail -50
journalctl -b | grep -i cifs
```

Check:

- NAS reachable
- DNS
- credentials
- share path
- mount options

## Network Interface Problem

```bash
ip addr
ip route
ethtool <interface>
dmesg | grep -i <interface>
journalctl -b | grep -i link
```

## Previous Boot Failure

```bash
journalctl -b -1
journalctl -k -b -1
```

## Systemd Service

```bash
systemctl status <service>
journalctl -u <service>
```

## DNS vs Network Test

```bash
ping <gateway-ip>
ping <remote-ip>
nslookup example.com <pihole-ip>
curl -I https://example.com
```

This separates:

- local routing
- internet routing
- DNS
- HTTPS/application connectivity
