# Water Softener Salt Level Sensor - ESPHome

This project uses an ESP8266 D1 Mini with an HC-SR04 ultrasonic distance sensor to measure the salt level inside a water softener container and expose the data to Home Assistant via ESPHome.

The current ESPHome configuration is in [`salt_level_sensor.yaml`](salt_level_sensor.yaml).

## Features

- Salt level percentage calculated from measured distance
- Raw salt distance sensor in meters
- WiFi signal strength in dBm
- WiFi signal quality percentage
- Device uptime
- IP address, connected SSID, and BSSID
- ESPHome version sensor
- Encrypted Home Assistant API
- Password-protected OTA updates
- Web interface
- Fallback access point and captive portal for recovery
- Remote restart button

## Hardware Requirements

- ESP8266 D1 Mini
- HC-SR04 ultrasonic sensor, or compatible trigger/echo ultrasonic sensor
- Jumper wires
- 5V power supply, USB or regulated
- Water softener container
- Voltage divider or level shifter for the HC-SR04 ECHO pin

## Wiring

| HC-SR04 | D1 Mini |
| ------- | ------- |
| VCC     | 5V      |
| GND     | GND     |
| TRIG    | D1      |
| ECHO    | D2 via voltage divider or level shifter |

Important: the HC-SR04 ECHO pin outputs 5V. The ESP8266 GPIO pins are 3.3V only, so use a voltage divider or level shifter between ECHO and D2.

## Required ESPHome Secrets

Add these values to your ESPHome `secrets.yaml`:

```yaml
wifi_ssid: "your-wifi-name"
wifi_password: "your-wifi-password"
fallback_ap_password: "your-fallback-ap-password"
api_encryption_key: "your-32-byte-base64-api-key"
ota_password: "your-ota-password"
```

Generate a valid API encryption key with:

```bash
openssl rand -base64 32
```

Or in PowerShell:

```powershell
[Convert]::ToBase64String((1..32 | ForEach-Object { Get-Random -Maximum 256 }))
```

## How It Works

The HC-SR04 measures the distance from the sensor to the top of the salt. The YAML exposes this as `Salt Distance` and then calculates `Salt Level Percentage` from two calibration points:

- `empty_distance`: distance from sensor to salt level when the tank is considered empty
- `full_distance`: distance from sensor to salt level when the tank is considered full

Current defaults:

```yaml
empty_distance: "0.505"
full_distance: "0.080"
```

Formula:

```text
(empty_distance - distance) / (empty_distance - full_distance) * 100
```

The result is clamped between 0% and 100%.

## Calibration

1. Measure the distance from the sensor to the salt level when the tank should count as empty.
2. Set that value as `empty_distance` in meters.
3. Fill the tank to the level that should count as full.
4. Measure the distance from the sensor to the salt level again.
5. Set that value as `full_distance` in meters.

Example:

```yaml
empty_distance: "0.505"
full_distance: "0.080"
```

Because the sensor is mounted above the salt, `full_distance` normally should not be `0.000`.

## Home Assistant Entities

The main entities created by the YAML are:

- `Salt Level Percentage`
- `Salt Distance`
- `WiFi Signal`
- `WiFi Signal Quality`
- `Uptime`
- `IP Address`
- `Connected SSID`
- `Connected BSSID`
- `ESPHome Version`
- `Restart`

Home Assistant may generate entity IDs such as:

- `sensor.salt_level_percentage`
- `sensor.salt_distance`
- `sensor.wifi_signal`
- `sensor.wifi_signal_quality`

Check the exact entity IDs in Home Assistant after adoption.

## Home Assistant Automation - Low Salt Alert

Example automation:

```yaml
alias: Waterontharder - Zout bijna op
description: Notify when the salt level is low
trigger:
  - platform: numeric_state
    entity_id: sensor.salt_level_percentage
    below: 20
action:
  - service: notify.mobile_app_yourphone
    data:
      title: "Waterontharder"
      message: "Zoutniveau is laag. Tijd om bij te vullen."
mode: single
```

## Lovelace Gauge Card

```yaml
type: gauge
entity: sensor.salt_level_percentage
name: Zoutniveau
min: 0
max: 100
severity:
  green: 50
  yellow: 20
  red: 0
```

## Web Interface

Open the ESPHome web interface at:

```text
http://<device-ip>
```

## Sensor Alternatives

### JSN-SR04T

The JSN-SR04T is a waterproof ultrasonic distance sensor and is a better choice in damp environments. Some JSN-SR04T boards can work in HC-SR04-compatible trigger/echo mode, so the same `platform: ultrasonic` configuration can often be reused.

ESPHome also has a dedicated `jsn_sr04t` sensor platform that uses UART. That requires different YAML with a `uart:` section and `platform: jsn_sr04t`.

### VL53L1X

The VL53L1X is an optical time-of-flight sensor using I2C. It requires different wiring and different YAML with an `i2c:` bus. ESPHome has native support for VL53L0X, but VL53L1X commonly requires an external component, which can be less future-proof than built-in ESPHome components.

For this salt tank use case, HC-SR04 is the simplest option and JSN-SR04T is the most practical moisture-resistant upgrade.

## Future ESP Variants

This configuration currently targets an ESP8266 D1 Mini:

```yaml
esp8266:
  board: d1_mini
```

For newer ESP boards, especially ESP32-C3, ESP32-C6, or ESP32-S3, update the board section and use GPIO pin names instead of D1/D2 aliases. Example direction for an ESP32-C3 board:

```yaml
esp32:
  board: esp32-c3-devkitm-1
  framework:
    type: esp-idf

substitutions:
  trigger_pin: "GPIO5"
  echo_pin: "GPIO4"
```

The sensor logic can stay mostly the same, but always verify the actual GPIO pins for your specific board.

## 3D-Printable Lid - Aqmos R2D2-72

Custom lid with ESP and ultrasonic mount:

https://makerworld.com/en/models/1726087-aqmos-r2d2-72-lid-with-esp-and-level-sensor#profileId-1832738

Print in PETG or ASA for better humidity resistance.

## Troubleshooting

- If ESPHome reports `Secret 'fallback_ap_password' not defined`, add it to `secrets.yaml` or remove the fallback AP block.
- If ESPHome reports `Invalid key format` for `api_encryption_key`, generate a new base64 key with `openssl rand -base64 32`.
- If readings are inverted or inaccurate, re-measure `empty_distance` and `full_distance`.
- If readings jump around, check sensor mounting and make sure the sensor points at a reasonably flat part of the salt surface.
- If the ESP8266 becomes unstable, verify that HC-SR04 ECHO is reduced to 3.3V before reaching the GPIO pin.
- If the device is offline after changing WiFi, connect to the fallback AP and reconfigure it.

## License

MIT License
