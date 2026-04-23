# Third-Party Licenses

This firmware binary includes components from the following open-source projects. The binary is distributed with their respective copyright notices and in compliance with their licenses. Full license texts are linked below.

| Component | License | Project |
|-----------|---------|---------|
| ESP32 Arduino Core | LGPL 2.1 | https://github.com/espressif/arduino-esp32 |
| LovyanGFX | FreeBSD (BSD 2-clause) | https://github.com/lovyan03/LovyanGFX |
| PNGdec | Apache 2.0 | https://github.com/bitbank2/PNGdec |
| ArduinoJson | MIT | https://github.com/bblanchon/ArduinoJson |
| ESPAsyncWebServer | LGPL 3.0 | https://github.com/ESP32Async/ESPAsyncWebServer |
| AsyncTCP | LGPL 3.0 | https://github.com/ESP32Async/AsyncTCP |
| DNSServer (Arduino ESP32 core) | LGPL 2.1 | https://github.com/espressif/arduino-esp32 |
| ESP_DoubleResetDetector | MIT | https://github.com/khoih-prog/ESP_DoubleResetDetector |

## Notices

- All components listed above are used **as-is**, linked into the firmware binary.
- The original copyright notices and license terms for each component are preserved and can be retrieved from the project URLs above.
- LGPL components (ESP32 Arduino Core, ESPAsyncWebServer, AsyncTCP) are distributed dynamically linked into the firmware. A copy of the corresponding LGPL library source (unmodified) is available from the upstream repositories linked above.

## Firmware itself

The firmware corresponding to these binaries is provided in the public source repository:

- https://github.com/jerrylion77/SanapTouch-and-Wayne-Box-Dryer

Project lineage attribution:

- Original Snapmaker U1 mirror foundation by https://github.com/suchmememanyskill/ESP32-Snapmaker-U1-display-mirror

License for firmware source is documented in the source repository (`LICENSE`).

---

If any upstream license terms change or you notice a misattribution, please open an issue.
