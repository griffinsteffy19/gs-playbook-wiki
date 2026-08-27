# UniFi Firewall and VLANs

## Overview

The UDM Pro is the primary firewall and router.

A core design goal is segmentation: devices should only reach other networks when required.

## Known Segmentation

At minimum, the homelab includes:

- trusted/default network
- isolated TV / IoT-style network

Home Assistant needs selected cross-network access for some integrations.

## Device Internet Blocking

UniFi can be used to block internet access for individual devices while preserving local LAN
access.

This is useful for:

- IoT devices
- TVs
- appliances
- devices that only need Home Assistant/local control

## Cross-VLAN Rule Pattern

Prefer rules like:

```text
Source: specific VLAN/device
Destination: specific host/service
Port: specific port(s)
Action: allow
```

rather than broad:

```text
allow VLAN A -> VLAN B
```

## Example Use Case

Samsung Frame TV:

```text
TV VLAN
   ->
Home Assistant
```

Only the traffic required by the integration should be allowed.

## Documentation Table

| Rule | Source | Destination | Ports | Action | Reason |
|---|---|---|---|---|---|
| HA-TV | TV VLAN | Home Assistant | fill in | Allow | Samsung integration |
| Internet Block | Device | WAN | Any | Deny | Local-only device |

## Troubleshooting

When a cross-VLAN integration fails:

1. confirm source IP
2. confirm destination IP
3. confirm route exists
4. inspect firewall rule order
5. inspect return traffic rules
6. confirm application port
7. test from same VLAN
8. test from source VLAN

## UniFi Event History

UniFi event timestamps have been useful when correlating host outages.

For infrastructure nodes, record:

- switch
- switch port
- MAC address
- disconnect time
- reconnect time

This can help separate:

- host power loss
- NIC failure
- switch-port failure
- upstream network failure
