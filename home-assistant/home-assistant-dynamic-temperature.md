# Home Assistant Dynamic Temperature

## Overview

Home Assistant is being used to calculate dynamic thermostat targets based on a home-state
selector.

Primary selector:

```text
input_select.home_state
```

Known states include:

- Home Day
- Home Night
- Away
- Sleep
- Naptime
- Hosting

## Goal

The design separates:

```text
desired temperature calculation
```

from:

```text
automatic thermostat control
```

This allows the logic to be previewed and validated before Home Assistant is allowed to
change the thermostat automatically.

## Helper Pattern

Each state can have one or more helper values.

Examples:

```text
input_number.home_day_temperature
input_number.home_night_temperature
input_number.away_temperature
```

Additional high/low offsets can be used to construct heating/cooling targets.

## Dynamic Sensor

A calculated sensor such as:

```text
sensor.dynamic_home_temperature
```

can expose the currently selected target.

Conceptual logic:

```text
home_state
   ->
lookup corresponding input_number
   ->
apply offsets / season logic
   ->
dynamic target sensor
```

## Preview-First Approach

Recommended rollout:

### Phase 1

Calculate only.

Show:

- selected state
- desired heat target
- desired cool target
- current thermostat setting

### Phase 2

Notify when actual != desired.

### Phase 3

Allow manual "Apply" action.

### Phase 4

Enable full automation.

This makes thermostat automation safer and easier to debug.

## Inputs Worth Considering

Potential inputs:

- home state
- outdoor temperature
- indoor temperature
- time of day
- occupancy
- hosting mode
- naptime
- sleep state

## Avoid Rapid Adjustment

Add hysteresis / debounce so the target does not constantly change due to noisy state changes.

## Related Pages

- [HVAC Runtime Tracking](hvac-runtime-tracking)
