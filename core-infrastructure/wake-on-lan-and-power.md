# Wake-on-LAN and Power Management

## Overview

Wake-on-LAN has been tested on Ubuntu 24.04 systems in the homelab.

One known interface from prior testing:

```text
enp86s0
```

## Verify Driver Support

```bash
sudo ethtool enp86s0 | grep Wake-on
```

Example working configuration:

```text
Supports Wake-on: pumbg
Wake-on: g
```

`g` means magic-packet wake is enabled.

## NetworkManager Configuration

Verify:

```bash
nmcli -f 802-3-ethernet.wake-on-lan connection show "Wired connection 1"
```

Expected:

```text
802-3-ethernet.wake-on-lan: magic
```

Configure:

```bash
nmcli connection modify "Wired connection 1" 802-3-ethernet.wake-on-lan magic
```

## Applying Without Dropping SSH

Avoid manually taking the active connection down while connected only through SSH.

Prefer:

```bash
nmcli device reapply <interface>
```

when supported.

Otherwise, apply during a maintenance window with console access.

## Sending Magic Packet

From macOS:

```bash
wakeonlan <MAC>
```

Typical output:

```text
Sending magic packet to 255.255.255.255:9
```

## Shutdown vs Suspend

Wake-on-LAN behavior depends on motherboard firmware and power state.

A system may support WOL from:

- suspend
- soft power-off (S5)

but not necessarily both.

## Previous Failure Pattern

A tested machine had:

- `Wake-on: g`
- Ethernet port lights after shutdown
- magic packet successfully transmitted
- no remote power-up
- physical power button required

This strongly suggests a firmware / motherboard power-state limitation rather than a Linux
configuration problem.

## BIOS Settings to Check

Look for:

```text
Wake on LAN
Power On By PCI-E
Resume by LAN
Deep Sleep Control
ErP
S5 Wake
```

Disable aggressive deep-sleep / ErP options if they cut auxiliary NIC power.

## Verification Procedure

1. boot normally
2. verify `Wake-on: g`
3. record MAC address
4. shut down
5. confirm NIC link/activity lights remain on
6. send magic packet
7. wait for DHCP / ping / SSH
8. repeat after BIOS changes if needed

## Important Note

If the motherboard removes NIC wake capability in S5, Linux settings alone cannot fix it.
