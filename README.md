# ATEM-tally-light

[![GitHub release](https://img.shields.io/github/v/release/AronHetLam/ATEM_tally_light_with_ESP8266)](https://github.com/AronHetLam/ATEM_tally_light_with_ESP8266/releases/latest)
[![License](https://img.shields.io/github/license/AronHetLam/ATEM_tally_light_with_ESP8266)](LICENSE)

<a href="https://www.buymeacoffee.com/aronhetlam" target="_blank"><img src="https://cdn.buymeacoffee.com/buttons/v2/default-yellow.png" alt="Buy Me A Coffee" style="height: 60px !important;width: 217px !important;" ></a>

Wireless tally light and 'On Air' sign for use with ATEM switchers. Connects over WiFi using only a D1 mini board (ESP8266 WiFi module) and a RGB LED or LED strip. This solution is __not__ limited by the ATEM switchers' connection limit, making it possible to connect as many as you need.

ESP32 is also supported. Depending on your board, you might need to change the pin numbers.

It should easily be convertable to use with regular Arduino boards and a WiFi module, by changeing the include statements and a few other things (however, this is not tested).

__DIY guide__ is available in the [wiki](https://github.com/AronHetLam/ATEM_tally_light_with_ESP8266/wiki/DIY-guide). __No coding required!__

[Video](https://www.youtube.com/watch?v=238HlUx3dgI) tutorial available by Josh from [@Budget Church Livestreaming](https://www.youtube.com/@budget-church-livestreaming)

# M5StickC version (this fork)
This fork adds support for the **M5StickC** (also M5StickC Plus / Plus2, detected at runtime via [M5Unified](https://github.com/m5stack/M5Unified)), using its built-in display, LED and buttons. Build and upload the `M5StickC` PlatformIO environment:

```
pio run -e M5StickC -t upload
```

### Display
- The whole screen shows the tally color (follows the *LED 1* mode) with the tally number in large digits. In *On Air* mode it shows "ON AIR".
- While connecting, the screen shows the state as text on the usual status colors (blue: WiFi, pink: ATEM, white: setup access point with its IP).
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

### WiFi setup access point
The "Tally Light setup" access point is only started when the WiFi network can't be found after power on. If WiFi is lost later, the tally keeps reconnecting without starting the access point; power cycle it to get back to setup. (This applies to all boards in this fork.)

### Battery
- Battery level uses a Li-ion discharge curve instead of a linear 3.3–4.15 V scale, with smoothing and a voltage calibration offset (`M5_BATTERY_OFFSET_MV`, calibrated against a multimeter on one unit — adjust for yours).
- The AXP192 charger (M5StickC / Plus) is set to 4.2 V and 450 mA (`M5_CHARGE_VOLTAGE_MV`, `M5_CHARGE_CURRENT_MA`), meant for a large replacement cell (tested with an iPhone 13 mini battery, 2406 mAh). The default 100 mA is too low for such a cell to finish charging before the AXP192 charge timer stops it. **With the stock small cell, lower `M5_CHARGE_CURRENT_MA` to 100.**

### Other changes
- A blank EEPROM (first flash) now gets default settings instead of garbage (name "Tally", tally 1, full brightness).

# What does it do?
Once set up, it will automatically connect to an ATEM switcher over WiFi and function as a tally light or 'On Air' sign.

When the program is uploaded to the ESP8266 the setup is done with a webpage it serves over WiFi where you are able to see status details, and perform the basic setup. Depending on if it's connecting to a known network or not it will serve the webpage on it's IP address, or on [192.168.4.1](HTTP://192.168.4.1) (default) over a softAP (access point) named "Tally light setup". For more details, see the guide int the [wiki](https://github.com/AronHetLam/ATEM_tally_light_with_ESP8266/wiki/DIY-guide).

As Atem swithcers only allow for 5-8 simultanious clients (dependant on the model) v2.0 introduced Tally Server functionality. This makes the system only require one connection from the switcher, as the tally lights can retransmit data to other tallys. An example setup is shown in the diagram below, where arrows indicate the direction of tally data from swtcher/tally unit to client tally unit.

![asdf](./Wiki/DIY_guide/img/Example_setup.jpg)

NOTE: As this brings a lot of flexibility with how to connect the units, bear in mind that the ESP8266 isn't that powerful, and is limited to 5 clients each. (In some cases 5 might even be too many).

## Connection and tally state indication
The different states of connection is signalled with LED colors.

Color | Description
------|--------
BLUE | Connecting to WiFi
WHITE | Unable to connect to WiFi - serves softAP "Tally light setup", while still trying to connect.
PINK | Connecting to ATEM Swithcher (Connected to WiFi).
RED | Tally program
GREEN | Tally preview
OFF | Tally neither live or preview (or no power. Blue LED on the D1 mini is on when not in program).
ORANGE | Connected and running (Only shown by status LED on addressable LED strip).

## Modes of operation
By default the tally light works as a normal tally light would in a professional enviroment, but other modes are available as described.

Mode | Description
-----|------------
Normal | As describen in the above table.
Preview stay on | Tally will be green whenever not in program
Program only | Tally will be off whenever not in program
On Air | Red when switcher is streaming, off otherwise.

Note: only some Atem models support streaming, so On Air mode only works with those models. On Air mode also needs a direct connection to switcher, not another tally unit. However, it still retransmits tally data to other units if needed.

# Use Arduino IDE with ESP8266 module
See details at [ESP8266](https://github.com/esp8266/Arduino) on how to setup and use ESP8266 modules like a regular Arduino board.
They have links for further documentation as well.

# Credits
Based on ATEM libraries for Arduino by [SKAARHOJ](https://www.skaarhoj.com/), available at Git repo: [SKAARHOJ-Open-Engineering](https://github.com/kasperskaarhoj/SKAARHOJ-Open-Engineering)

Inspired by [ATEM_Wireless_Tally_Light](https://github.com/kalinchuk/ATEM_Wireless_Tally_Light) (However, this works completely different)
