# WW2 Dogfight — Flight Controller Hardware

The hardware and firmware side of **WW2 Dogfight**: a realistic flight controller built into a
single joystick, so a player can fly the entire flight deck without ever leaving the stick.

> **Status:** early bring-up. The PlatformIO project builds and flashes to an ESP32-S3, and the
> current `src/main.cpp` is a hardware sanity check (onboard LED blink + serial heartbeat).
> Sensor, axis and HID work is in progress — see [Roadmap](#roadmap).

---

## What this project is

Most flight sims make you pay for a stick, then a separate throttle quadrant, then pedals, then a
panel of switches. This project collapses that whole setup into **one physical joystick**.

The stick is the aircraft. Physical travel on the grip drives the flight controls, an onboard
microcontroller reads those inputs, and the result is presented to the PC as a standard USB game
controller. From the game's point of view it is a joystick with a lot of axes and buttons; from the
player's point of view it is the cockpit.

That means the *feel* matters as much as the electronics: control travel, centring, dead zones and
smoothing are tuned so a small physical movement produces a believable control input rather than a
twitchy digital on/off.

This repository is the **hardware half** of WW2 Dogfight. The game/simulation side lives in a
separate repository (link to be added).

## Design goals

- **One device, full flight deck.** Primary flight controls plus the controls a pilot reaches for
  mid-fight, without moving a hand to another peripheral.
- **Realistic but playable.** Response curves and filtering that reward smooth inputs, not just
  maximum resolution.
- **Self-contained.** Plugs in over USB, no separate driver or companion app required.
- **Serviceable.** Standard dev board, hand-solderable wiring, documented pin map, so the stick can
  be opened, repaired and recalibrated.

## How it works

```
 physical input  ->  ESP32-S3  ->  USB HID  ->  WW2 Dogfight
 (grip axes,        (sampling,     (joystick
  buttons,           filtering,     report)
  switches)          mapping)
```

1. **Sense** — axes are read as analog voltages and discretes as digital inputs.
2. **Condition** — each axis is calibrated at its mechanical limits, then passed through a dead
   zone, an expo/response curve and light smoothing so output is proportional and stable.
3. **Report** — the conditioned values are packed into a HID joystick report and sent over USB to
   the host at a fixed polling rate.

### Input map (draft)

Final channel list is still being nailed down; this is the working target.

| Channel | Type | Suggested control |
| --- | --- | --- |
| Pitch | Analog axis | Stick fore/aft |
| Roll | Analog axis | Stick left/right |
| Yaw | Analog axis | Stick twist or rocking lever |
| Throttle | Analog axis | Lever on the grip base |
| Guns | Digital button | Trigger |
| Cannon / secondary | Digital button | Thumb button |
| Flaps up / down | Digital button | Two-position switch |
| Gear | Digital toggle | Toggle switch |
| View / look | Digital hat | 4-way hat |
| Trim | Analog or encoder | Wheel on grip |

## Hardware

| Part | Notes |
| --- | --- |
| ESP32-S3 DevKitC-1 | Main controller, native USB for HID |
| Analog sensors | _TBD — hall-effect or potentiometer per axis_ |
| Buttons / switches | _TBD — trigger, toggles, hat_ |
| Enclosure / grip | _TBD — donor joystick shell or printed grip_ |

### Pin map

_To be filled in once the harness is fixed. Keep this table in sync with the firmware — it is the
documentation people will actually use when something stops working._

| Signal | GPIO | Notes |
| --- | --- | --- |
| Pitch | — | ADC |
| Roll | — | ADC |
| Yaw | — | ADC |
| Throttle | — | ADC |
| Trigger | — | digital, pull-up |
| LED (status) | `LED_BUILTIN` | bring-up only |

## Repository layout

```
platformio.ini   PlatformIO environment (ESP32-S3 DevKitC-1, Arduino framework)
src/main.cpp     Firmware entry point
include/         Project headers
lib/             Project-private libraries
test/            Firmware tests
```

## Building and flashing

Requires [PlatformIO](https://platformio.org/) (VS Code extension or CLI) and an ESP32-S3 board
connected over USB.

```bash
pio run                 # build
pio run --target upload # flash
pio device monitor      # serial output at 115200
```

Current bring-up firmware prints a heartbeat to serial and blinks the onboard LED. Change the
monitor speed to match `Serial.begin()` in `src/main.cpp` if you alter it.

> **Boot mode:** if upload fails, hold **BOOT**, tap **RESET**, release **BOOT**, then retry.

## Calibration

Planned flow: move every axis to both mechanical limits on first boot, firmware records min/max,
values are stored in NVS so calibration survives power cycles. Until that lands, axis limits are
compile-time constants.

## Roadmap

- [x] PlatformIO project, board config, serial + LED bring-up
- [ ] Read all analog axes, verify wiring and noise floor
- [ ] Calibration routine persisted to NVS
- [ ] USB HID joystick enumeration with full axis/button report
- [ ] Response curves and dead zones tuned against the game
- [ ] Grip hardware: trigger, hat, toggles
- [ ] Enclosure integration and cable strain relief

## Related

- **WW2 Dogfight** — the game / simulation this controller is built for: _link TBD_
- **Controller mapping** and in-game setup notes: _link TBD_

## License

_TBD — add a license before publishing so others know what they may reuse._
