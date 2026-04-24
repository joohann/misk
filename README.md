# Misk 🌿

ESPHome firmware for repurposed Freshmatic-style air fresheners, retrofitted with a D1 Mini.

The name *Misk* (مسك) refers to musk — in Islamic tradition, the scent of paradise.

## Features

- Manual spray via physical button
- Automatic spray on configurable intervals (5/15/30/60/120 minutes — adjustable per device)
- Spray counter with "fill level" sensor (% of max)
- Counter reset by holding the button for 5 seconds
- Home Assistant integration via the ESPHome API
- Local web server on port 80
- Status LED feedback

## Hardware

- **MCU**: WeMos D1 Mini (ESP8266)
- **Spray driver**: GPIO (default `D8`) → transistor/MOSFET → motor of the air freshener
- **Button**: GPIO `D2` with internal pull-up
- **Status LED**: GPIO `D4` (inverted, uses the onboard LED)

## Repo structure

```
.
├── common/
│   └── misk.yaml          # Shared config for all devices
├── misk-bedroom.yaml      # Example device config
├── misk-livingroom.yaml   # Example device config
├── secrets.yaml.example   # Template for secrets.yaml
└── .gitignore
```

## Usage

### 1. Clone the repo into your ESPHome folder

```bash
cd /config/esphome   # or wherever your ESPHome configs live
git clone https://github.com/joohann/misk.git
```

Or just drop the files alongside your existing ESPHome configs.

### 2. Create your secrets file

```bash
cp secrets.yaml.example secrets.yaml
# fill in your wifi/api credentials
```

### 3. Add a new device

Create a file `misk-<room>.yaml` with the substitutions you want:

```yaml
substitutions:
  device: "misk-hallway"
  spray_pin: "D8"
  interval1: "10"
  interval2: "30"
  interval3: "60"
  interval4: "120"
  interval5: "240"
  squirt_pulse: "150ms"
  maxSquirts: "2500"

packages:
  misk: !include common/misk.yaml
```

### 4. Flash

```bash
esphome run misk-hallway.yaml
```

## Substitutions

| Name            | Description                                        | Example       |
|-----------------|----------------------------------------------------|---------------|
| `device`        | Hostname and friendly-name base                    | `misk-bedroom`|
| `spray_pin`     | GPIO pin driving the spray motor                   | `D8`          |
| `interval1..5`  | Available intervals in minutes                     | `5`, `15`, …  |
| `squirt_pulse`  | Spray pulse duration (reserved for future use)     | `150ms`       |
| `maxSquirts`    | Estimate of max sprays per can (for the % sensor)  | `2500`        |

## Remote include via Git

If you want to use the package on other HA instances without cloning the full repo:

```yaml
substitutions:
  device: "misk-bedroom"
  # ... other substitutions

packages:
  misk:
    url: https://github.com/joohann/misk
    ref: main
    files:
      - common/misk.yaml
    refresh: 1d
```

## Notes

- The `delay(150)` in the spray action is blocking. For 150ms this is fine in practice, but if you want to refactor it to non-blocking using an `output:` component, see the ESPHome docs.
- `restore_from_flash: true` keeps the spray counter across reboots — useful for maintaining a reasonably accurate fill level.

## License

MIT
