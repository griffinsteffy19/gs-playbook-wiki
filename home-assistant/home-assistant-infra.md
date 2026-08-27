# Home Assistant Infrastructure

## Platform

Home Assistant OS runs as a VM on Proxmox.

It is a central aggregation point for both home automation and homelab monitoring.

## Known Infrastructure Integrations

Previously configured or discussed integrations include:

- Proxmox VE
- Plex
- Tautulli
- UniFi Network
- Synology DSM
- Ambient Weather

## DNS

Home Assistant has its own internal resolver behavior.

When troubleshooting local names, verify resolution against Pi-hole explicitly rather
than assuming Home Assistant's default DNS path matches another LAN client.

## Cross-VLAN Access

Some integrations require carefully scoped access across network boundaries.

Example: Samsung Frame TV → Home Assistant. Documented rule lives in
[UniFi Firewall and VLANs](../network-dns/unifi-firewall-and-vlans).

Document any cross-VLAN rules with:

- source network
- destination host
- protocol / port
- reason

## Infrastructure Dashboard Direction

The preferred monitoring direction has been a high-level Home Assistant page for:

- Proxmox
- Plex / Tautulli
- UDM Pro / UniFi
- Synology
- Linux hosts / Glances

The goal is a concise operational overview rather than a large service-by-service
wall of cards.

## Climate / Runtime Tracking

Home Assistant is also being used to track HVAC behavior.

Work has included:

- Ecobee thermostat
- cooling runtime
- heating runtime
- daily / weekly / monthly / yearly utility meters
- dynamic home temperature helpers

See [HVAC Runtime Tracking](hvac-runtime-tracking) and
[Home Assistant Dynamic Temperature](home-assistant-dynamic-temperature).

## Internet Resilience

See [Home Assistant Internet Resilience](home-assistant-internet-resilience)
for the Verizon gateway auto-restart automation.

## To Add

- [ ] VM ID
- [ ] Proxmox node
- [ ] backup schedule
- [ ] internal / external URL
- [ ] critical integrations
- [ ] cross-VLAN firewall rules
- [ ] restore procedure
