# Modbus function codes — complete reference

Modbus exposes four data tables, each with its own address space and its own read/write function codes. Knowing which code touches which table saves hours of "Illegal Function" debugging.

## The four data tables
| Table | Size | Access | Classic address | Read | Write |
|-------|------|--------|-----------------|------|-------|
| Coils | 1 bit | R/W | 00001–09999 | 01 | 05 / 15 |
| Discrete Inputs | 1 bit | R | 10001–19999 | 02 | — |
| Input Registers | 16-bit | R | 30001–39999 | 04 | — |
| Holding Registers | 16-bit | R/W | 40001–49999 | 03 | 06 / 16 |

## Function codes
- **01** Read Coils · **02** Read Discrete Inputs
- **03** Read Holding Registers · **04** Read Input Registers
- **05** Write Single Coil · **06** Write Single Register
- **15 (0x0F)** Write Multiple Coils · **16 (0x10)** Write Multiple Registers
- 07, 08, 11, 12, 17 — diagnostics/status, rarely needed in the field.

## FC06 vs FC16 — a real gotcha
Many analog-output / DAC devices update their physical output **only on FC06 (write single register)**. A FC16 "write multiple" may just store the value in the register without moving the output. If you write a value and nothing physically changes, switch to FC06 and read the register back to confirm.

## Addressing off-by-one
The classic address `40001` is 1-based and includes the table digit; on the wire the **protocol address is 0-based**, so `40001` = offset `0`. Forgetting this is the #1 cause of exception `02` (Illegal Data Address).

## Verify without hardware
Point a master at a simulator, exercise each code, and watch the raw frames. I use [Modbus Simulator](https://modbussimulator.com) — free Windows master + slave for RTU and TCP, with per-register values and data-type control.
