# SnapTouch & Wayne Box Dryer — Firmware Releases

Pre-built firmware for the Snapmaker U1 touch mirror + Wayne-Box filament-dryer monitor running on an ESP32-3248S035C (3.5" CYD). Flash the binary from the Releases page — no build toolchain needed.

## Upstream credit and source

This firmware line started from [suchmememanyskill/ESP32-Snapmaker-U1-display-mirror](https://github.com/suchmememanyskill/ESP32-Snapmaker-U1-display-mirror). It has since been substantially rewritten and extended, but credit remains to the original author for creating the initial Snapmaker U1 mirroring project.

## What is this?

This repository distributes ready-to-flash firmware binaries so you can run:

- Snapmaker U1 screen mirroring with touch forwarding.
- Optional Home Assistant dryer monitoring UI.
- Captive portal setup (WiFi, printer IP, HA settings, display options).

## Use cases

- Mirror your U1 screen on a side-mounted CYD near the printer.
- Run a dryer status dashboard (temperature, humidity, ON/OFF) from Home Assistant.
- Use one device for both mirror and dryer workflows, or run either mode on its own.

## Features and controls

- Printer mirror + touch forwarding over local network.
- Optional dryer mode with dedicated UI and status LED behavior.
- Top-right 10-second hold: reboot device.
- Top-center 5-second hold: toggle Printer <-> Dryer UI (when both modes are enabled).
- First touch after sleep only wakes the screen (not forwarded).

### LED status meanings

- **Blue (solid):** connecting to WiFi.
- **Green (solid):** setup complete / ready state.
- **Amber/Yellow (solid):** captive portal mode (`SETUP_U1_MIRROR` AP).
- **Red ramp/pulse:** reboot long-press progress indicator (top-right hold).
- **Tiny blue pulse/ramp:** mode-toggle long-press progress indicator (top-center hold).

## Flashing instructions

### GUI (Windows, Espressif Flash Download Tool)

1. Download [Espressif Flash Download Tool](https://www.espressif.com/en/support/download/other-tools).
2. Chip: ESP32, WorkMode: Develop.
3. Load `wayne-dryer-box-vX.Y.Z.bin` at offset `0x0`.
4. SPI settings: 4MB flash, 40MHz, DIO.
5. Select COM port and click Start.
6. If connection fails, hold BOOT while flashing begins.

### CLI (macOS / Linux / Windows)

```bash
pip install esptool
esptool.py --chip esp32 --port /dev/ttyUSB0 --baud 460800 write_flash 0x0 wayne-dryer-box-vX.Y.Z.bin
```

Replace `/dev/ttyUSB0` with your serial port (`/dev/cu.usbserial-*` on macOS).

## First-time setup

1. After flash, device starts AP `SETUP_U1_MIRROR` (LED yellow/orange).
2. Connect and open captive portal.
3. Configure WiFi, printer IP, brightness/rotation, and optional HA settings.
4. Save and reboot.
5. To force setup portal later: double-tap reset within 10 seconds.

## Full manual

For detailed setup, Home Assistant integration, captive portal field-by-field guidance, and troubleshooting:

- [`docs/user-manual.md`](./docs/user-manual.md)

## Buy me a coffee

If this project saved you time or helped your setup, support development:

- [Buy me a coffee - Jerryninov](https://buymeacoffee.com/jerryninov)

[![Buy Me A Coffee](https://cdn.buymeacoffee.com/buttons/v2/default-yellow.png)](https://buymeacoffee.com/jerryninov)

Your support helps maintain firmware updates, testing, and documentation.

## License

- [`LICENSE`](./LICENSE) (GPL-3.0)
- [`LICENSES.md`](./LICENSES.md) (third-party attributions)
