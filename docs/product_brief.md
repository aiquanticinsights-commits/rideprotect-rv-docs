# RideProtect-RV — Product Brief

## Open-Source RISC-V Vehicle Telematics SoC

**Target Applications:** 2-Wheeler & 3-Wheeler Electric Vehicle Telematics, BMS, Motor Control
**Technology:** Xilinx Artix-7 FPGA Prototype → SkyWater 130nm ASIC (planned)
**Status:** v2.5.0 — Production-ready FPGA bitstream with formal verification

---

## Key Features

### Core Processor
- **CPU:** VexRiscv RV32IMC — 5-stage in-order pipeline with hardware multiply/divide and compressed instruction support
- **Performance:** ~50 MHz on Artix-7 (-1 speed grade), ~80 MHz on -2 grade
- **Debug:** JTAG (IEEE 1149.1) with OpenOCD/GDB support via wb_debug peripheral
- **Interrupts:** 32-input vectored interrupt controller (PLIC-style)

### Memory
- 8 KB on-chip SRAM (Wishbone slave, single-cycle access)
- Off-chip SPI flash boot (64 MB addressable via wb_spi)
- Firmware executes directly from SRAM

### Vehicle Communication Interfaces
- **CAN 2.0B** — 1 channel, up to 1 Mbps, with acceptance filtering
- **UART** — 1 channel, up to 3 Mbps, with fractional baud-rate generator
- **SPI** — Master/Slave, up to 25 MHz, 4 chip-select outputs
- **I2C** — Master/Slave, up to 400 kHz (Fast Mode)
- **USB 2.0 ULPI** — High-Speed (480 Mbps), 8 EP, with dedicatied DMA channel

### EV-Specific Peripherals
- **Motor PWM (wb_motor_pwm)** — 6-channel complementary PWM with dead-time insertion for 3-phase BLDC/PMSM
- **Advanced PWM (wb_pwm_adv)** — 4 independent PWM timers with complementary outputs and fault input
- **Hall Sensor (wb_hall)** — 3-input hall decoder with speed measurement
- **Quadrature Encoder (wb_qei)** — 4x resolution, index pulse capture
- **Resolver (wb_resolver)** — Sin/Cos tracking with programmable excitation frequency
- **BMS ADC (wb_bms_adc)** — 8-channel, 12-bit SAR, up to 1 Msps
- **BMS I2C (wb_bms_i2c)** — Dedicated I2C for battery management ICs
- **BMS Balancer (wb_bms_bal)** — Passive cell balancing sequencer
- **BMS Temperature (wb_bms_temp)** — NTC thermistor interface, up to 4 channels
- **ADC12 (wb_adc12)** — General-purpose 12-bit SAR with sequencer
- **PMU (wb_pmu)** — Power management unit with Sleep/Wake/DeepSleep modes

### General-Purpose Peripherals
- **GPIO** — 32-bit programmable I/O with interrupt-on-change
- **Timer/PWM** — 1 channel, 16-bit, with capture/compare
- **Watchdog (wb_watchdog)** — 32-bit programmable, windowed mode
- **RTC (wb_rtc)** — 32-bit counter with alarm, runs from 32.768 kHz
- **Interrupt Controller** — 32 input lines, 4 priority levels
- **Debug Module (wb_debug)** — 4 hardware breakpoints, 32-word mailbox, JTAG↔WB bridge
- **Matrix Multiply (wb_matrix_mult)** — 4x4 fixed-point accelerator

### Fault Management
- **Fault Logger (wb_fault)** — 32-entry fault FIFO with timestamp
- **Clocked Output (wb_clkout)** — Programmable frequency output for diagnostic validation
- **Watchdog** — Independent windowed watchdog with reset output
- **Immobilizer (wb_immob)** — Challenge-response authentication engine

---

## System Architecture

**Bus Topology:** 14-slot Wishbone B4 shared bus with a single master (VexRiscv CPU via wb_cpu_wrapper). Fixed-priority arbitration where CPU always wins. DMA controller can request bus mastership.

**Clock Domains:**
- `clk_i` (sys_clk): 50 MHz main system clock
- `ulpi_clk`: 60 MHz USB ULPI clock
- `dbg_clk`: JTAG TCK (variable, up to 25 MHz)

**Clock Domain Crossings:** All async inputs (IRQ, debug, USB) use 2-flop synchronizers. Formal CDC properties verified in simulation.

**Memory Map Summary:**
| Address Range | Peripheral | Size |
|---|---|---|
| 0x0000_0000 – 0x0000_1FFF | SRAM (boot + firmware) | 8 KB |
| 0x1000_0000 – 0x1000_FFFF | GPIO | 64 KB |
| 0x2000_0000 – 0x2000_FFFF | UART | 64 KB |
| 0x3000_0000 – 0x3000_FFFF | Timer/PWM | 64 KB |
| 0x4000_0000 – 0x4000_FFFF | Interrupt Controller | 64 KB |
| 0x5000_0000 – 0x5000_FFFF | Watchdog | 64 KB |
| 0x6000_0000 – 0x6000_FFFF | SPI | 64 KB |
| 0x7000_0000 – 0x7000_FFFF | I2C | 64 KB |
| ... | (14 slaves total) | |

Full bus map: `docs/bus_map.md`

---

## Development Tools & Flow

| Tool | Version | Purpose |
|---|---|---|
| Yosys | 0.48+ | RTL synthesis & formal verification |
| nextpnr | 0.7+ | FPGA place & route |
| Project X-Ray | db latest | Artix-7 bitstream generation |
| Vivado | 2024.2 | Alternate flow (Arty A7-35T + Nexys A7-100T) |
| cocotb | 1.8+ | Python-based simulation |
| SymbiYosys (sby) | 0.32+ | Formal property verification |
| RISC-V GCC | 12.2+ | Firmware compilation |
| OpenOCD | 0.12+ | JTAG debug bridge |

---

## Comparison: RideProtect-RV vs. ARM-Based Alternatives

| Parameter | RideProtect-RV | ARM Cortex-M4 (typical) | ARM Cortex-M7 (typical) |
|---|---|---|---|
| ISA | RISC-V RV32IMC | ARMv7E-M | ARMv7E-M |
| License | Open-source (BSD / Apache 2.0) | Proprietary (ARM license) | Proprietary (ARM license) |
| FPGA Cost | Free (open-source flow) | N/A (commercial IP only) | N/A (commercial IP only) |
| ASIC Path | SkyWater 130nm (OpenLane) | Foundry-dependent | Foundry-dependent |
| Peripheral Set | EV-optimized (BMS, motor PWM, resolver) | Generic MCU | Generic MCU |
| Formal Verification | Included (25+ SVA assertions) | Vendor-dependent | Vendor-dependent |
| Toolchain | GCC + OpenOCD | ARM Keil / IAR / GCC | ARM Keil / IAR / GCC |

---

## Ordering & Licensing

RideProtect-RV is released under a dual license:
- **Open-source (BSD 2-Clause):** RTL source code, formal verification suite, simulation testbench, firmware examples
- **Commercial License:** For proprietary ASIC integration or closed-source derivative works

For FPGA evaluation, use:
- **Arty A7-35T** (xc7a35ticsg324-1L) — bitstream included in v2.5.0
- **Nexys A7-100T** (xc7a100tcsg324-1) — constraints provided, build via TCL script

Contact: [your-email / website]

---

*RideProtect-RV v2.5.0 — Document version 1.0*
