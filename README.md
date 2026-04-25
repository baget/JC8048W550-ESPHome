# JC8048W550 / XTouch 5" ESPHome Configs

ESPHome YAML configurations for the **JC8048W550** (also sold as **XTouch 5"**) — an ESP32-S3 development board with an 800×480 RGB parallel touchscreen display.

> **Note:** This project was created with the assistance of AI (Claude by Anthropic).

---

## Hardware

**Board:** [JC8048W550 — ESP32-S3-WROOM N16R8 + 5" 800×480 Capacitive Touch Display](https://he.aliexpress.com/item/1005006715794302.html?spm=a2g0o.order_list.order_list_main.45.7c8f1802WfBWRM&gatewayAdapt=glo2isr)

| Component | Details |
|-----------|---------|
| MCU | ESP32-S3-WROOM-1 N16R8 (16 MB Flash, 8 MB PSRAM) |
| Display | 5" ST7262 800×480 RGB parallel, 16-bit |
| Touch | GT911 capacitive, I²C |
| Backlight | GPIO2, active HIGH |
| PSRAM | Octal mode, 80 MHz |

---

## Configurations

### `jc8048w550-basic.yaml` — Hello World

Minimal configuration to validate the hardware is working:

- Backlight on
- "Hello World!" centered on screen
- Touch interaction (updates label on press/release)
- GT911 touch controller
- Heartbeat log every 5 seconds

Use this first to confirm the display, backlight, and touch are all functional before flashing more complex configs.

### `jc8048w550-bambu.yaml` — Bambu Lab P1S Dashboard

A full printer-monitoring dashboard for the **Bambu Lab P1S Combo**, pulling live telemetry from Home Assistant via the [greghesp/ha-bambulab](https://github.com/greghesp/ha-bambulab) integration.

![Bambu Lab P1S Dashboard](/asserts/jc8048w550-bambu.jpg)

**Dashboard layout (800×480):**

```
┌──────────────────────────────────────────────────────────┐
│  Bambu Lab P1S Combo          ● [STAGE]                  │
├──────────────┬──────────────┬────────────────────────────┤
│  NOZZLE      │  BED         │  PROGRESS                  │
│  temp + set  │  temp + set  │  big % + bar + layer cnt   │
├──────────────┼──────────────┼────────────────────────────┤
│  TIME        │  FANS        │  FILAMENT                  │
│  start/end/  │  part+aux    │  type + AMS slot           │
│  remaining   │  bars        │                            │
└──────────────┴──────────────┴────────────────────────────┘
```

**Displayed data:**
- Nozzle temperature (current + target)
- Bed temperature (current + target)
- Print progress (% + bar + layer counter)
- Start time, estimated end time, remaining time
- Part-cooling and aux/chamber fan speeds
- Active filament type and AMS slot

---

## Prerequisites

- [ESPHome](https://esphome.io/) 2024.x or later
- Home Assistant with [greghesp/ha-bambulab](https://github.com/greghesp/ha-bambulab) (for the Bambu dashboard)
- A `secrets.yaml` file (not tracked) containing:

```yaml
api_key: "your_esphome_api_key"
ota_password: "your_ota_password"
wifi_ssid: "your_wifi_ssid"
wifi_password: "your_wifi_password"
ap_password: "your_fallback_ap_password"
```

---

## Quick Start

```bash
# Validate config (no device needed)
esphome config jc8048w550-basic.yaml

# Compile only
esphome compile jc8048w550-basic.yaml

# Flash via USB (first time)
esphome upload jc8048w550-basic.yaml --device /dev/ttyUSB0

# OTA update (device on network)
esphome upload jc8048w550-basic.yaml

# Stream logs
esphome logs jc8048w550-basic.yaml
```

---

## Bambu Dashboard Setup

1. In Home Assistant, go to **Settings → Devices → ha-bambulab → your printer**
2. Note your printer's entity prefix (e.g. `p1s_01p00a461100093`)
3. In `jc8048w550-bambu.yaml`, replace every occurrence of `p1s_01p00a461100093` with your prefix
4. Set `tz_offset_hours` to your UTC offset (`3` for Israel IDT, `2` for IST)
5. Flash the device

---

## Critical Hardware Notes

### Backlight — GPIO2 active HIGH

GPIO2 **must** be driven HIGH explicitly — it is not pulled up by default. The `on_boot` action calls `output.turn_on: backlight` before showing any LVGL page. Removing this causes a blank screen with no error.

### RGB Data Pin Re-ordering

ESPHome's `rpi_dpi_rgb` with `color_order: BGR` internally assembles the 16-bit bus in a non-obvious order. The board's physical GPIO wiring does **not** match those slots, so the `data_pins` arrays in `red`/`green`/`blue` are intentionally scrambled to map physical GPIOs to the correct bus positions. **Do not "fix" this mapping** — it is correct by design.

### Display Timing

`pclk_inverted: true` (PCLK_ACTIVE_NEG). Timings: h/v pulse = 4, back/front porch = 8, `pclk_frequency: 16MHz`.

### Touch — GT911

I²C on SDA = GPIO19, SCL = GPIO20, address `0x5D`, 400 kHz, 16 ms poll interval.

### PSRAM / Buffer

`buffer_size: 100%` with octal PSRAM at 80 MHz — a full-frame buffer fits in PSRAM. Do not add a display `lambda:` — LVGL drives redraws internally.

---

## Color Palette (Bambu Dashboard)

| Role | RGB565 |
|------|--------|
| Background | `0x18C3` |
| Card | `0x2945` |
| Border | `0x8C71` |
| Accent (Bambu green) | `0x07E0` |
| Nozzle orange | `0xFD20` |
| Bed teal | `0x07FF` |
| Time amber | `0xFFE0` |
| Filament pink | `0xE21F` |
| Fans purple | `0xC81F` |

---

## License

MIT License

Copyright (c) 2026 Oren Weil

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all
copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
SOFTWARE.
