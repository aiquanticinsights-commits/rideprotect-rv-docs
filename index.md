---
layout: default
---

# RideProtect-RV v2.0.0

A full-featured RISC-V SoC for vehicle telematics and motor control, targeting the Digilent Arty A7-35T.

**Core:** VexRiscv RV32IMC · **Bus:** Wishbone B.4 32-bit shared · **Process:** 130nm-ready RTL

## Documentation

| Document | Description |
|----------|-------------|
| [Architecture](architecture.md) | SoC architecture, CPU, bus, peripherals, clock domains, safety mechanisms |
| [Memory Map & Register Reference](bus_map.md) | Complete address map, register maps, interrupt mapping, JTAG TAP |
| [Quick Start Guide](quickstart.md) | 5-minute guide: simulate, build, deploy |
| [Setup Guide](setup_guide.md) | Environment setup, toolchain installation |
| [Firmware Development Guide](firmware_guide.md) | API patterns, linker setup, IRQ handling, best practices |
| [Integration Guide](integration_guide.md) | Adding peripherals, bus protocol, FPGA deployment |
| [User Manual](RideProtect_RV_User_Manual.md) | Full architecture, pinout, peripheral programming, troubleshooting |
| [Project Report](RideProtect_RV_Project_Report.md) | Complete project documentation with metrics, verification results |
| [VexRiscv Migration Notes](vexriscv_migration.md) | CPU migration guide |
| [Matrix Multiply Integration](wb_matrix_mult_integration.md) | Matrix multiply accelerator integration guide |
| [Changelog v1.0 → v2.0](CHANGELOG_v1.0_to_v2.0.md) | Complete changelog between versions |

## Key Features

- **CPU:** VexRiscv RV32IMC, 5-stage pipeline, JTAG debug, Privileged ISA
- **Memory:** 32 KB SRAM with Hamming(39,32) SECDED ECC
- **Peripherals:** 16 Wishbone slaves — GPIO, UART, Timer/PWM, INTC, WDT, SPI, I2C, CAN 2.0B, DMA, Advanced PWM, RTC, Debug/JTAG, Matrix Mult, HS-LINK LVDS, USB 2.0 ULPI
- **AI/ML:** Hardware-accelerated 8×8 matrix multiply, NN inference engine, predictive maintenance
- **Safety:** ECC SRAM, watchdog timer, bus timeouts, PMP
- **Build:** Verilator simulation, Yosys synthesis, Vivado P&R
- **Tests:** 18/19 functional tests passing

## P&R Results (Vivado 2026.1)

| Metric | Value |
|--------|-------|
| Target | Arty A7-35T (xc7a35ticsg324-1L) |
| System clock | 50 MHz |
| Worst Negative Slack | 2.756 ns |
| Slice LUTs | 6,884 (33%) |
| Block RAM Tiles | 33 (66%) |
| DSP48E1 | 4 |

## Downloads

Pre-built bitstreams and firmware are available on the
[GitHub Releases](https://github.com/aiquanticinsights-commits/RideProtect-SOC-Automobile/releases)
page of the main repository.

## License

Documentation: **CC BY-NC 4.0** — free to share with attribution.
Source code (RTL, firmware, build scripts): **All Rights Reserved**.
