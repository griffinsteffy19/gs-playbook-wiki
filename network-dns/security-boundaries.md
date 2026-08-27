# Security Boundaries

## Philosophy

The homelab uses several layers of security rather than relying on one control.

Primary layers:

1. UDM Pro firewall
2. VLAN segmentation
3. DNS control
4. Traefik routing
5. Authelia where appropriate
6. application authentication
7. service-level secrets
8. host-level permissions

## VLAN Segmentation

Known segmentation includes an isolated TV / IoT-style network.

Cross-VLAN access should be granted only for required flows.

### Samsung Frame TV

See [UniFi Firewall and VLANs](unifi-firewall-and-vlans) for the
documented rule (TV VLAN → Home Assistant).

A good firewall rule should be:

- source-specific
- destination-specific
- port-specific
- justified in the wiki

## Reverse Proxy Authentication

Authelia is used for selected applications.

Do not automatically place every application behind Authelia.

Consider application-native authentication when:

- the app already has strong MFA
- mobile clients need direct API access
- proxy auth breaks integrations
- WebSocket / API behavior becomes fragile

## Home Assistant

Home Assistant should be treated as a high-value system because it can influence physical
devices and has broad visibility into the home network.

Priorities:

- MFA
- strong backups
- limited exposure
- explicit cross-VLAN rules
- careful integration credentials

## Vaultwarden

Vaultwarden is highly sensitive.

Priorities:

- reliable backup
- HTTPS only
- database/config backup
- secure admin token handling
- restore testing

## Secrets

Do not store in WikiMD:

- passwords
- private keys
- API tokens
- Cloudflare credentials
- Authelia secrets
- VPN credentials
- `.env` secret values

Document only where the secret is stored.

## Docker

Avoid unnecessary privileges.

Prefer:

- least privilege
- read-only mounts where possible
- specific bind mounts
- explicit ports
- no Docker socket exposure unless required

## NAS

The Synology holds high-value data and backup copies.

Protect:

- DSM admin accounts
- SMB credentials
- rsync credentials
- backup share permissions
- Docker registry data

## Firewall Documentation Template

| Source | Destination | Port / Protocol | Allow/Deny | Reason |
|---|---|---|---|---|
| | | | | |

## Service Exposure Review

Review periodically:

```text
Which services are reachable from the internet?
Which services need to be?
Which have MFA?
Which rely only on a password?
Which expose administrative interfaces?
```
