# 💧 Waterontharder (Water Softener) Salt Level Sensor -- ESPHome

This project uses an ESP8266 D1 Mini with an ultrasonic distance sensor
to measure the salt level inside a water softener container and expose
that data to Home Assistant via ESPHome.

## Features

-   Salt level in percentage
-   WiFi signal strength
-   Device uptime
-   IP / SSID / BSSID
-   ESPHome version
-   OTA updates
-   Web interface
-   Remote restart switch

## Hardware Requirements

-   ESP8266 D1 Mini
-   Ultrasonic sensor (HC-SR04 or compatible)
-   Jumper wires
-   5V power supply (USB or regulated)
-   Water softener container

### Wiring

  Ultrasonic Sensor   D1 Mini
  ------------------- ---------
  VCC                 5V
  GND                 GND
  TRIG                D1
  ECHO                D2

⚠️ HC-SR04 ECHO outputs 5V. Use a voltage divider or level shifter for
ESP8266 (3.3V).

## How It Works

Distance is converted to fill percentage:

0.42 m = Empty (0%) 0.00 m = Full (100%)

Formula: (max_height - distance) / max_height \* 100

## ESPHome Configuration

(see repository for full YAML)

## Calibration

Replace MAX_HEIGHT with your container height (meters):

(max_height - x) \* (100 / max_height)

Example for 35 cm: (0.35 - x) \* (100 / 0.35)

## 3D-Printable Lid (Aqmos R2D2-72)

Custom lid with ESP & ultrasonic mount:
https://makerworld.com/en/models/1726087-aqmos-r2d2-72-lid-with-esp-and-level-sensor#profileId-1832738

Print in PETG or ASA for humidity resistance.

## Home Assistant Automation (Low Salt Alert)

Example:

alias: Waterontharder -- Zout bijna op trigger: - platform:
numeric_state entity_id: sensor.salt_level_percentage below: 20
action: - service: notify.mobile_app_yourphone data: title:
"Waterontharder" message: "Zoutniveau is laag. Tijd om bij te vullen."

## Lovelace Card (Gauge)

type: gauge entity: sensor.salt_level_percentage name: Zoutniveau min: 0
max: 100

## Web Interface

Open: http://`<device-ip>`{=html}

## Troubleshooting

-   Check WiFi secrets if offline
-   Re-measure container height for wrong readings
-   Use voltage divider on ECHO

## License

MIT License
