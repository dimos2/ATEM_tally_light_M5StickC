# ATEM tally light for ESP8266 / ESP32 / M5StickC

> **This is a fork** of [AronHetLam/ATEM_tally_light_with_ESP8266](https://github.com/AronHetLam/ATEM_tally_light_with_ESP8266) by Aron N. Het Lam. All credit for the original tally light, the tally server and the ATEM integration goes to the original project. This fork adds M5StickC support, a button for setting the tally number and battery monitoring for ESP8266, and a few fixes — see [Changes in this fork](#changes-in-this-fork).

Wireless tally light and 'On Air' sign for Blackmagic ATEM switchers. It connects to the switcher over WiFi and shows the tally state on an RGB LED, a Neopixel LED strip or the M5StickC display.

# How it works
1. On power on the tally light connects to the saved WiFi network (**blue**). If the network can't be found, it also starts a setup access point (**white**).
2. It connects to the ATEM switcher (**pink**) and receives the tally state of all inputs.
3. It shows the state of its own input (the tally number): **red** for program, **green** for preview, off otherwise. Other [modes](#modes-of-operation) are available.
4. Each tally light also acts as a tally server: other tally lights can connect to it instead of the switcher, so the switcher only needs one connection no matter how many tally lights you use (ATEM switchers allow only 5–8 clients).

If the WiFi or switcher connection is lost, the tally light shows the connecting color and reconnects on its own.

# Changes in this fork
Compared to the original project:

**ESP8266**
- **Tally number button** — set the tally number without opening the setup page: hold the button, then press it as many times as the tally number. See [Tally number button](#tally-number-button).
- **Battery monitoring** — measures the battery voltage directly on the cell (one resistor to A0), blinks red 3 times every minute below 3.5 V and shows the voltage on the setup page. See [Battery monitoring](#battery-monitoring).

**M5StickC / Plus / Plus2**
- New `M5StickC` build: the display shows the tally color and number, buttons change the tally number, WiFi signal and battery level on screen, early connection warnings, charger settings for a large replacement battery. See [M5StickC](#m5stickc).

**All boards**
- **Setup access point only after power on.** The original starts the "Tally Light setup" access point every time WiFi is lost. This fork starts it only if the network can't be found after power on; when WiFi drops later, the tally light just reconnects. Power cycle it to get back to setup.
- **Blue light at startup.** The original sets the startup color before reading the settings, while the LED brightness is still 0, so the LED stays dark until WiFi is found. This fork lights it right away.
- **Defaults on a freshly flashed board** (name "Tally", tally 1, normal mode, full brightness) instead of random values from empty memory.

The extra code takes about 3 KB of flash and 350 bytes of RAM on ESP8266.

# Supported boards
| PlatformIO environment | Board |
|------------------------|-------|
| `ESP8266` | Wemos D1 mini, NodeMCU and other ESP8266 boards |
| `M5StickC` | M5StickC, M5StickC Plus, M5StickC Plus2 (model detected at runtime) |
| `ESP32` | Generic ESP32 dev board |
| `ESP32_C3`, `ESP32_S2`, `ESP32_S3` | ESP32-C3 / S2 / S3 dev boards |
| `ATOM_Matrix` | M5Stack ATOM Matrix |

Each environment also has a `_test_server` variant that simulates a switcher (see [Test server](#test-server)).

# Building and uploading
Install [PlatformIO](https://platformio.org/) (for example the *PlatformIO IDE* extension for VS Code), open this folder, select the environment and upload, or from the command line:

```
pio run -e ESP8266 -t upload
```

Uploading new firmware keeps the saved settings and WiFi network.

# Setup
On first start (or when the saved WiFi network can't be found after power on) the tally light starts an access point named **"Tally Light setup"**. Connect to it and open [192.168.4.1](http://192.168.4.1) to set:

- tally light name and number (camera input)
- tally mode for LED 1 and LED 2
- LED and Neopixel brightness, number of Neopixels and status LED position
- WiFi network and password, optional static IP
- ATEM switcher IP — or the IP of another tally light to receive tally data from it

After it has connected to WiFi, the same page is available on the tally light's own IP address. It also shows the connection status and, on ESP8266 with battery monitoring, the battery voltage.

# ESP8266
### Wiring
| Function | Pin |
|----------|-----|
| LED 1 red / green / blue | D0 / D2 / D1 |
| LED 2 red / green / blue | D4 / D5 / D6 |
| Neopixel data (optional) | D7 |
| Tally number button | D3 → button → GND |
| Battery voltage | Battery + → 100 kΩ → A0 |

Pins can be changed with `build_flags` in `platformio.ini` (`PIN_RED1`, `TALLY_DATA_PIN`, `TALLY_BUTTON_PIN`, `TALLY_BATTERY_PIN` …). The button and battery parts are optional: without them the tally light works as usual.

### Tally number button
Connect a push button between **D3** and **GND** — no resistor needed, D3 is pulled up on the board. On NodeMCU the built-in FLASH button can be used. Don't hold the button while powering on, or the ESP8266 starts in flash mode.

| Action | Result |
|--------|--------|
| Short press | Blinks the current tally number in green (5 blinks = tally 5) |
| Hold 1.5 s | The light blinks yellow — release, then press as many times as the tally number. Each press lights it white |
| Wait 3 s after the last press | The number is saved and blinked back in green |
| Hold again while yellow | Cancels with 3 red blinks. Also cancelled after 10 s without any press, or for numbers above 41 |

The number is saved permanently and is the same as on the setup page. While the button is in use, the LEDs show the button feedback instead of the tally; the connection to the switcher and the tally server keep running.

### Battery monitoring
For a tally light running on a Li-ion cell (for example a phone battery with a charger board and a 5 V boost converter), connect the **battery +** (B+ on the charger board) to **A0** through a **100 kΩ** resistor. The ground is already shared through the charger and converter.

The D1 mini and NodeMCU have a 220k/100k divider on A0; with the extra 100 kΩ the measuring range is 0–4.2 V. In a test it showed 4.14 V with 4.128 V on a multimeter. The resistor draws about 13 µA.

- The voltage is measured every second and smoothed, so WiFi transmit spikes don't cause false warnings.
- At or below **3.5 V** the light **blinks red 3 times every minute**. The warning stops above 3.6 V (when charging).
- The setup page shows the voltage ("Battery voltage", with "(low)" when low), and it is logged to the serial port every minute.
- With nothing connected A0 reads 0 V and battery monitoring is off.

A 1 MΩ resistor works too, less precisely: build with `-D BATTERY_FULL_SCALE_V=13.2`. To calibrate, compare the setup page with a multimeter and adjust `BATTERY_FULL_SCALE_V` (voltage at full scale) in proportion.

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

The tally number button and battery monitoring are ESP8266-only by default. The button can be enabled on ESP32 with `-D TALLY_BUTTON_PIN=<pin>`.

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

Short blinks on ESP8266:

Blinks | Description
-------|------------
YELLOW, continuous | Waiting for tally number button presses
GREEN × N | Tally number N (after a short press, or saved after entering it)
RED × 3 | Tally number entry cancelled
RED × 3, every minute | Battery at or below 3.5 V

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
