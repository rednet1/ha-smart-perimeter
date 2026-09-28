# Setup worksheet

Fill this in with your own entities before importing the blueprints — it's
easier to have them all in one place than to look them up one at a time
while clicking through each blueprint's input form.

| Role | Example domain | Your entity |
|---|---|---|
| Armed state helper | `input_boolean.*` | |
| Partial mode flag | `input_boolean.*` or spare `automation.*` | |
| Siren | `switch.*` | |
| LED / visual indicator (optional) | `light.*` | |
| Authentication ID sensor (fingerprint/keypad/NFC) | `sensor.*` | |
| Motion sensors | `binary_sensor.*` (device_class: motion) | |
| Door/window sensors | `binary_sensor.*` (device_class: door/window/opening) | |
| Pet-occupancy sensors (per camera) | `binary_sensor.*` | |
| Presence trackers (for auto-arm) | `person.*` / `device_tracker.*` | |
| Voice announcement targets (optional) | `assist_satellite.*` | |

Per-person table (one row → one `person_trigger.yaml` automation instance):

| Person | ID on sensor | Notes |
|---|---|---|
| | | |
| | | |
