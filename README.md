[English](README.md) | [Italiano](docs/i18n/README.it.md) | [Español](docs/i18n/README.es.md) | [日本語](docs/i18n/README.ja.md)

# Smart Perimeter

A Home Assistant alarm/perimeter system built from blueprints, with two
things most DIY alarm setups don't have:

1. **Multi-person arm/disarm** from any "who authenticated" numeric-ID
   sensor (fingerprint reader, keypad, NFC tag reader...), with a single
   source of truth for armed state — so it doesn't matter which device or
   which person triggered it, the state never gets out of sync.
2. **Pet-aware partial mode.** If you have a pet that stays home alone,
   a normal alarm's motion sensors will false-alarm on it constantly. This
   system uses a camera AI object detector (tested with
   [Frigate](https://frigate.video/)'s "dog" object tracking, but any
   integration that gives you a "pet is in view" binary sensor per camera
   works) to automatically arm in **Full** mode (all sensors) when nobody
   — human or pet — is home, or **Partial** mode (doors/windows only, no
   interior motion) when a pet is home alone.

Everything here is packaged as [Home Assistant
Blueprints](https://www.home-assistant.io/docs/automation/using_blueprints/):
import them, fill in your own entities, done. No YAML surgery required for
a basic setup.

## Why not just use a normal alarm integration?

You can, and for the door/siren basics you probably should reuse whatever
your alarm panel integration already gives you. This project exists for
the specific gap most alarm setups have: **they can't tell your pet apart
from an intruder**, so either you disable interior motion sensors
permanently (weaker security) or you live with false alarms every time
the pet features get triggered.

## Architecture

```
                    ┌─────────────────────┐
  fingerprint/  ───▶│ Person Trigger       │
  keypad/NFC        │ (one per person)     │
                    └──────────┬───────────┘
                               │ calls
                               ▼
                    ┌─────────────────────┐        ┌───────────────────────┐
  presence   ──────▶│   Arm script        │───────▶│ Disarm script         │
  (auto-arm)        │ (decides Full/      │◀───────│                       │
                    │  Partial from pet    │        └───────────────────────┘
                    │  camera sensors)     │
                    └──────────┬───────────┘
                               │ enables
                               ▼
                    ┌─────────────────────┐        ┌───────────────────────┐
  motion/door ─────▶│ Perimeter Trigger    │──────▶ │ Pet False-Alarm Guard │
  sensors           │ (ignores motion in   │        │ (auto-corrects if     │
                    │  Partial mode)       │        │  wrong mode + pet     │
                    └──────────┬───────────┘        │  triggers siren)      │
                               │                     └───────────────────────┘
                               ▼
                    ┌─────────────────────┐
                    │ Rearm (auto-clear)   │
                    └─────────────────────┘
```

## Blueprints in this repo

| Blueprint | Domain | Purpose |
|---|---|---|
| `arm.yaml` | script | Arms the system; auto-picks Full/Partial from pet sensors unless told otherwise |
| `disarm.yaml` | script | Disarms; undoes siren side-effects if it was actually sounding |
| `person_trigger.yaml` | automation | One instance per person; toggles arm/disarm from a numeric ID sensor |
| `perimeter_trigger.yaml` | automation | Sounds the siren on a breach; mode-aware |
| `rearm.yaml` | automation | Clears the siren once sensors are back to resting state |
| `auto_arm_away.yaml` | automation | Arms automatically once presence trackers show everyone away |
| `pet_false_alarm_guard.yaml` | automation | Self-corrects a false alarm caused by the pet, and always notifies you |

See [`docs/hardware.md`](docs/hardware.md) for notes on what hardware
actually matters (cameras/GPU, authentication device, siren, voice
satellites, VPN for reliable away-from-home presence).

## Prerequisites

- A way to authenticate people with a numeric ID per person (a Zigbee/Tuya
  fingerprint reader exposing a `sensor.*` with the matched ID is the
  reference implementation this was built against; a keypad with codes
  mapped to numbers, or an NFC reader, would work the same way).
- A siren `switch.*` entity.
- An `input_boolean` for "armed" state.
- A second boolean-like entity for the "Partial mode" flag. Ideally a
  second `input_boolean` — **but** if your instance doesn't let you create
  new helper entities from automation tooling (this happened to us: the
  Home Assistant REST config API supports creating/editing automations and
  scripts, but not `input_boolean`/`input_select` helpers), you can reuse
  any spare `automation` entity purely for its on/off state instead. It's
  a hack, but it works fine and is invisible to the end user — see
  `blueprints/script/ha-smart-perimeter/arm.yaml` for how the on/off is
  done generically via `homeassistant.turn_on`/`turn_off` so either kind
  of entity works.
- (Optional, for pet-aware mode) A camera AI pipeline that gives you a
  binary sensor per camera for "pet detected". With Frigate, this means
  adding `dog` (or whatever species you have) to `objects.track` in your
  camera config — no custom model/training needed, `dog` is already one
  of the 80 standard COCO classes most Frigate detector models support.
- (Optional, for auto-arm-when-away) Reliable `person.*`/`device_tracker.*`
  presence — see the note in `auto_arm_away.yaml` about VPN/remote-access
  being a prerequisite for this to actually work when you leave your Wi-Fi
  network.

## Installation

1. In Home Assistant: Settings → Automations & Scenes → Blueprints →
   Import Blueprint, and paste the raw GitHub URL of each `.yaml` file in
   this repo (or just copy the files into your `config/blueprints/...`
   folder structure and reload).
2. Create the `input_boolean` for armed state (and, ideally, a second one
   for partial mode) under Settings → Devices & Services → Helpers.
3. Create a script from `arm.yaml` and one from `disarm.yaml`, filling in
   your entities.
4. Create one `person_trigger.yaml` automation **per person** who should
   be able to arm/disarm.
5. Create automations from `perimeter_trigger.yaml`, `rearm.yaml`, and
   `pet_false_alarm_guard.yaml`. **Leave them disabled at first** — the Arm
   script turns them on/off for you; they shouldn't be independently
   enabled outside of an armed session.
6. Optionally, `auto_arm_away.yaml` for hands-free arming when everyone
   leaves.

## Lessons learned (read before you rely on this for real security)

- **Instantaneous presence checks are fragile.** Our first version checked
  "is the pet visible right now" at the exact moment of arming, and it
  regularly got it wrong — the pet would just be out of frame for a
  minute. The `arm.yaml` script instead checks a **time window** (pet seen
  in the last N minutes, default 15), which is far more forgiving. The
  `pet_false_alarm_guard.yaml` automation deliberately does the opposite
  (instantaneous check) because there the question is different: "is the
  pet causing this specific trigger right now".
- **Camera-based "is anyone home" is not the same as phone-based
  presence.** Cameras only cover the rooms you point them at. We initially
  used "no person visible on any camera" as the trigger for auto-arm, and
  it armed the house while a family member was still inside, just in an
  uncovered room — a real, live false alarm. `auto_arm_away.yaml` is built
  around `person`/`device_tracker` presence specifically to avoid this;
  don't swap in camera occupancy sensors there.
- **Two independent arm/disarm input devices need one source of truth.**
  If you have more than one way to arm (e.g. a fingerprint reader *and* a
  key fob remote), route both through the same Arm/Disarm scripts rather
  than writing separate logic per device — otherwise the "armed" helper
  and the real system state can silently drift apart.
- **A repeating background check for "did the pet actually leave" is a
  real tradeoff, not a free upgrade.** We considered adding a periodic
  re-check that would auto-upgrade Partial → Full once the pet hasn't been
  seen in a while, and deliberately did not add it here as default
  behavior: it uses the exact same fragile "is the camera seeing it"
  signal that caused the original bug, just in the other direction. If you
  add this yourself, use a noticeably wider "definitely gone" window (we'd
  suggest 30-40 minutes, not 15) than the arm-time check, since a wrong
  Partial→Full upgrade re-introduces false alarms, while staying in
  Partial a bit too long only costs you reduced motion coverage.

## License

MIT — see `LICENSE`.
