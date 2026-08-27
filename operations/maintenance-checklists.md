# Maintenance Checklists

## Weekly

- [ ] Review Proxmox node health
- [ ] Review thin-pool usage
- [ ] Check failed VM backups
- [ ] Check NAS health
- [ ] Check critical Docker hosts
- [ ] Review Home Assistant alerts
- [ ] Check Plex / media availability
- [ ] Confirm Pi-hole is serving DNS
- [ ] Confirm Traefik certificates are healthy

## Monthly

- [ ] Review disk utilization trends
- [ ] Review Docker disk usage
- [ ] Remove known-safe abandoned containers/images
- [ ] Review NAS volume utilization
- [ ] Verify service-backup snapshots
- [ ] Test one restore
- [ ] Review firmware / package updates
- [ ] Review UniFi disconnected infrastructure clients
- [ ] Review public service exposure
- [ ] Review Authelia-protected services
- [ ] Check fstrim.timer on thin-provisioned VMs

## Quarterly

- [ ] Test Proxmox VM restore
- [ ] Test Home Assistant restore
- [ ] Test Vaultwarden restore
- [ ] Review backup retention
- [ ] Review firewall rules
- [ ] Review VLAN design
- [ ] Audit `.env` / secret storage
- [ ] Export critical WikiMD runbooks
- [ ] Review unused DNS records
- [ ] Review unused Traefik routers
- [ ] Review unused Docker volumes
- [ ] Verify NAS SMART / drive-health reporting

## Before Major Changes

- [ ] Take backup
- [ ] Record current config
- [ ] Record package / image versions
- [ ] Confirm rollback procedure
- [ ] Verify console access if SSH may be interrupted
- [ ] Confirm maintenance affects no critical automations

## After Major Changes

- [ ] Check service health
- [ ] Check logs
- [ ] Test DNS
- [ ] Test HTTPS
- [ ] Test authentication
- [ ] Verify mounts
- [ ] Verify backups
- [ ] Update WikiMD
