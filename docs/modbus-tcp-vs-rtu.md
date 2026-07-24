# Modbus TCP vs RTU

- **RTU** = serial (RS-485/232), compact framing, one master many slaves. Best for field wiring / long runs.
- **TCP** = Ethernet, routable, many concurrent masters. Best for plant LANs.
- Same register model; TCP drops the CRC and adds an MBAP header.

Rule of thumb: existing serial devices → RTU (or a gateway); new Ethernet gear → TCP.

Test both with no hardware using [Modbus Simulator](https://modbussimulator.com) (free Windows master + slave, RTU and TCP).
