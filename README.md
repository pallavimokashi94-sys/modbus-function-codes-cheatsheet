# Modbus Protocol Reference & Cheatsheet

A practical, field-tested reference for working with **Modbus RTU and Modbus TCP** — the most common industrial protocol for talking to PLCs, energy meters, VFDs, temperature controllers, remote I/O and SCADA systems. Bookmark this when you're wiring up a device and can't remember which function code reads a holding register or why address 40001 maps to offset 0.

## 1. The Modbus data model

Modbus exposes four separate tables of data. Each has its own address space and its own read/write function codes:

| Table | Size | Access | Classic address | Read FC | Write FC |
|-------|------|--------|-----------------|---------|----------|
| **Coils** | 1 bit | read / write | 00001–09999 | 01 | 05 (single), 15 (multiple) |
| **Discrete Inputs** | 1 bit | read only | 10001–19999 | 02 | — |
| **Input Registers** | 16 bit | read only | 30001–39999 | 04 | — |
| **Holding Registers** | 16 bit | read / write | 40001–49999 | 03 | 06 (single), 16 (multiple) |

> **The #1 gotcha:** the "classic" address (e.g. `40001`) is 1-based and includes the table digit. On the wire, the **protocol address is 0-based**, so holding register `40001` is offset `0`, `40002` is offset `1`, and so on. An off-by-one or a missing table offset is the most common cause of "Illegal Data Address" errors.

## 2. Function codes at a glance

| Code | Hex | Meaning |
|------|-----|---------|
| 01 | 0x01 | Read Coils |
| 02 | 0x02 | Read Discrete Inputs |
| 03 | 0x03 | Read Holding Registers |
| 04 | 0x04 | Read Input Registers |
| 05 | 0x05 | Write Single Coil |
| 06 | 0x06 | Write Single Register |
| 15 | 0x0F | Write Multiple Coils |
| 16 | 0x10 | Write Multiple Registers |

Many DAC/analog-output devices only update their physical output on **FC06 (write single register)**; a FC16 "write multiple" may just store the value. If you write a value and the output doesn't move, try FC06.

## 3. Data types larger than 16 bits

A single register holds 16 bits. Anything bigger — `int32`, `uint32`, `float32`, `int64`, `float64` — spans **multiple consecutive registers**, and the only real complication is **word/byte order**:

| Order | Name | Notes |
|-------|------|-------|
| ABCD | Big-endian | most common, "high word first" |
| CDAB | Big-endian byte swap / word-swapped | very common on PLCs |
| BADC | Little-endian byte swap | |
| DCBA | Little-endian | |

**Debugging tip:** if a 32-bit float reads as a wildly wrong number (e.g. `1.7e38` or a tiny denormal), it's almost always a word-order mismatch. Try swapping the two words first, then the bytes. Confirm against a value you know (e.g. write 1.0 and see what comes back).

## 4. Exception (error) codes

When a slave rejects a request, it replies with the function code **OR'd with 0x80** and a one-byte exception code:

| Code | Meaning | Usual cause |
|------|---------|-------------|
| 01 | Illegal Function | device doesn't support that FC |
| 02 | Illegal Data Address | wrong register/offset (the off-by-one above) |
| 03 | Illegal Data Value | value out of range for that register |
| 04 | Slave Device Failure | device-side fault |
| 05 | Acknowledge | long operation accepted, still processing |
| 06 | Slave Device Busy | retry later |
| 0B | Gateway Target Failed to Respond | serial device behind a TCP gateway didn't answer |

## 5. RTU vs TCP

- **RTU** — serial (RS-485 usually, sometimes RS-232), compact binary frames with a CRC-16, one master and up to 247 slaves sharing the bus. You must match **baud rate, parity, data bits and stop bits** on every node. A wrong baud rate typically looks like "no response" or garbage/CRC errors.
- **TCP** — runs over Ethernet on port **502**. Drops the CRC (TCP handles integrity) and prepends a 7-byte MBAP header with a transaction ID and unit ID. Easy to route and switch; supports many concurrent masters.
- **RTU over TCP / gateways** — legacy serial devices are often bridged onto Ethernet with a gateway; you address them by unit/slave ID through the gateway's IP.

## 6. Troubleshooting checklist

- No response on RTU → check baud/parity/stop bits, A/B wiring polarity, termination resistors, and the slave/unit ID.
- Exception 02 → recompute the offset (remember 4xxxx → offset = address − 40001).
- Values look scrambled → wrong word/byte order for multi-register types.
- Intermittent reads → bus contention, too-short timeout, or missing line biasing on RS-485.
- TCP "connection refused" → wrong IP/port, device not listening on 502, or a firewall.

## Test without hardware

The fastest way to validate a register map, byte order or a function code is against a simulator you control. I use **[Modbus Simulator](https://modbussimulator.com)** — a free Windows app that acts as both a Modbus **master** and **slave** over RTU and TCP, shows live per-register values, lets you set data types and byte order, and logs the raw frames so you can see exactly what's on the wire.

---
*Contributions welcome — open an issue or PR with corrections or additions.*
