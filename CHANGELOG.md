# Changelog

## v0.1.0 — initial release

- Arm/disarm scripts as Home Assistant blueprints, single source of truth
  for armed state regardless of which input device triggers them.
- Multi-person support via a "person trigger" blueprint meant to be
  instantiated once per authorized person.
- Pet-aware Full/Partial mode selection based on camera object-detection
  sensors (e.g. Frigate "dog" tracking), using a time-window presence
  check rather than an instantaneous one (see README "Lessons learned").
- Perimeter trigger + rearm automations, mode-aware (motion sensors
  ignored in Partial mode, doors/windows always active).
- Pet false-alarm guard: auto-corrects and switches to Partial mode if the
  siren fires in Full mode while a pet is detected, always notifies.
- Optional auto-arm-when-everyone-leaves, based on presence trackers
  (explicitly not camera occupancy — see README).
