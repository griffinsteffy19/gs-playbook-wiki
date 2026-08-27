# Networking & DNS

## Core Network

Primary router / firewall:

```text
Ubiquiti UDM Pro
```

Responsibilities include:

- routing
- firewall policy
- VLANs
- switch/client visibility
- internet edge

## DNS

Primary internal DNS platform:

```text
Pi-hole
```

Pi-hole is used for local name resolution and wildcard DNS.

Known operational note:

When testing local DNS from Home Assistant or another client, query Pi-hole directly
rather than assuming the client's default resolver is using the expected server.

Example:

```bash
nslookup host.example <pihole-ip>
```

## Local Wildcard DNS

Local wildcard records have been implemented using dnsmasq-style configuration.

After changing local dnsmasq entries, Pi-hole FTL must be reloaded / restarted
for the new records to take effect.

## Reverse Proxy Flow

Typical application path:

```text
client
  ↓
DNS
  ↓
UDM Pro / LAN
  ↓
Traefik
  ↓
application
```

WAN HTTPS traffic is forwarded to Traefik for publicly exposed services.

Existing Traefik configuration details belong on the existing:

[Traefik](traefik)

page.

## Certificates

Traefik uses Cloudflare DNS challenge for certificate issuance.

Wildcard certificate patterns have been used for internal/service subdomains.

Certificate state is stored in:

```text
acme.json
```

Treat this as sensitive infrastructure state.

## VLAN / Isolation Notes

Known network segmentation includes a separate TV / isolated network.
Home Assistant requires selective cross-VLAN access for integrations such
as the Samsung Frame TV — see
[UniFi Firewall and VLANs](unifi-firewall-and-vlans) for the documented
rule and rule-design pattern.

## VPN Egress

The media stack uses Gluetun with Private Internet Access.

Typical Docker relationship:

```yaml
network_mode: service:gluetun
```

Services previously using the VPN path include:

- qBittorrent
- Sonarr
- Radarr
- Prowlarr

See [Media Networking and VPN](../services/media-networking-and-vpn).

## Useful Diagnostics

```bash
ip addr
ip route
ss -tulpn
nslookup <name> <dns-server>
dig <name>
curl -vk https://<service>
```

## To Add

- [ ] VLAN inventory
- [ ] subnet inventory
- [ ] DHCP ranges
- [ ] Pi-hole IPs
- [ ] internal domain / wildcard inventory
- [ ] firewall rule matrix
- [ ] WAN-exposed service inventory
