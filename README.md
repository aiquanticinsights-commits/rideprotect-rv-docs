# RideProtect-RV

**A RISC-V SoC for vehicle telematics and motor control.**

Targeting the Digilent Arty A7-35T FPGA board, RideProtect-RV integrates a VexRiscv RV32IM CPU with CAN 2.0B, PWM, SPI, I2C, UART, GPIO, watchdog timer, and interrupt controller on a shared Wishbone bus.

## Features

- VexRiscv RV32IM CPU with JTAG debug
- CAN 2.0B controller for vehicle bus communication
- 4-channel advanced PWM for motor control
- SPI (dual-mode) + I2C for sensor integration
- UART console at 115200 baud
- 32-bit GPIO for switches, LEDs, and general I/O
- Watchdog timer with configurable timeout
- 8 KB on-chip SRAM
- 50 MHz system clock, timing closure verified

## Documentation

| Document | Description |
|----------|-------------|
| [User Manual](User_Manual.md) | Architecture, pinout, peripheral programming, troubleshooting |
| [Quick Start](quickstart.md) | 5-minute guide: simulate, build, deploy |

## Downloads

Pre-built bitstreams and firmware are available from the
[Releases](https://github.com/aiquanticinsights-commits/RideProtect-SOC-Automobile/releases)
page of the main repository.

## P&R Results

| Metric | Value |
|--------|-------|
| Target | XC7A35TICSG324-1L |
| System clock | 50 MHz |
| Worst Negative Slack | 2.869 ns |
| LUT utilization | 31% (6,441 / 20,800) |
| Register utilization | 10% (4,229 / 41,600) |
| Block RAM | 2% (1 tile / 2×RAMB18E1) |

## License

Documentation in this repository is licensed under **CC BY-NC 4.0** —
you are free to share and adapt with attribution, for non-commercial use only.

The source code (RTL, firmware, scripts) is **All Rights Reserved**.
Contact aiquanticinsights@gmail.com for access or commercial licensing.

## Links

- [Main Repository](https://github.com/aiquanticinsights-commits/RideProtect-SOC-Automobile) (private)
- [GitHub Releases](https://github.com/aiquanticinsights-commits/RideProtect-SOC-Automobile/releases)
