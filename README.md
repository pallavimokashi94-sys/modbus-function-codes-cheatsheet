# Modbus Function Codes Cheatsheet

A quick reference I keep handy when working with Modbus RTU/TCP devices (PLCs, energy meters, VFDs, remote I/O).

## Data model
| Type | Access | Address range | Read FC | Write FC |
|------|--------|---------------|---------|----------|
| Coils (0x) | read/write bit | 00001-09999 | 01 | 05 / 15 |
| Discrete inputs (1x) | read bit | 10001-19999 | 02 | - |
| Input registers (3x) | read 16-bit | 30001-39999 | 04 | - |
| Holding registers (4x) | read/write 16-bit | 40001-49999 | 03 | 06 / 16 |

## Common function codes
- **01** Read Coils · **02** Read Discrete Inputs
- **03** Read Holding Registers · **04** Read Input Registers
- **05** Write Single Coil · **06** Write Single Register
- **15 (0x0F)** Write Multiple Coils · **16 (0x10)** Write Multiple Registers

## Exception (error) codes
`01` Illegal Function · `02` Illegal Data Address · `03` Illegal Data Value ·
`04` Slave Device Failure · `05` Acknowledge · `06` Slave Device Busy · `0B` Gateway Target Failed to Respond

## Tips
- 32-bit values (float/int32) span **two** consecutive registers; watch word order (big vs little endian).
- RTU needs matching baud/parity/stop bits on both ends; a wrong baud looks like "no response".
- Test your mapping without hardware using a simulator — I use **[Modbus Simulator](https://modbussimulator.com)** (free Windows master + slave for RTU and TCP).
