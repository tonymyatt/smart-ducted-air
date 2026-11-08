# smart-ducted-air

ESPHome based smart ducted air conditioning, with per-zone temperature control in Home Assistant.

## My goal

I wanted to make my basic, non-smart zone controller and ducted air conditioner smart. With the old setup, zones could be opened and closed, but there was no way to set a temperature for each zone, and nothing I could automate or see from Home Assistant.

This project replaces that controller. Each zone becomes a Home Assistant climate entity with its own mode (off, cool, heat) and its own target temperature, and the system works out how to share the air between zones to reach those targets.

I elected not to use PID control for this application. Temperatures change slowly, so a simpler control approach is more than fit for purpose.

## How it works

- **Zone dampers.** 24VAC zone dampers are driven by relays on an ESP32 mounted in the roof, replacing the proprietary zone controller. Each of the 5 zones has an open and a close relay, and a time-based cover tracks the damper position.
- **Indoor unit.** The indoor unit is controlled over the Mitsubishi CN105 port, so Home Assistant can set its temperature and feed it a remote room temperature.
- **Zone climate entities.** Per-zone climate entities (Room 1 to Room 5) take their current temperature from a Home Assistant sensor and hold their own setpoint.
- **Control logic.** The zone furthest from its setpoint is the "winner". Its setpoint and temperature are sent to the AC unit, and the other dampers open pro-rata to how far each zone is from target. Zones that are off are fully closed.
- **Wall panel.** The proprietary wall controller is replaced with an ESP32-S3 touch screen showing actual temperature, setpoint, +/- buttons and an on/off switch for every zone. ESPHome syncs directly with Home Assistant entities, so the display and Home Assistant offer the same controls and status.

## Repository layout

| Folder | Contents |
| --- | --- |
| [`esphome/`](esphome/) | ESPHome configs: [`ductedairroof.yaml`](esphome/ductedairroof.yaml) (dampers, CN105, zone logic) and [`ductedairscreen.yaml`](esphome/ductedairscreen.yaml) (touch panel UI) |
| [`hardware/`](hardware/) | Bill of materials ([`bom.md`](hardware/bom.md)) and wiring diagrams |
| [`3dmodels/`](3dmodels/) | 3D printed parts (TODO) |
| [`references/`](references/) | Useful reference material |

## Damper wiring

The dampers use a standard RJ12 (6P6C) connector with the following pin allocation:

| Pins | Function |
| --- | --- |
| 1 & 2 | 24VAC drive open |
| 3 & 4 | 24VAC neutral / common |
| 5 & 6 | 24VAC drive closed |

## Getting started

1. Make sure your dampers are 24VAC, and test that they use the standard wiring shown above.
2. Build the in-roof unit from the bill of materials and wiring diagram, and print the enclosure if you want one.
3. Add `api_key`, `wifi_ssid`, `wifi_password` and `wifi_fallback` to your ESPHome `secrets.yaml`.
4. In `ductedairroof.yaml`, point each room's temperature sensor at your own Home Assistant entity.
5. In `ductedairscreen.yaml`, update the `ha_climate_roomN` substitutions if your device name differs.
6. Flash `ductedairroof.yaml` to the ESP32 and `ductedairscreen.yaml` to the ESP32-S3 touch screen.

## Find me

- MakerWorld: [@tonymyatt](https://makerworld.com/en/@tonymyatt)
- Home Assistant Community: [tonymyatt](https://community.home-assistant.io/u/tonymyatt/)

## Licence

Apache License 2.0. See [LICENSE](LICENSE).
