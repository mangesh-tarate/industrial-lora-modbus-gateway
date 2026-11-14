# Industrial LoRa-Based Modbus RTU Monitoring System

**Wireless Industrial Monitoring | Embedded Firmware | RS485 | LoRa | MQTT**

An industrial IoT system that collects data from Modbus RTU field devices through RS485 wireless nodes
and delivers it to a central gateway over secure LoRa communication.

The gateway verifies the incoming data and publishes it to an MQTT broker as structured JSON, ready for
dashboards, SCADA, data logging and cloud platforms.

---

## Project Overview

The system removes long-distance RS485 cabling: field devices are wired to a nearby wireless node,
and the data travels to the gateway over LoRa.

The system consists of:

- **LoRa Modbus Node:** connects to Modbus RTU devices over RS485, polls the configured registers and sends the data wirelessly.
- **LoRa Gateway:** receives and verifies node data, confirms every transmission and publishes the data over Wi-Fi/MQTT.
- **Monitoring Platform:** any MQTT-capable dashboard, SCADA, database or cloud service.

---

## Project Prototype

<p align="center">
  <img src="Images/LoRa%20Gateway%20and%20Modbus%20Node.png" alt="LoRa Gateway and Modbus Node Prototype" width="700">
</p>

<p align="center"><i>Industrial LoRa Gateway and Modbus Node hardware prototype</i></p>

---

## System Architecture

```
       INDUSTRIAL FIELD DEVICES

    +----------+  +----------+  +----------+
    | Modbus   |  | Modbus   |  | Modbus   |
    | Device 1 |  | Device 2 |  | Device N |
    +-----+----+  +-----+----+  +-----+----+
          |             |             |
          +-------------+-------------+
                        |
                     RS485
                        |
               +--------+--------+
               |  LoRa Modbus    |
               |     Node        |
               | Register polling|
               | Status checking |
               +--------+--------+
                        |
             LoRa Wireless (encrypted)
                        |
               +--------+--------+
               |   LoRa Gateway  |
               | Verification    |
               | Acknowledgement |
               | JSON conversion |
               +--------+--------+
                        |
                  Wi-Fi / MQTT
                        |
               +--------+--------+
               |   MQTT Broker   |
               +--------+--------+
                        |
               +--------+--------+
               | Dashboard /     |
               | SCADA / Cloud   |
               +-----------------+
```

---

## Key Features

### Modbus RTU Field Communication
- Works with standard Modbus RTU slave devices (energy meters, level sensors, flow meters, PLCs, drives and more).
- Multiple Modbus devices per wireless node.
- Configurable slave addresses, function codes (FC01–FC04), register ranges and serial settings (baud rate, parity, stop bits).
- Built-in diagnostics: device scan, one-shot register read and raw serial test.
- Per-device status in every report: OK, no response, communication error or device exception.

### Secure LoRa Wireless Link
- Long-range communication between field nodes and the gateway on the 865–867 MHz band (India).
- AES-128 encryption with message authentication on every transmission.
- Protection against replayed or modified messages.
- Every transmission is acknowledged by the gateway.
- Link quality reporting (RSSI / SNR) in both directions.
- Status LEDs on node and gateway for RX/TX activity.

### Secure Device Onboarding
- Each device has a unique factory ID (MFID) assigned at production, including its manufacturing date.
- Nodes are added to a gateway using their MFID. Network credentials are delivered securely and automatically, with no manual key entry on site.
- Only authorised devices can join and publish data; unknown or unverified devices are blocked and reported.
- Automatic reconnection if a node loses contact with its gateway.
- Warnings for duplicate device IDs.

### Gateway and MQTT Integration
- Structured JSON telemetry over MQTT, compatible with any standard broker.
- Device information and gateway online/offline status published as retained messages.
- Configurable MQTT topic prefix and gateway ID to fit existing topic structures.
- Primary and backup Wi-Fi networks with automatic failover.
- Live log of received data and published messages for commissioning.

