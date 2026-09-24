![picokit-16-ir-keys](https://raw.githubusercontent.com/mytechnotalent/picokit-16-ir-keys/main/picokit-16-ir-keys.png)

<br>

## FREE Reverse Engineering Self-Study Course [HERE](https://github.com/mytechnotalent/reverse-engineering)
## FREE Embedded Hacking Course [HERE](https://github.com/mytechnotalent/Embedded-Hacking)

<br>

# PICOKIT-16 IR KEYS

### Remote Key Actions on the LEDs and Authenticated Heartbeat
#### Lesson 16 of the Picokit Series

<br>

***
**LEGAL DISCLAIMER:**
The information, tools, and code provided in this repository and course are strictly for educational, research, and defensive purposes only.

You are explicitly prohibited from using any materials contained herein to access, test, modify, or exploit any device, network, or system that you do not own 100% or for which you do not have explicit, documented, and legally binding authorization to interact with.

By using this repository and course, you acknowledge and agree that:

1. Any illegal, unauthorized, or malicious use of this information is solely your responsibility.
2. The author(s) and contributor(s) of this repository and course shall not be held liable for any damages, legal repercussions, criminal charges, or unauthorized actions resulting from the use, misuse, or abuse of the contents herein.
3. You will comply with all applicable local, state, national, and international laws regarding cybersecurity and computer fraud.

**IF YOU DO NOT AGREE WITH THESE TERMS, DO NOT USE THIS REPOSITORY AND COURSE.**
***

<br>
<br>

## Overview

The sixteenth Picokit lesson. The node decodes NEC infrared frames from a
VS1838B receiver on GP5, maps each remote key to a named action shown on
the red, yellow, and green LEDs, and every five seconds it transmits an
authenticated heartbeat over LoRa to a Python gateway that logs and
displays it. It reuses the standard node shape and adds a remote key map on
top of the NEC decoder.

<br>

## What it teaches

- Mapping a decoded NEC command byte to a named action.
- Showing a named action on several GPIO outputs.
- Sealing a tiny JSON body with Argon2id and XChaCha20-Poly1305 and sending it
  with an AT+SEND over the RYLR998.
- The gateway side: receive, authenticate, reject, log, and display.

<br>

## Hardware

| Peripheral | Pico 2 pin | Role |
| --- | --- | --- |
| VS1838B OUT | GP5 | NEC infrared input |
| Red / Yellow / Green | GP16 / GP17 / GP18 | action display |
| Onboard LED | GP25 | heartbeat, one blink per transmit |
| RYLR998 | GP8 TX / GP9 RX | LoRa heartbeat |
| Debug Probe | SWCLK/SWDIO/GND, GP0/GP1 | SWD and the console |

<br>

## How it works

The node runs `monitor_step` in a loop. Every 200 ms it polls the VS1838B
on GP5 and maps a decoded key to a named action: 0x45 is up on the red
lamp, 0x46 is down on the yellow lamp, and 0x47 is ok on the green lamp.
Every 5 seconds it seals `{"n":16,"s":<seq>,"k":<key>}` with the field key
and sends it over LoRa. The gateway authenticates each frame and only then
parses it.

<br>

## Build and flash

```bash
cd firmware
cmake -S . -B build -G Ninja -DPICO_BOARD=pico2 -DPICO_PLATFORM=rp2350-arm-s
cmake --build build
openocd -f interface/cmsis-dap.cfg -f target/rp2350.cfg \
  -c "program build/picokit_16_ir_keys.elf verify reset exit"
```

<br>

## Watch the node

Open the console at 115200 and reset:

```text
BOOT
=== PICOKIT-16 IR KEYS // REMOTE KEY ACTIONS + HEARTBEAT ===
KEY 0x45 UP
KEY 0x46 DOWN
RX from 0x0001, N bytes
```

<br>

## The gateway

```bash
cd gateway
python3 -m venv .venv && source .venv/bin/activate
pip install -r requirements.txt
python3 listen.py --port /dev/cu.usbserial-A50285BI --hub 0001 --network 18 --db gateway.db
```

It prints `OK node=16 rssi=...` per authenticated heartbeat. The terminal
dashboard `python3 tui.py --db gateway.db` and the web dashboard
`python3 web/app.py --db gateway.db` show the same rows.

<br>

## Verify

```bash
python3 .opencode/skill/embedded-c-standard/audit_c_standard.py
python3 .opencode/skill/embedded-python-standard/audit_python_standard.py
python3 .opencode/skill/iot-readme-standard/validate_readme.py
python3 .opencode/skill/iot-banner-standard/validate_banner.py
python3 scripts/run_tests.py
python3 scripts/check_coverage.py
```

<br>

# Next
[picokit-17-ir-keypad](https://github.com/mytechnotalent/picokit-17-ir-keypad)

<br>

# License
[MIT License](https://github.com/mytechnotalent/picokit-16-ir-keys/blob/main/LICENSE)
