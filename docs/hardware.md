[English](hardware.md) | [Italiano](i18n/hardware.it.md) | [Español](i18n/hardware.es.md) | [日本語](i18n/hardware.ja.md)

# Hardware notes

This project is hardware-agnostic — the blueprints only care about entity
types, not brands. This page collects what actually mattered when building
and running the reference setup, so you can shop smart instead of guessing.

## Camera / AI detection

- Any camera Frigate (or an equivalent object detection pipeline) supports
  works. What matters for the pet-aware feature is **hardware video decode
  (NVDEC or equivalent)**, not raw GPU compute — even a low-end, older GPU
  can decode several camera streams in parallel and run the detector fast
  enough.
- If you also transcode a stream for browser/remote playback (e.g. via
  go2rtc, commonly needed for cameras that don't natively speak a
  browser-friendly codec), that's a **separate** cost from detection, and
  on CPU/software encoding it gets expensive fast. A GPU with **hardware
  encode (NVENC or equivalent)** removes that load from the CPU. Cheap GPUs
  often have decode but not encode — check both specs, not just "has a
  GPU", before buying one for this.
- Detecting a pet doesn't need custom training or a custom model: species
  like "dog" and "cat" are already part of the standard 80-class COCO label
  set that most Frigate detector models ship with — just add the species to
  the camera's tracked object list.

## Authentication device (fingerprint / keypad / NFC)

- Anything that ends up as a Home Assistant `sensor.*` exposing "which
  person matched" as a numeric/string ID works as the arm/disarm trigger.
  The reference implementation used a Zigbee/Tuya fingerprint reader.
- If the device has an addressable status LED, wiring it to an HA `light.*`
  entity is a nice touch for install/error feedback, but isn't required by
  any blueprint here.

## Siren

- A plain relay or smart switch driving a standard 12V siren is enough — no
  need for a "smart alarm" branded siren product. The blueprints only ever
  call `switch.turn_on` / `switch.turn_off`.

## Voice announcements

- Any `assist_satellite.*` entity works — ESPHome-based voice satellites,
  Home Assistant's own voice hardware, etc. The blueprints announce to a
  **list** of device IDs, so you can target several satellites at once, and
  it degrades gracefully if one of them happens to be offline.

## Reliable away-from-home presence

- Phone-based presence (`person.*` / `device_tracker.*`) only stays
  accurate once you leave your home Wi-Fi if the phone keeps a live
  connection back to your Home Assistant instance. Without that, presence
  freezes at "last known state" the instant the phone drops off Wi-Fi —
  which silently breaks `auto_arm_away.yaml`.
- An always-on VPN is what fixes this. It doesn't need to be fancy: a
  consumer router with a **built-in WireGuard client+server** (no separate
  add-on box needed) is enough on the network side. On the phone, enable
  the OS-level "always-on VPN" setting rather than relying on an
  automation app to toggle a third-party VPN app on/off — most automation
  apps can only *launch* a VPN app, not reliably start/stop its tunnel.

## Minimum viable hardware

- One GPU with hardware video decode, only if you want the pet-aware
  feature — otherwise not required at all.
- One authentication sensor per entry point (fingerprint / keypad / NFC).
- Whatever door/window/motion sensors you already have — nothing
  alarm-specific required, standard HA `binary_sensor` entities work as-is.
- A relay-driven siren.
- Everything else (voice announcements, auto-arm-when-away) is optional
  and additive.
