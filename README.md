# ATEM tally light for M5StickC / ESP32

> **This is a fork** of [AronHetLam/ATEM_tally_light_with_ESP8266](https://github.com/AronHetLam/ATEM_tally_light_with_ESP8266) by Aron N. Het Lam, focused on **ESP32** and the **M5StickC** family (M5StickC / Plus / Plus2). All credit for the original tally light, the tally server and the ATEM integration goes to the original project.

Wireless tally light and 'On Air' sign for Blackmagic ATEM switchers. It connects to the switcher over WiFi and shows the tally state on the M5StickC display, an RGB LED or a Neopixel LED strip.

The tally lights can retransmit tally data to each other, so the switcher only needs one connection no matter how many tally lights you use (ATEM switchers allow only 5–8 clients).

# Supported boards
| PlatformIO environment | Board |
|------------------------|-------|
| `M5StickC` | M5StickC, M5StickC Plus, M5StickC Plus2 (model detected at runtime) |
| `ESP32` | Generic ESP32 dev board |
| `ESP32_C3`, `ESP32_S2`, `ESP32_S3` | ESP32-C3 / S2 / S3 dev boards |
| `ATOM_Matrix` | M5Stack ATOM Matrix |

Each environment also has a `_test_server` variant that simulates a switcher (see [Test server](#test-server)).

# Building and uploading
Install [PlatformIO](https://platformio.org/) (for example the *PlatformIO IDE* extension for VS Code), open this folder, select the environment and upload, or from the command line:

```
pio run -e M5StickC -t upload
```

# Setup
On first start (or when the saved WiFi network can't be found after power on) the tally light starts an access point named **"Tally Light setup"**. Connect to it and open [192.168.4.1](http://192.168.4.1) to set:

- tally light name and number (camera input)
- tally mode for LED 1 and LED 2
- LED and Neopixel brightness, number of Neopixels and status LED position
- WiFi network and password, optional static IP
- ATEM switcher IP — or the IP of another tally light to receive tally data from it

After it has connected to WiFi, the same page is available on the tally light's own IP address. If WiFi is lost later, the tally light keeps reconnecting without starting the access point; power cycle it to get back to setup.

# M5StickC
### Display
- The whole screen shows the tally color (follows the *LED 1* mode) with the tally number in large digits. In *On Air* mode it shows "ON AIR".
- While connecting, the screen shows the state as text on the status colors below.
- Top line: tally name, WiFi signal (4 bars + dBm) and battery level (`+` while charging).
- The *Led brightness* setting controls the display backlight.

### Built-in LED and Grove port
- The built-in red LED follows the *LED 2* mode and is lit only for program.
- A Neopixel strip can be connected to the Grove port (G32).

### Buttons
| Button | Action |
|--------|--------|
| M5 (front), short press | Tally number +1 |
| Side button, short press | Tally number −1 |
| M5 (front), hold | Info screen for 5 s: IP, WiFi, ATEM status, battery voltage and current, USB input |

The tally number changes immediately and is saved 3 seconds after the last press. It wraps around the number of sources of the connected switcher.

### Connection warnings
- **Blinking frame** around the screen: no data from the switcher for 2.5 s. The tally color shown may be outdated (the connection is considered lost after 5 s, then the screen turns pink).
- **Blinking signal indicator**: WiFi signal below −80 dBm. About −65 dBm or better is recommended.

### Battery
- Battery level uses a Li-ion discharge curve instead of a linear 3.3–4.15 V scale, with smoothing and a voltage calibration offset (`M5_BATTERY_OFFSET_MV`, calibrated against a multimeter on one unit — adjust for yours).
- The AXP192 charger (M5StickC / Plus) is set to 4.2 V and 450 mA (`M5_CHARGE_VOLTAGE_MV`, `M5_CHARGE_CURRENT_MA`), meant for a large replacement cell (tested with an iPhone 13 mini battery, 2406 mAh). The default 100 mA is too low for such a cell to finish charging before the AXP192 charge timer stops it. **With the stock small cell, lower `M5_CHARGE_CURRENT_MA` to 100.**

# Other ESP32 boards
Default pins (can be changed with `build_flags` in `platformio.ini`, see the `ATOM_Matrix` environment for an example):

| Function | ESP32 pin |
|----------|-----------|
| LED 1 red / green / blue | 32 / 33 / 25 |
| LED 2 red / green / blue | 26 / 27 / 14 |
| Neopixel data | 12 (4 on ESP32-C3) |

# Status colors
Color | Description
------|--------
BLUE | Connecting to WiFi
WHITE | Unable to connect to WiFi after power on — serves the "Tally Light setup" access point while still trying to connect
PINK | Connecting to ATEM switcher (connected to WiFi)
RED | Tally program
GREEN | Tally preview
OFF | Tally neither in program nor preview
ORANGE | Connected and running (only shown by the status LED on a Neopixel strip)

# Modes of operation
Mode | Description
-----|------------
Normal | As described in the table above
Preview stay on | Green whenever not in program
Program only | Off whenever not in program
On Air | Red while the switcher is streaming, off otherwise

On Air mode only works with ATEM models that support streaming, and needs a direct connection to the switcher (not to another tally light). It still retransmits tally data to other tally lights.

# Connecting many tally lights
Every tally light also acts as a tally server for up to 5 other tally lights. Point a tally light's "ATEM switcher IP" to another tally light instead of the switcher, and it receives tally data from that unit. This way the switcher only needs one connection.

# Test server
The `_test_server` environments build a switcher simulator: tally lights connect to it like to an ATEM switcher. Press Enter in the serial monitor (115200 baud) to cycle all tallies through off → program → preview → both.

# Credits and license
- Original project: [ATEM_tally_light_with_ESP8266](https://github.com/AronHetLam/ATEM_tally_light_with_ESP8266) by [Aron N. Het Lam](https://aronhetlam.github.io/)
- ATEM libraries for Arduino by [SKAARHOJ](https://www.skaarhoj.com/): [SKAARHOJ-Open-Engineering](https://github.com/kasperskaarhoj/SKAARHOJ-Open-Engineering)
- M5StickC support uses [M5Unified](https://github.com/m5stack/M5Unified)

Licensed under the GNU General Public License v3.0, see [LICENSE](LICENSE).
