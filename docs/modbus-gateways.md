# Using Modbus gateways (serial ↔ Ethernet)

A gateway bridges Modbus RTU serial devices onto a Modbus TCP network so SCADA can reach them over Ethernet. Great for retrofits — but it adds a layer that hides serial issues if you're not careful.

## Two common modes
- **Modbus TCP ↔ RTU gateway**: the master speaks TCP to the gateway's IP; the gateway relays to serial devices by **unit/slave ID**. This is the clean, addressable option.
- **Modbus RTU-over-TCP ("encapsulated")**: raw RTU frames (with CRC) tunneled inside TCP. Simpler but less standard; the client must speak the same variant.

## Addressing
With a TCP↔RTU gateway you keep each serial device's **slave ID** and target it through the one gateway IP: e.g. read holding registers from unit 3 at `192.168.1.50:502`. The gateway routes by unit ID onto the RS-485 bus.

## Gotchas
- **Exception 0B** (Gateway Target Failed to Respond) means the Ethernet side is fine but the serial device didn't answer — check baud, wiring, and that the slave ID exists.
- Don't over-poll: the serial side is still the bottleneck. Ten fast TCP requests still queue onto one slow RS-485 bus.
- Match the gateway's serial settings (baud/parity) to the devices exactly.

## Test the TCP side independently
Before blaming the serial bus, confirm your master ↔ gateway TCP path works by pointing it at a plain Modbus TCP endpoint like [Modbus Simulator](https://modbussimulator.com); if that's clean, the problem is downstream on the serial side.
