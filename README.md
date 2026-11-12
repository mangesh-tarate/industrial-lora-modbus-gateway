# Industrial LoRa-Based Modbus RTU Monitoring System

**Wireless Industrial Monitoring | Embedded Firmware | RS485 | LoRa | MQTT**

An industrial IoT communication system designed to collect data from multiple Modbus RTU field devices through RS485-based wireless nodes and transfer the acquired data to a central gateway using LoRa communication.

The gateway processes incoming data and forwards monitoring information to an MQTT broker for remote supervision and integration with industrial monitoring platforms.

---

## Project Overview

This project demonstrates a scalable wireless monitoring architecture for industrial automation and remote instrumentation applications.

The system consists of:

- **LoRa Modbus Node:** Communicates with Modbus RTU slave devices over RS485, collects register data and transmits it wirelessly.
- **LoRa Gateway:** Receives wireless data, manages acknowledgements and forwards acquired information through MQTT.
- **Monitoring Platform:** Receives MQTT data for visualization, storage and remote monitoring.

The architecture is designed to reduce long-distance RS485 wiring and simplify industrial equipment monitoring.

## Project Prototype


## Project Prototype

<p align="center">
  <img src="./Images/LoRa%20Gateway%20and%20Modbus%20Node.png"
       alt="LoRa Gateway and Modbus Node Prototype"
       width="400">
</p>

<p align="center">
  <em>Industrial LoRa Gateway and Modbus Node Hardware Prototype</em>
</p>

## System Architecture

```text
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
               |                 |
               | Modbus RTU      |
               | Register Polling|
               | Data Processing |
               +--------+--------+
                        |
                  LoRa Wireless
                        |
               +--------+--------+
               |   LoRa Gateway  |
               |                 |
               | Packet Handling |
               | ACK Management  |
               | Data Processing |
               +--------+--------+
                        |
                   MQTT / IP
                        |
               +--------+--------+
               |   MQTT Broker   |
               +--------+--------+
                        |
               +--------+--------+
               | Monitoring      |
               | Dashboard /     |
               | Cloud Platform  |
               +-----------------+
```

## Key Features

### Modbus RTU Communication

- RS485-based Modbus RTU master communication.
- Multiple Modbus slave devices per wireless node.
- Configurable slave addresses and register polling.
- Register block-based data acquisition.
- Communication timeout and exception handling.
- Per-block communication status reporting.

### LoRa Wireless Communication

- Wireless communication between field nodes and gateway.
- Compact binary packet format.
- DATA, ACK and INFO frame types.
- Frame counters and network/node identification.
- RSSI and SNR diagnostics.
- Communication timeout and link monitoring.
- AES-CCM authenticated encrypted payloads.

### Gateway and MQTT Integration

- Wireless packet reception and validation.
- Gateway acknowledgement handling.
- Modbus data forwarding through MQTT.
- Structured JSON telemetry.
- Node identification and communication diagnostics.
- Integration capability with external MQTT brokers and monitoring applications.

### Network Management

- Configurable data reporting intervals.
- Node and gateway communication management.
- Packet loss monitoring.
- Adaptive data rate and time-slot scheduling identified as planned network enhancements.

## Technical Specifications

| Parameter                      | Current design         |
| ------------------------------ | ---------------------- |
| Wireless technology            | LoRa                   |
| Radio platform                 | SX1262-based           |
| Example frequency              | 865.2 MHz              |
| Field communication            | Modbus RTU             |
| Physical interface             | RS485                  |
| Gateway communication          | MQTT                   |
| Security                       | AES-CCM authentication |
| DATA frame example             | 50 bytes               |
| ACK frame                      | 19 bytes               |
| INFO frame                     | 40 bytes               |
| Maximum configured slaves/node | 8                      |
| Maximum register units/node    | 40                     |
| Maximum configured blocks/node | 16                     |
| Gateway firmware node capacity | 16                     |

## LoRa Communication Protocol

The communication protocol uses compact binary frames to minimize wireless transmission overhead.

### DATA Frame

Example configuration: one block containing 12 registers.

| Field                           |         Size |
| ------------------------------- | -----------: |
| Header                          |      9 bytes |
| Fixed payload fields            |     11 bytes |
| Block status and exception code |      2 bytes |
| Register values                 |     24 bytes |
| AES-CCM authentication tag      |      4 bytes |
| **Total**                       | **50 bytes** |

