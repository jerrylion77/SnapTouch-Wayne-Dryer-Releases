# SnapTouch & Wayne Box Dryer — Firmware Releases

Pre-built firmware for the **Snapmaker U1 touch mirror + Wayne-Box filament-dryer monitor** running on an ESP32-3248S035C (3.5" CYD). Flash the binary from the Releases page — no build toolchain needed.

## Upstream credit and source

This firmware line started from [suchmememanyskill/ESP32-Snapmaker-U1-display-mirror](https://github.com/suchmememanyskill/ESP32-Snapmaker-U1-display-mirror). It has since been substantially rewritten and extended, but credit remains to the original author for creating the initial Snapmaker U1 mirroring project.

## What this firmware does

- Mirrors the Snapmaker U1 printer screen over WiFi with touch support (tap the mirror, the printer responds).
- Optionally polls a Home Assistant sensor for filament-dryer data and shows a dedicated Dryer UI.
- Onboard RGB LED reflects dryer state at a glance (green = dryer ON).
- Discrete touch gestures: 10-second top-right hold = reboot; 5-second top-center hold = toggle between Printer and Dryer screens.
- Captive portal on first boot for WiFi, printer IP, Home Assistant host / sensor / token.

## Hardware required

- ESP32-3248S035C (Cheap Yellow Display, 3.5" model with ST7796 + GT911 touch).
- USB-C cable.
- Host PC (Windows, macOS, or Linux) for initial flashing.

## Flashing — easy path (GUI, Windows)

1. Download [Espressif Flash Download Tool](https://www.espressif.com/en/support/download/other-tools).
2. Choose chip: **ESP32**, work mode: **Develop**.
3. Load `wayne-dryer-box-vX.Y.Z.bin` at offset `0x0`.
4. Set SPI: Flash size 4MB, SPI speed 40MHz, SPI mode DIO.
5. Pick the correct COM port, click **Start**.
6. While flashing starts, hold the **BOOT** button on the CYD if it fails to connect.

## Flashing — CLI path (any OS)

```bash
pip install esptool
esptool.py --chip esp32 --port /dev/ttyUSB0 --baud 460800 write_flash 0x0 wayne-dryer-box-vX.Y.Z.bin
```
(replace `/dev/ttyUSB0` with your serial port; on macOS it's often `/dev/cu.usbserial-*`).

## First-time setup

1. After flashing, the LED turns yellow/orange and the CYD creates a WiFi network called **`SETUP_U1_MIRROR`**.
2. Connect from your phone or laptop — a captive portal opens automatically.
3. Fill in:
   - WiFi SSID and password (pick from the scan list or enter manually).
   - Printer IP address.
   - Display brightness and rotation.
   - *(Optional)* **Enable HA dryer mode** + HA IP + Sensor Entity ID + Long-Lived Access Token.
4. Save. The device reboots and connects to your WiFi.
5. The onboard LED goes off when it's ready. If you re-enter the portal later: double-tap the reset button within 10 seconds.

## Controls

| Gesture | Action |
|---------|--------|
| 10-second hold at top-right corner | Restart the device. |
| 5-second hold at top-center (small zone) | Toggle between Printer Mirror and Dryer UI. |
| Any touch after sleep | Wakes screen only (first tap is not forwarded). |
| No activity or no frames for 5 minutes | Screen sleeps to save panel lifetime. |

## Home Assistant integration

If you enable "Dryer mode" in the portal, the firmware polls:

```
GET http://<HA_IP>:8123/api/states/<entity_id>
Authorization: Bearer <token>
```

The response must contain these keys under `attributes` (numeric strings are accepted):

```json
{
  "entity_id": "sensor.your_dryer_sensor",
  "state": "OK",
  "attributes": {
    "temp": 55.5,
    "hum": 12.0,
    "dryer_status": "ON"
  }
}
```

Behavior:
- Auto-switches to Dryer UI on the OFF → ON transition only. Manual exit stays sticky until HA cycles OFF and back to ON.
- Temperature, humidity and status render with equal weight.
- `HA: OK` / `HA: <error>` banner in the bottom-right during Dryer mode.

## Troubleshooting

- **Nothing shows after flash** → verify offset is `0x0` and flash mode DIO/4MB.
- **Stuck on splash** → WiFi credentials wrong. Double-tap reset to re-enter portal.
- **Dryer UI shows `--`** → HA unreachable, token invalid, or sensor name typo. Connect USB and watch `[dryer]` logs at 115200 baud.

## License

These binaries are built from GPL-licensed source code. See:

- [`LICENSE`](./LICENSE) for GPL-3.0 text.
- [`LICENSES.md`](./LICENSES.md) for third-party notices.
