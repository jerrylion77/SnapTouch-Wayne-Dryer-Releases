# SnapTouch & Wayne Box Dryer - User Manual

This manual explains installation, configuration, and troubleshooting for the SnapTouch and Wayne Box Dryer firmware release binaries.

## What is this?

SnapTouch firmware turns an ESP32-3248S035C (3.5" CYD) into:

- A Snapmaker U1 screen mirror with touch forwarding.
- An optional Home Assistant dryer monitor UI with temperature, humidity, and dryer ON/OFF status.

It is distributed as pre-built `.bin` files in this releases repository.

## Use cases

- Side display for Snapmaker U1 controls.
- Dryer dashboard view from Home Assistant entities.
- Combined mirror + dryer interface with quick mode toggling on one device.
- Dryer-only mode where printer mirror is disabled.

## Hardware requirements

- ESP32-3248S035C (Cheap Yellow Display, 3.5" panel, ST7796 + GT911 touch): [Amazon link](https://amzn.to/3OEduE4)
- USB-C cable (data-capable).
- Computer for flashing (Windows/macOS/Linux).
- 2.4GHz WiFi network reachable by the device.
- Snapmaker U1 with remote screen endpoint available.
- [Paxx12 Extended Firmware](https://github.com/paxx12/SnapmakerU1-Extended-Firmware) on your U1, with remote screen enabled.

## Hardware requirements for Home Assistant dryer integration

For robust dryer state and telemetry, recommended hardware in HA:

- Temperature/humidity sensor (for `temp` and `hum` values).
- Smart switch/plug with power monitoring (for dryer ON/OFF inference).
- Required helper (`input_boolean`) representing dryer running state when using the example template/automation flow below.

This lets you derive stable dryer status from measured power and expose a clean merged sensor for the CYD.

## Features and controls

### Core features

- Snapmaker screen mirroring over WiFi.
- Touch forwarding to printer.
- Dedicated dryer UI (HA-driven).
- Captive portal configuration.
- Sleep/wake behavior to preserve panel life.

### Touch controls

- Top-right hold 10s: reboot.
- Top-center hold 5s: toggle Printer <-> Dryer UI (when both modes are enabled).
- First touch after sleep wakes screen only.

### LED status meanings

- **Blue (solid):** connecting to WiFi.
- **Green (solid):** setup complete / ready state.
- **Amber/Yellow (solid):** captive portal mode (`SETUP_U1_MIRROR` AP).
- **Red ramp/pulse:** reboot long-press progress indicator (top-right hold).
- **Tiny blue pulse/ramp:** mode-toggle long-press progress indicator (top-center hold).

## Flashing instructions

### Option A: Windows GUI (Espressif Flash Download Tool)

1. Open Flash Download Tool.
2. Select `ESP32` and `Develop` mode.
3. Load merged file `wayne-dryer-box-vX.Y.Z.bin` at offset `0x0`.
4. Set flash options:
   - Flash size: 4MB
   - SPI speed: 40MHz
   - SPI mode: DIO
5. Start flashing.
6. If connection fails, hold BOOT while connection starts.

### Option B: CLI

```bash
pip install esptool
esptool.py --chip esp32 --port /dev/ttyUSB0 --baud 460800 write_flash 0x0 wayne-dryer-box-vX.Y.Z.bin
```

On macOS use a port like `/dev/cu.usbserial-*`.

## First-time setup

1. After successful flash, LED turns yellow/orange and AP `SETUP_U1_MIRROR` appears.
2. Connect from phone/laptop.
3. Captive portal should open automatically.
4. Fill required fields and HA fields (if using dryer integration).
5. Save and allow reboot.
6. Device joins WiFi and starts normal operation.

## Captive portal fields (definition and purpose)

The exact UI labels may vary slightly by release, but this is the intent of each field:

- **WiFi SSID**: target network name.
- **WiFi password**: target network password.
- **Printer IP**: IPv4 address of Snapmaker U1 endpoint.
- **Enable printer mirror**: enable/disable printer polling and touch forwarding.
- **Display brightness**: active backlight level.
- **Display rotation**: screen orientation.
- **Enable HA dryer mode**: enables HA polling and dryer UI logic.
- **HA host/IP**: Home Assistant host (IPv4) reachable on local network.
- **HA sensor entity ID**: sensor entity to poll via HA state API.
- **HA token**: long-lived access token used as Bearer auth header.

### Example portal configuration

- WiFi SSID: `MyHomeWiFi`
- WiFi password: `********`
- Printer IP: `192.168.1.50`
- Enable printer mirror: `ON`
- Brightness: `200`
- Rotation: `1`
- Enable HA dryer mode: `ON`
- HA host/IP: `192.168.1.20`
- HA sensor entity ID: `sensor.dryer_u1one_data`
- HA token: `eyJ...` (long-lived token from your HA profile)

## Forcing captive portal mode

If you need to reconfigure:

- Press reset twice within about 10 seconds.
- Wait for yellow/orange LED and AP `SETUP_U1_MIRROR`.
- Reconnect and update settings.

## Configuring Home Assistant

This firmware expects an HA state entity with attributes:

- `temp`
- `hum`
- `dryer_status` (values usually `ON` or `OFF`)

It reads:

`GET http://<HA_IP>:8123/api/states/<entity_id>`

with:

`Authorization: Bearer <token>`

### 1) Create a long-lived token

In Home Assistant:

1. Open your user profile.
2. Scroll to Long-Lived Access Tokens.
3. Create a token and copy it.
4. Paste token into the captive portal HA token field.

Keep this token private.

### 2) Add template sensor in `configuration.yaml`

Example:

```yaml
template:
  - sensor:
      - name: "Dryer U1one Data"
        unique_id: dryer_u1one_hmi_test
        state: "OK"
        attributes:
          temp: "{{ states('sensor.aqara_temperature_office_temperature') | float(0) | round(1) }}"
          hum: "{{ states('sensor.aqara_temperature_office_humidity') | float(0) | round(1) }}"
          dryer_status: "{{ 'ON' if is_state('input_boolean.statusu1one', 'on') else 'OFF' }}"
```

After editing HA config, validate and restart/reload template entities.

This template is exactly what the CYD expects to receive in HA state attributes:

- `temp` (numeric)
- `hum` (numeric)
- `dryer_status` (`ON`/`OFF`)

### 3) Create helper for dryer running state (required)

Create `input_boolean.statusu1one` (or your preferred name).  
This helper is required for the example template + automation because `dryer_status` is derived from this boolean and then passed to the CYD.

### 4) Add automation for smart-plug power logic

Example:

```yaml
alias: "Dryer: Sync U1 State from Smart Plug"
description: >-
  Toggles the HMI helper based on power draw (30s buffer) or immediate plug
  state.
triggers:
  - trigger: numeric_state
    entity_id:
      - sensor.smart_plug_2_power
    above: 20
    for:
      hours: 0
      minutes: 0
      seconds: 1
    id: should_be_on
  - trigger: numeric_state
    entity_id:
      - sensor.smart_plug_2_power
    below: 20
    for:
      hours: 0
      minutes: 1
      seconds: 0
    id: should_be_off
  - trigger: state
    entity_id: switch.smart_plug_2_socket_1
    to: "off"
    id: immediate_off
conditions: []
actions:
  - choose:
      - conditions:
          - condition: trigger
            id: should_be_on
          - condition: state
            entity_id: input_boolean.statusu1one
            state: "off"
        sequence:
          - action: input_boolean.turn_on
            target:
              entity_id: input_boolean.statusu1one
      - conditions:
          - condition: or
            conditions:
              - condition: trigger
                id: should_be_off
              - condition: trigger
                id: immediate_off
          - condition: state
            entity_id: input_boolean.statusu1one
            state: "on"
        sequence:
          - action: input_boolean.turn_off
            target:
              entity_id: input_boolean.statusu1one
mode: restart
```

### 5) Entity naming and portal values

- Use the template sensor entity ID in portal HA sensor field.
- Example entity ID: `sensor.dryer_u1one_data`.
- Use HA IP (not URL path) in HA host/IP portal field.
- Confirm HA and CYD are on same LAN and reachable.

### HA behavior notes

- Dryer UI auto-enters on OFF -> ON transition (based on `dryer_status`).
- Manual exit from dryer UI stays sticky until HA reports OFF then ON again.
- Bottom-right status shows `HA: OK` or error.

## Troubleshooting

- **Cannot flash/connect**
  - Use known-good data cable.
  - Hold BOOT during connect.
  - Try lower baud rate (115200).

- **No display after flash**
  - Confirm merged bin at offset `0x0`.
  - Confirm flash settings: 4MB, 40MHz, DIO.

- **Captive portal does not appear**
  - Connect manually to `SETUP_U1_MIRROR`.
  - Force portal by double-reset within 10 seconds.

- **Mirror not updating**
  - Verify printer IP.
  - Verify U1 remote screen feature availability.
  - Check LAN reachability.

- **Touch not working on printer**
  - Confirm mirror mode enabled.
  - Check if first touch after sleep only woke the screen.

- **Dryer fields show `--`**
  - Verify HA host/IP, token, and sensor entity ID.
  - Check template sensor attributes (`temp`, `hum`, `dryer_status`) exist.
  - Check HA logs and device serial logs.

- **Dryer status toggles too fast**
  - Increase automation debounce windows.
  - Tune smart plug power thresholds for your dryer profile.

---

For release binaries and notes, return to repository root `README.md`.
