# RideProtect-RV

A full-featured RISC-V SoC for vehicle telematics and motor control, targeting the Digilent Arty A7-35T.

**Core**: VexRiscv (RV32IM) · **Bus**: Wishbone B.4 32-bit shared · **Process**: 130nm-ready RTL

## Features

- **CPU**: VexRiscv RV32IM, ~1.4k LCs, 5-stage pipeline, JTAG debug via BuG
- **Memory**: 8 KB SRAM (BRAM), firmware loaded via `$readmemh`
- **Peripherals**: GPIO (32-bit), UART (115200 baud), Timer/PWM (advanced 4-ch), Watchdog, SPI (dual-mode), I2C, CAN 2.0B
- **Clock**: 100 MHz input → 50 MHz system clock (toggle + BUFG)
- **Build**: Verilator sim + Vivado 2026.1 P&R

## Block Diagram

```
┌─────────────────────────────────────────────────────┐
│  VexRiscv RV32IM          Wishbone Shared Bus       │
│  ┌──────────┐    ┌──────────────────────────────┐   │
│  │ CPU core │────│  Arbiter + Decoder + Timeout  │   │
│  │ JTAG TAP │    └──┬──┬──┬──┬──┬──┬──┬──┬──────┘   │
│  └──────────┘       │  │  │  │  │  │  │  │          │
│              ┌──────┘  │  │  │  │  │  │  └──────┐   │
│              ▼         ▼  ▼  ▼  ▼  ▼  ▼         ▼   │
│  ┌──────┐ ┌──────┐ ┌──┐ ┌─┐ ┌─┐ ┌─┐ ┌─┐ ┌──────┐  │
│  │ SRAM │ │Debug │ │GP│ │UA│ │Ti│ │WD│ │SP│ │ CAN  │  │
│  │ 8 KB │ │ JTAG │ │IO│ │RT│ │mer│ │T │ │I │ │2.0B  │  │
│  └──────┘ └──────┘ └──┘ └─┘ └─┘ └─┘ └─┘ └──────┘  │
└─────────────────────────────────────────────────────┘
```

## Hardware Required

| Item | Purpose |
|------|---------|
| Digilent Arty A7-35T (xc7a35ticsg324-1L) | Target FPGA board |
| Micro-USB cable | Power + UART + programming |
| USB-UART (built-in FTDI) | 115200 8-N-1 console |

## Toolchain Setup

| Tool | Version | Purpose | Install |
|------|---------|---------|---------|
| riscv64-linux-gnu-gcc | ≥ 15.x | RISC-V cross-compiler | `apt install gcc-riscv64-linux-gnu` |
| Verilator | ≥ 5.032 | RTL simulation | `apt install verilator` |
| Vivado HL WebPACK | 2026.1 | FPGA P&R + bitstream | [AMD/Xilinx](https://www.xilinx.com/support/download.html) |
| Pandoc | 3.4 | .md → .docx conversion | `tools/pandoc/pandoc-3.4/pandoc.exe` |

## Quick Start

```bash
# Build firmware
make firmware

# Simulate (Verilator)
make sim

# Lint all RTL
make lint

# FPGA P&R → bitstream (requires Vivado)
make pnr-arty

# Program via Vivado Hardware Manager
# Connect USB, open Hardware Manager, select build/soc_top.bit
```

## Project Structure

```
rideprotect-rv/
├── rtl/              # RTL source (Verilog)
│   ├── soc_top.v     # Top-level SoC wrapper
│   ├── wb_*.v        # Wishbone peripherals
│   └── VexRiscv.v    # CPU core
├── sim/              # Verilator testbench
│   ├── sim_main.cpp  # C++ simulation harness
│   └── isa_test.S    # RISC-V ISA tests
├── sw/               # RISC-V firmware
│   ├── boot.S        # Startup + vector table
│   ├── link.ld       # Linker script
│   └── test_prog.c   # Test firmware (15 tests)
├── fpga/             # FPGA build files
│   ├── arty_a7.xdc   # Arty A7-35T constraints
│   ├── run_vivado.tcl# Batch P&R script
│   ├── run_pnr.ps1   # PowerShell runner
│   └── create_project.tcl  # Vivado GUI project
├── docs/             # Documentation
│   ├── RideProtect_RV_Project_Report.md
│   ├── RideProtect_RV_User_Manual.md
│   ├── quickstart.md
│   ├── setup_guide.md
│   ├── firmware_guide.md
│   └── integration_guide.md
├── .github/workflows/ # CI (lint + sim + syn)
├── build/            # Build artifacts
│   ├── firmware.hex  # Firmware memory image
│   ├── soc_top.bit   # Bitstream (2.1 MB)
│   ├── timing.rpt    # Timing report
│   └── utilization.rpt  # Utilization report
└── Makefile          # Top-level build targets
```

## P&R Results (Vivado 2026.1)

| Metric | Value |
|--------|-------|
| Target | Arty A7-35T (xc7a35ticsg324-1L) |
| System clock | 50 MHz |
| Worst Negative Slack | 2.869 ns |
| Worst Hold Slack | 0.032 ns |
| Slice LUTs | 6,441 (31%) |
| Slice Registers | 4,229 (10%) |
| Block RAM | 1 tile / 2×RAMB18E1 |
| DSP48E1 | 4 |
| BUFG | 3 |

## Memory Map

| Address | Peripheral | Size |
|---------|-----------|------|
| 0x0000_0000 | SRAM (code + data) | 8 KB |
| 0x2000_0000 | GPIO | 4 B |
| 0x2000_1000 | UART | 4 B |
| 0x2000_2000 | Timer/PWM | 4 B |
| 0x2000_3000 | Interrupt Controller | 4 B |
| 0x2000_4000 | Watchdog | 4 B |
| 0x2000_5000 | SPI | 4 B |
| 0x2000_6000 | I2C | 4 B |
| 0x2000_7000 | CAN 2.0B | 4 B |

## License

Source code (RTL, firmware, scripts, build files): **All Rights Reserved**.  
Documentation (User Manual, Quick Start): **CC BY-NC 4.0**.

See [LICENSE](LICENSE) for full terms. Contact aiquanticinsights@gmail.com for commercial licensing.

## Public Documentation

User Manual and Quick Start are publicly available at:
https://aiquanticinsights-commits.github.io/rideprotect-rv-docs/
