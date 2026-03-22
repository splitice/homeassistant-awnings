## Home Assistant Awnings Automations

This repository contains Home Assistant YAML automations for awning control:

- `automationawning_controller_supervisor.yml` (entity `automation.awning_controller_supervisor`): main 1-minute supervisor loop
- `automation.awning_ensure_supervisor_is_running.yml`: watchdog that restarts the supervisor if it stops
- `automation.awning_bedroom_backlight_schedule.yml`: bedroom awning backlight schedule

## Behavior

The supervisor automation:

- Discovers awnings by labels (`awning`, `awning_hot_close`, `awning_bedroom`, direction labels)
- Calculates sun exposure from `sun.sun` azimuth/elevation plus:
  - `input_number.awning_min_sun_elevation`
  - `input_number.awning_exposure_half_band`
- Closes exposed hot-day awnings when `binary_sensor.awning_hot_day` is `on`
- Opens bedroom awnings to position `11` during each awning's `bedroom_wakeup_HHMM` label window (first 20 minutes after configured wake time)
- Closes bedroom awnings in afternoon/evening only (from 12:00 onward) when not exposed or sun elevation is below minimum
- Opens non-bedroom awnings in morning conditions
- Detects manual overrides and temporarily suppresses automatic control for those awnings

## Persisted state map topic

The supervisor persists its internal `state_map` as retained JSON in MQTT topic:

- `ha/state/awning_controller_supervisor`

This retained topic is used to restore state across automation restarts/reloads.

Requirements:

1. Home Assistant MQTT integration configured and connected to your broker.
2. Broker allows publish/subscribe for topic `ha/state/awning_controller_supervisor`.

If no retained payload exists yet, the automation starts with an empty state map and publishes retained state once it changes.