The header contains the frame marker, type, network identifier, node identifier and frame counter.

Status is represented using binary fields rather than text strings.

### Other Frame Types

| Frame |               Size | Purpose                        |
| ----- | -----------------: | ------------------------------ |
| DATA  | 50 bytes (example) | Register telemetry             |
| ACK   |           19 bytes | Gateway acknowledgement        |
| INFO  |           40 bytes | Node information/configuration |

INFO frames are intended for boot-time transmission or configuration changes.

## Modbus Network Capacity

The current node configuration defines:

- Up to 8 Modbus slave devices per node.
- Up to 16 polling blocks per node.
- Up to 40 register units across all blocks.

At 9600 baud, an example polling cycle with eight slaves and five registers per slave is approximately 425 ms, assuming around 20 ms slave response time.

Dead-device timeouts are an important consideration. For an eight-slave configuration, a 150–200 ms timeout is the recommended starting point in the current design.

**Important:** These are configured design limits, not universal RS485 electrical limits.

## Wireless Performance Considerations

Example airtime at SF7, BW125 kHz and CR 4/5:

| Frame |     Size | Airtime |
| ----- | -------: | ------: |
| DATA  | 50 bytes | 97.5 ms |
| ACK   | 19 bytes | 51.5 ms |
| INFO  | 40 bytes |   82 ms |

At a two-second reporting interval, a 50-byte DATA frame and 19-byte ACK occupy approximately 149 ms of channel time per successful exchange.

At SF12, the example DATA airtime increases to approximately 2.3 seconds. Consequently, the reporting interval must account for the selected spreading factor and actual radio conditions.

### Multi-node reliability

The current gateway firmware has a configured capacity of 16 nodes. However, nodes currently use random-access transmission rather than coordinated transmission slots.

For a two-second reporting interval, the current design should be treated conservatively: one node per gateway is the reliable baseline in the supplied performance analysis.

Future time-slot scheduling is intended to improve multi-node reliability by assigning transmission offsets through gateway acknowledgements.

## MQTT Data Integration

The gateway forwards acquired Modbus values through MQTT using structured JSON telemetry.

Example payload structure:

```json
{
  "node": 1,
  "fcnt": 1024,
  "rssi": -85,
  "snr": 8.5,
  "sf": 7,
  "pwr": 14,
  "cycles": 100,
  "slave": 1,
  "fc": 3,
  "addr": 0,
  "count": 12,
  "st": "ok",
  "v": [230, 231, 229, 12, 15, 18]
}
```

Example MQTT topic:

```text
plant/gw01/node/1/data
```

The payload above illustrates the telemetry structure; actual register values and field names depend on the active firmware configuration.

## Industrial Applications

- Remote industrial instrumentation monitoring.
- Energy meter data acquisition.
- Pump and motor monitoring.
- Water treatment and utility monitoring.
- Remote sensor networks.
- Distributed industrial automation.
- Agricultural equipment monitoring.
- Remote Modbus device integration.

## Engineering Focus

The development focuses on:

- Efficient embedded firmware architecture.
- Reliable RS485 communication.
- Wireless packet design and airtime optimization.
- Secure data communication.
- Communication fault detection.
- Network scalability.
- MQTT-based industrial data integration.

## Development Roadmap

- [ ] Modbus RTU data acquisition architecture
- [ ] LoRa node-to-gateway communication
- [ ] Binary DATA, ACK and INFO frame design
- [ ] Communication status reporting
- [ ] MQTT telemetry integration
- [ ] AES-CCM authenticated frame design
- [ ] Adaptive Data Rate (ADR)
- [ ] Gateway-managed time-slot scheduling
- [ ] Expanded multi-node field validation
- [ ] Long-duration reliability and packet-loss testing
- [ ] Production-oriented hardware validation

## Project Information

**Project Category:** Industrial IoT / Embedded Systems

**Core Technologies:** Embedded C, LoRa, SX1262, Modbus RTU, RS485, MQTT, AES-CCM.

**Development:** Hardware and firmware development, wireless communication, protocol design and system integration.

---

*This repository documents the system architecture, engineering approach and development progress. Hardware specifications, firmware configuration and measured performance should be validated against the actual deployed prototype.*
