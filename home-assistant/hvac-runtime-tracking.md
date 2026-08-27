# HVAC Runtime Tracking

## Overview

Home Assistant has been configured / designed to track heating and cooling runtime from the
Ecobee thermostat.

Primary climate entity:

```text
climate.thermostat_2
```

## Goal

Track HVAC runtime across:

- today
- week
- month
- year

for both:

- cooling
- heating

## Conceptual Entity Chain

The cleanest model is:

```text
thermostat action
    ->
binary runtime sensor
    ->
history/statistics source
    ->
utility meter
```

separately for heating and cooling.

## Cooling

Conceptual chain:

```text
climate.thermostat_2
    ->
cooling-active sensor
    ->
runtime accumulation
    ->
daily cooling
    ->
weekly cooling
    ->
monthly cooling
    ->
yearly cooling
```

## Heating

Conceptual chain:

```text
climate.thermostat_2
    ->
heating-active sensor
    ->
runtime accumulation
    ->
daily heating
    ->
weekly heating
    ->
monthly heating
    ->
yearly heating
```

## Utility Meter Example

A utility meter should use the cumulative runtime sensor as its source.

Example concept:

```yaml
utility_meter:
  cooling_runtime_daily:
    source: sensor.cooling_runtime_total
    cycle: daily
```

Equivalent meters can be created for:

- weekly
- monthly
- yearly

and duplicated for heating.

## Dashboard Presentation

For mobile, display runtime as:

```text
Cooling Today
2h 18m

Heating Today
0h 42m
```

rather than raw decimal hours.

## Dynamic Temperature Work

Home Assistant also contains work around:

```text
input_select.home_state
```

with states such as:

- Home Day
- Home Night
- Away
- Sleep
- Naptime
- Hosting

and helper-driven target temperatures.

The goal has been to preview / calculate desired targets before enabling automatic control.

## Documentation Recommendation

Keep climate-control automation documentation separate from runtime tracking.

Suggested future pages:

```text
hvac-runtime-tracking
hvac-dynamic-temperature
```
