# Modbus exception codes & what causes them

When a slave rejects a request it replies with the function code **OR'd with 0x80** followed by a one-byte exception code. Reading them correctly points straight at the fault.

| Code | Meaning | Usual real-world cause |
|------|---------|------------------------|
| 01 | Illegal Function | device doesn't support that FC (e.g. FC16 on a read-only meter) |
| 02 | Illegal Data Address | wrong register/offset — the 40001→offset 0 trap |
| 03 | Illegal Data Value | value out of range for that register |
| 04 | Slave Device Failure | device-side fault while processing |
| 05 | Acknowledge | long operation accepted, still running |
| 06 | Slave Device Busy | retry after a short delay |
| 08 | Memory Parity Error | device memory issue |
| 0A | Gateway Path Unavailable | gateway misconfigured |
| 0B | Gateway Target Failed to Respond | serial device behind a TCP gateway didn't answer |

## Triage
- **02** — recompute the offset: 4xxxx → offset = address − 40001; 3xxxx → address − 30001. Also check you're using the right table (holding vs input).
- **01** — you're using a write code on a read-only table, or the device is a subset implementation.
- **0B** — the Ethernet side is fine but the serial device behind the gateway is unreachable (baud, wiring, slave ID).
- **No reply at all** (timeout) is *not* an exception — that's usually wrong baud/parity, wrong slave ID, or bus wiring.

Reproduce any of these safely, without risking a live device, using [Modbus Simulator](https://modbussimulator.com).
