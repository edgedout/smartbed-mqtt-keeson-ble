# SmartBed MQTT Bridge

> Control compatible Tempur-Pedic and Keeson adjustable beds from Home Assistant using MQTT and an ESPHome Bluetooth Proxy.

![Home Assistant](https://img.shields.io/badge/Home%20Assistant-MQTT-blue)
![ESPHome](https://img.shields.io/badge/ESPHome-2026.x-green)
![Node.js](https://img.shields.io/badge/Node.js-20+-brightgreen)
![License](https://img.shields.io/badge/License-MIT-lightgrey)

---

## Overview

SmartBed MQTT Bridge connects compatible Bluetooth-enabled adjustable beds to Home Assistant.

It communicates directly with the bed over BLE through an ESPHome Bluetooth Proxy and exposes all controls using MQTT Discovery, allowing the bed to appear as a native Home Assistant device.

This fork modernizes the original project for current ESPHome Bluetooth Proxy firmware and has been fully validated on a real Tempur-Pedic Sleeptracker Gen2 adjustable base.

---

## Verified Hardware

This project has been tested with:

| Component | Version |
|-----------|---------|
| Tempur-Pedic Adjustable Base | Sleeptracker Gen2 |
| BLE Device | KSBT03C201099417 |
| ESPHome Bluetooth Proxy | 2026.5.1 |
| ESPHome API | 1.14 |
| Home Assistant | Current |
| MQTT | Mosquitto |
| Host | Raspberry Pi |

---

## Features

- Home Assistant MQTT Discovery
- ESPHome Bluetooth Proxy support
- ESPHome 2026.x compatible
- Encrypted ESPHome API support
- Automatic BLE discovery
- Persistent BLE connections
- Automatic reconnects
- Raw Bluetooth V2 advertisement support
- KSBT controller support
- BaseI4 support
- BaseI5 support

### Bed Controls

- Flat
- Zero G
- Anti-Snore
- Memory Presets
- Head Lift
- Foot Lift
- Lumbar Lift
- Tilt
- Massage
- Massage Timer

---

# Architecture

```
                Bluetooth LE
        ┌─────────────────────────┐
        │ Tempur-Pedic / Keeson   │
        └─────────────┬───────────┘
                      │
                      │
              ESPHome Bluetooth Proxy
                      │
               ESPHome API (6053)
                      │
                      │
          SmartBed MQTT Bridge (Node.js)
                      │
                    MQTT
                      │
               Home Assistant
```

---

# Requirements

- Home Assistant
- MQTT Broker
- Dedicated ESPHome Bluetooth Proxy
- Node.js 20+
- Compatible Keeson / Tempur-Pedic adjustable bed

---

# Why a Dedicated Bluetooth Proxy?

ESPHome Bluetooth advertisements can only be subscribed to by a single API client.

This bridge requires ownership of the BLE advertisement stream.

For best results:

```
Bed
 │
Dedicated ESPHome Proxy
 │
SmartBed MQTT Bridge
 │
MQTT
 │
Home Assistant
```

Do **not** connect the dedicated proxy to Home Assistant.

Your other Bluetooth proxies can continue working normally.

---

# Configuration

Example configuration:

```json
{
  "mqtt_host": "192.168.x.xxx",
  "mqtt_port": 1883,

  "bleProxies": [
    {
      "host": "esp32-bluetooth-proxy.local"
    }
  ],

  "keesonDevices": [
    {
      "name": "KSBT0xxxxxxxxx",
      "friendlyName": "Master Bed"
    }
  ]
}
```

---

# Installation

```
git clone https://github.com/edgedout/smartbed-mqtt-keeson-ble.git

cd smartbed-mqtt-keeson-ble

npm install

npm run build
```

Run:

```
MQTTHOST=192.168.x.xxx \
MQTTPORT=1883 \
MQTTUSER="insert_your_user" \
MQTTPASSWORD="insert_your_password" \
node dist/tsc/index.js
```

---

# Running as a Service

Example `systemd` unit:

```ini
[Unit]
Description=SmartBed MQTT Bridge
After=network-online.target

[Service]
Type=simple

WorkingDirectory=/root/smartbed-mqtt-keeson-ble

Environment=MQTTHOST=192.168.2.203
Environment=MQTTPORT=1883
Environment=MQTTUSER=
Environment=MQTTPASSWORD=

ExecStart=/root/.nvm/versions/node/v20.20.2/bin/node /root/smartbed-mqtt-keeson-ble/dist/tsc/index.js

Restart=always
RestartSec=5

User=root

[Install]
WantedBy=multi-user.target
```

Enable:

```
sudo systemctl daemon-reload
sudo systemctl enable --now smartbed-mqtt.service
```

Logs:

```
journalctl -u smartbed-mqtt.service -f
```

---

# Supported BLE Services

Validated Nordic UART service:

```
Service UUID:
6e400001-b5a3-f393-e0a9-e50e24dcca9e

Write Characteristic:
6e400002-b5a3-f393-e0a9-e50e24dcca9e

Notify Characteristic:
6e400003-b5a3-f393-e0a9-e50e24dcca9e
```

---

# ESPHome Compatibility

This fork adds compatibility with modern ESPHome releases.

### Changes

- frameAndSend() transport support
- Raw BLE Advertisement V2 support
- API 1.14 compatibility
- Correct Bluetooth message IDs
- Modern scanner state handling
- KSBT03 discovery using BLE name only
- Notification subscription support
- Persistent BLE connections

---

# Current Status

## Fully Working

- Device discovery
- BLE connection
- Service discovery
- Characteristic writes
- Home Assistant discovery
- All movement commands
- Presets
- Massage
- Automatic reconnect

## Known Limitations

The tested KSBT03 firmware accepts notification subscriptions but does not emit position updates while the physical remote is used.

Because of this, Home Assistant currently operates the bed as a command-only device rather than reporting live position.

This appears to be a firmware limitation rather than an ESPHome limitation.

---

# Troubleshooting

### Bed not discovered

- Verify BLE device name
- Verify ESPHome proxy
- Verify proxy is **not** connected to Home Assistant
- Move proxy closer to bed

### MQTT entities missing

- Verify MQTT Discovery
- Restart Home Assistant
- Verify broker connectivity

### Commands do nothing

Verify:

- GATT services discovered
- Correct write characteristic found
- MQTT command published
- BLE proxy connected

---

# Development

```
npm run build

npm run test

npm run lint
```

---

# Acknowledgements

Original project by Richard Hopton.

Keeson support by phdindota.

ESPHome 2026 compatibility, Tempur-Pedic Gen2 validation, Bluetooth Proxy modernization, and KSBT03 support by the contributors to this fork.

---

# License

MIT
