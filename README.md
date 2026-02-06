# Ikea UPPÅTVIND Smart Air Purifier (ESP32-C6)

A custom firmware setup to make the IKEA UPPÅTVIND air purifier "smart" using an **ESP32-C6 SuperMini** and ESPHome.

## Features
- **WiFi Connectivity**: Full control via Home Assistant or Web Interface.
- **Fan Control**: Off, Low, Medium, High (exact speed matching).
- **Environmental Sensors**: Temperature, Humidity, and Pressure (via AHT20 + BMP280).
- **Bluetooth Provisioning**: Easily update WiFi credentials via phone (Improv Wi-Fi).
- **Fallback Hotspot**: Automatically creates a WiFi network if connection fails.
- **Reliability**: Self-correcting loop for fan speed adjustment.

---

## Wiring Schematic

```
┌─────────────────────────────────────────────────────────────────┐
│                    IKEA UPPÅTVIND PCB                           │
│                                                                 │
│   TP2 (GND) ──────┐                                             │
│   TP3 (5V)  ──────┼───┐                                         │
│   TP4 (BTN) ──────┼───┼───┐                                     │
│   TP7 (LED) ──────┼───┼───┼───┐                                 │
│                   │   │   │   │                                 │
└───────────────────┼───┼───┼───┼─────────────────────────────────┘
                    │   │   │   │
                    │   │   │   │
┌───────────────────┼───┼───┼───┼─────────────────────────────────┐
│                   │   │   │   │     ESP32-C6 SuperMini          │
│                   │   │   │   │                                 │
│   GND ────────────┘   │   │   │                                 │
│   5V (USB/VIN) ───────┘   │   │                                 │
│   GPIO0 ──────────────────┘   │  (Button simulation, open-drain)│
│   GPIO1 ──────────────────────┘  (LED pulse width sensing)      │
│                                                                 │
│   GPIO21 (SDA) ────────────────┐                                │
│   GPIO22 (SCL) ────────────────┼──┐                             │
│   3.3V ────────────────────────┼──┼──┐                          │
│   GND  ────────────────────────┼──┼──┼──┐                       │
│                                │  │  │  │                       │
└────────────────────────────────┼──┼──┼──┼───────────────────────┘
                                 │  │  │  │
                                 │  │  │  │
┌────────────────────────────────┼──┼──┼──┼───────────────────────┐
│                                │  │  │  │   AHT20+BMP280 Board  │
│   SDA ─────────────────────────┘  │  │  │                       │
│   SCL ────────────────────────────┘  │  │                       │
│   VDD ───────────────────────────────┘  │                       │
│   GND ──────────────────────────────────┘                       │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### Connection Summary

| UPPÅTVIND | ESP32-C6 | Function |
|-----------|----------|----------|
| TP2 | GND | Ground |
| TP3 | 5V/VIN | Power |
| TP4 | GPIO0 | Button press simulation |
| TP7 | GPIO1 | LED brightness sensing (fan speed) |

| Sensor | ESP32-C6 | Function |
|--------|----------|----------|
| SDA | GPIO21 | I2C Data |
| SCL | GPIO22 | I2C Clock |
| VDD | 3.3V | Power |
| GND | GND | Ground |

---

## Setup Instructions

1. **Install ESPHome**:
   ```bash
   pip install esphome
   ```

2. **Configure Secrets**:
   Copy `secrets.yaml.example` to `secrets.yaml` and enter your WiFi credentials.
   ```bash
   cp secrets.yaml.example secrets.yaml
   # Edit secrets.yaml
   ```

3. **Compile & Upload**:
   ```bash
   esphome run uppatvind-esp32c6.yaml
   ```

---

## Usage

### Web Interface
Access the device IP in your browser (e.g., `http://uppatvind-esp32c6.local`) to view sensors and control the fan.

### Fan Speed Logic
The ESP32 reads the LED pulse width to confirm the actual speed.
- **0-1%**: Off
- **2-33%**: Low Speed
- **34-66%**: Medium Speed
- **67-100%**: High Speed

### Connectivity & Recovery
This device supports two robust methods for establishing network connectivity.

#### 1. Bluetooth Provisioning (Improv Wi-Fi)
This is the **primary method** for initial setup or re-configuring WiFi without physical access.
- **Protocol**: Improv Wi-Fi over Bluetooth Low Energy (BLE).
- **Usage**:
    1. Power on the device.
    2. Open a compatible client (e.g., Home Assistant Companion App on Android/iOS, or [improv-wifi.com](https://www.improv-wifi.com/) on a desktop with BLE).
    3. Select "Uppatvind Air Purifier" from the list.
    4. Enter your WiFi SSID and Password securely.
- **Fallback Behavior**: Bluetooth provisioning remains active even if WiFi connects, allowing you to update credentials on the fly if you move the device to a new network.

#### 2. Fallback Hotspot (AP Mode)
If the device **cannot connect** to the configured WiFi network after 60 seconds (default), it creates its own emergency WiFi network.
- **Trigger**: WiFi connection failure.
- **SSID**: `Uppatvind-Fallback`
- **Password**: `fallback123`
- **Action**:
    1. Connect to this network with your phone or laptop.
    2. A "Captive Portal" should open automatically. (If not, navigate to `http://192.168.4.1`).
    3. Select your home WiFi network from the scanned list and enter the password.
    4. The device will save credentials and reboot.

### Home Assistant Entities
When connected to Home Assistant, the following entities are available:
- **Fan**: Control fan speed (Off, Low, Medium, High).
- **Sensors**: Temperature, Humidity, Atmospheric Pressure, UPPÅTVIND Fan Speed LED (raw pulse width).
- **Diagnostics**: IP Address, SSID, MAC Address, ESPHome Version.
- **Configuration**: Restart button.


---

## Notes
- **Power**: The UPPÅTVIND provides 5V at TP3, which powers the ESP32.
- **Sensors**: Includes calibration offsets to account for board heat:
  - **AHT20**: -5.42°C
  - **BMP280**: -5.8°C
- **Optimization**: Debug logging is disabled by default for performance.
- **If you need help, look in the images folder**
- **All credits belong to the original creator, this is something I vibecoded and it happens to work really well**