### Easy Configuration
- **Gateway:** interactive settings menu over USB (Wi-Fi, MQTT and node management), plus command-line control.
- **Node:** serial configuration console for Modbus devices, polling and reporting interval.
- Settings stored in non-volatile memory and kept across power cycles and firmware updates.

### Production Ready
- Production flashing tool that programs the firmware and assigns the factory MFID, with an automatic production log for traceability and labelling.

---

## Technical Specifications

| Parameter | Current design |
|---|---|
| Wireless technology | LoRa |
| Radio platform | STM32WL LoRa SoC (Ai-Thinker RA-09H) |
| Frequency band | 865–867 MHz (IN865) |
| Gateway controller | ESP32-C6 (Wi-Fi) |
| Field communication | Modbus RTU master |
| Physical interface | RS485 |
| Gateway output | MQTT (JSON) |
| Security | AES-128 encryption and authentication, authorised-device onboarding |
| Default reporting interval | 2 seconds (configurable) |
| Modbus devices per node | Up to 8 |
| Registers per node | Up to 40 |
| Nodes per gateway (firmware capacity) | Up to 16 |

---

## MQTT Data Integration

**Topics**

```
<prefix>/<gateway-id>/node/<n>/data      live telemetry
<prefix>/<gateway-id>/node/<n>/info      node configuration and identity (retained)
<prefix>/<gateway-id>/status             gateway online / offline (retained)
```

Example: `plant/gw01/node/1/data`

**Example telemetry**

```json
{
  "node": 1,
  "fcnt": 1024,
  "rssi": -85,
  "snr": 8,
  "sf": 7,
  "pwr": 14,
  "cycles": 100,
  "blocks": [
    {
      "slave": 1,
      "fc": 3,
      "addr": 0,
      "count": 6,
      "st": "ok",
      "v": [230, 231, 229, 12, 15, 18]
    }
  ]
}
```

`st` reports the communication status of each Modbus device, so your platform can tell fresh values
from stale ones. Field values depend on the connected devices and the node configuration.

---

## Industrial Applications

- Remote industrial instrumentation monitoring
- Energy meter data acquisition
- Fuel and tank level monitoring
- Pump and motor monitoring
- Water treatment and utility monitoring
- Remote sensor networks
- Distributed industrial automation
- Agricultural equipment monitoring
- Retrofitting existing Modbus devices with wireless connectivity

---

## Engineering Focus

- Reliable embedded firmware architecture
- Robust RS485 / Modbus communication
- Compact, efficient wireless protocol
- Secure communication and device onboarding
- Fault detection and diagnostics
- Simple field commissioning
- MQTT-based industrial data integration

---

## Development Roadmap

- [x] Modbus RTU data acquisition
- [x] LoRa node-to-gateway communication with acknowledgement
- [x] Per-device communication status reporting
- [x] AES-128 encrypted and authenticated communication
- [x] MQTT JSON telemetry with retained device information and gateway status
- [x] Unique factory device ID (MFID) and production flashing tool
- [x] Secure node onboarding with authorised-device list
- [x] Gateway settings menu, Wi-Fi failover and configurable MQTT topics
- [x] Node and gateway RX/TX status LEDs
- [ ] Remote node configuration from the gateway
- [ ] Adaptive Data Rate (ADR)
- [ ] Gateway-managed time-slot scheduling for dense multi-node networks
- [ ] Expanded multi-node field validation
- [ ] Long-duration reliability and packet-loss testing
- [ ] Production hardware validation

The current release is validated with single-node operation at a 2-second reporting interval.
Time-slot scheduling is planned to support many nodes per gateway at short intervals.

---

## Project Information

**Project Category:** Industrial IoT / Embedded Systems

**Core Technologies:** Embedded C, STM32WL, ESP32-C6, LoRa, Modbus RTU, RS485, MQTT, AES-128

**Development:** Hardware and firmware development, wireless communication, protocol design and system integration.

This repository documents the system architecture, features and development progress. Source code,
protocol internals and security implementation details are not published.
