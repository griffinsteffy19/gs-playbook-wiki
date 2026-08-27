# Home Assistant: Internet Resilience

## Verizon 5G Gateway Auto-Restart

Primary internet is a Verizon 5G Home Internet gateway that occasionally
drops connection and needs a power cycle. Automated via a smart plug +
ping-based outage detection.

### Hardware requirement

**Must use a locally-controlled smart plug** — not a cloud-dependent one
(stock Kasa/Tuya/SmartLife/most Amazon-brand plugs go through the
vendor's cloud, which is unreachable exactly when the internet is down,
i.e. exactly when the automation needs to fire).

Locally controlled options: Zigbee/Z-Wave plugs (Third Reality, Innr,
Zooz, Sonoff ZBMini), Shelly Plug S (cloud disabled), ESPHome/Tasmota,
or Matter over Thread/Wi-Fi.

Don't power Home Assistant or UniFi gear from the same plug being cut,
and if the plug is Wi-Fi-based, its AP must not be downstream of
whatever's being power-cycled.

### Outage detection

Two **Ping** integration entities (raw IPs, so DNS failure can't skew
detection):
- `8.8.8.8` (Google) → `binary_sensor.ping_google_dns`
- `1.1.1.1` (Cloudflare) → `binary_sensor.ping_cloudflare_dns`

Automation requires **both** to be down before acting, to avoid reacting
to a single endpoint's hiccup.

### Retry/backoff logic

- A `counter.gateway_reboots` helper caps retries at 3 attempts, to
  avoid an infinite reboot loop during a genuine Verizon-side outage
  (which no amount of power-cycling fixes).
- Phases tracked via `input_text.gateway_reboot_phase`: Idle → power
  cycle → Booting → Backoff (15 min cooldown) → Recovered / Gave up.
- On recovery, sends a push notification with total outage duration and
  reboot count via a `notify.all_phones` group.
- A separate "reset" automation clears the reboot counter once the
  connection has been stable for 1 hour.

### Reference implementation

Core entities: `switch.verizon_gateway_plug`, `timer.gateway_phase`,
`input_datetime.gateway_outage_started`, `script.power_cycle_gateway`.
Full automation YAML exists from prior setup — reconstruct from chat
history if the live config needs to be recovered.

## To Do

- [ ] Confirm this automation is still active/tuned as documented
- [ ] Add UniFi WAN-status as a third detection signal if a UDM-class
      gateway is in use
