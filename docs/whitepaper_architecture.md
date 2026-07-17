# RideProtect-RV Architecture Whitepaper

## An Open-Source RISC-V SoC for 2W/3W Electric Vehicle Telematics

**Document Version:** 1.0 | **SoC Version:** v2.5.0

---

## 1. Introduction

Modern two- and three-wheeler electric vehicles increasingly require sophisticated electronic control — battery management, motor commutation, telematics, and connectivity — all at a cost point that makes traditional automotive MCUs prohibitive. RideProtect-RV addresses this gap with an open-source, RISC-V-based system-on-chip designed explicitly for the 2W/3W EV segment.

The architecture is built around three principles:
1. **Determinism** — Fixed-priority bus arbitration and single-clock-domain core logic eliminate timing surprises
2. **Safety** — Dedicated fault-logging peripheral, windowed watchdog, and formal verification of critical paths
3. **Extensibility** — Wishbone B4 shared bus allows straightforward peripheral add/remove without revalidating the full system

---

## 2. Architectural Innovations

### 2.1 Wishbone Shared Bus with Fixed-Priority Arbitration

Unlike traditional ARM-based MCUs that use AHB/AXI crossbars with round-robin or TDMA arbitration, RideProtect-RV uses a **Wishbone B4 shared bus (`wb_shared_bus.v`)** with a single bus master — the VexRiscv CPU.

**Key design decisions:**

- **Single master, multiple slaves:** The CPU (via `wb_cpu_wrapper`) is the sole bus master except when the DMA controller explicitly requests mastership. This eliminates the need for complex coherency protocols.
- **Fixed-priority arbitration:** When both CPU and DMA request the bus, the CPU always wins. This guarantees interrupt latency is bounded by a single pending DMA transfer — typically 1–2 bus cycles.
- **Combinatorial decode + registered timeout:** Address decoding is purely combinatorial (select lines assert in the same cycle as `stb_i`). A 256-cycle timeout counter on each slave port prevents bus hangs from unresponsive peripherals.

**Why not AHB/AXI?** The Wishbone B4 specification is 11 pages (vs. hundreds for AMBA). The entire arbiter + decoder + timeout logic fits in ~250 lines of Verilog and is fully formal-verified with SymbiYosys.

### 2.2 Peripherals Optimized for EV Control

The peripheral set is not a generic MCU complement — it is curated for the EV use case:

| Domain | Peripheral | Purpose |
|---|---|---|
| Traction | `wb_motor_pwm` | 6-channel complementary PWM with programmable dead-time (0–10 µs) for 3-phase BLDC/PMSM |
| Traction | `wb_hall` | Hall-sensor decoder with 16-bit speed counter and direction detection |
| Traction | `wb_qei` | Quadrature encoder with 4x pulse multiplication and index reset |
| Traction | `wb_resolver` | Sin/Cos resolver tracking loop with programmable excitation (5–20 kHz) |
| Battery | `wb_bms_adc` | 8-channel, 12-bit SAR ADC sequencer with per-channel gain and sample-time |
| Battery | `wb_bms_i2c` | Dedicated I2C master for fuel-gauge and balancer ICs (separate from general-purpose I2C) |
| Battery | `wb_bms_bal` | Passive-balancing sequencer that cycles through N cells with programmable duty |
| Battery | `wb_bms_temp` | NTC thermistor front-end with β-equation compensation in firmware |
| Safety | `wb_fault` | 32-entry fault FIFO with microsecond-resolution timestamp |
| Safety | `wb_immob` | Challenge-response authentication with 128-bit secret key |
| Connectivity | `wb_usb` | USB 2.0 ULPI with 8 endpoints and dedicated DMA |

### 2.3 Clock Domain Crossing Strategy

The SoC has three clock domains:
- **sys_clk** (clk_i, nominally 50 MHz) — all Wishbone peripherals, CPU, SRAM
- **ulpi_clk** (60 MHz) — USB ULPI PHY interface
- **dbg_clk** (jtag_tck, variable) — JTAG debug interface

**Crossing methodology:**
- All CDC paths use **two-flop synchronizers** (no multi-cycle paths, no FIFO-based crossings for control signals)
- **Gray-code encoding** for FIFO pointers crossing between sys_clk and ulpi_clk in the USB peripheral
- **Pulse synchronizers** for one-shot events (interrupts, debug mailbox writes)
- Every synchronizer has formal properties coded in SVA: single-bit-toggle guarantee, pipeline delay, monotonicity of FIFO pointers

Two design bugs were discovered and fixed during formal CDC verification of `wb_usb.v`:
1. A registered flop with no reset was incorrectly driving the gray-code pointer
2. FIFO full/empty flags were comparing binary pointers against gray-coded pointers

This demonstrates the value of systematic CDC formal verification even in a "simple" single-master design.

### 2.4 Formal Verification Methodology

Rather than relying solely on simulation, RideProtect-RV includes a formal verification suite targeting SymbiYosys + ABC (PDR engine):

| Module | Properties | Status | Engine |
|---|---|---|---|
| `wb_dma` | 5 — transfer correctness, address increment, burst termination | PASS | ABC PDR |
| `wb_gpio` | 3 — write/read-back, interrupt-on-change, bit masking | PASS | ABC PDR |
| `wb_interrupt_controller` | 4 — pending/clear, priority encoding, vectored dispatch | PASS | ABC PDR |
| `wb_motor_pwm` | 4 — duty cycle, dead-time insertion, complementary output | PASS | ABC PDR |
| `wb_pwm_adv` | 3 — independent period, phase alignment, fault override | PASS | ABC PDR |
| `wb_shared_bus` | 3 — arbitration fairness, timeout recovery, decode accuracy | PASS | ABC PDR |
| `wb_timer` | 3 — reload, capture, interrupt generation | PASS | ABC PDR |
| `wb_usb` | 5 — CDC properties (gray-code, monotonicity, pipeline) | COVERAGE ONLY | smtbmc+Z3 rec. |
| `wb_debug` | 4 — CDC properties (pulse handshake, pipeline, read-back) | COVERAGE ONLY | smtbmc+Z3 rec. |

Seven of nine modules pass under ABC PDR. The two CDC modules fail due to ABC's limited dual-clock AIG modeling and are recommended for SMT-based engines or commercial CDC tools.

---

## 3. Comparison to Incumbent Solutions

| Aspect | Traditional ARM Cortex-M4/M7 | RideProtect-RV |
|---|---|---|
| ISA License | Proprietary, high NRE | RISC-V, royalty-free |
| Toolchain | Vendor-locked (Keil/IAR) | Open-source GCC + OpenOCD |
| Peripheral Fit | Generic MCU + external BMS | Integrated EV-specific blocks |
| Verification | Vendor-provided DV | User-verifiable formal properties |
| Customization | Fixed silicon | Configurable RTL (FPGA or ASIC) |
| Supply Chain | Single-source foundry | Multi-source (any fab with 130nm+) |

---

## 4. Path to Production: FPGA → ASIC

The v2.5.0 release targets Xilinx Artix-7 FPGA as a **functional prototype**. The path to ASIC production is designed to minimize risk:

1. **v2.x (current)** — FPGA prototype on Arty A7-35T / Nexys A7-100T. All peripherals verified in simulation or on FPGA. Formal verification suite serves as the "golden reference" for RTL correctness.
2. **v3.0 (planned)** — Synthesis for SkyWater 130nm (SKY130A) via OpenLane. Target: ~80 MHz @ 1.8V, ~2–5 mm² die area.
3. **v3.1+ (planned)** — Customer-specific configuration: subset peripherals, custom accelerators, security hardening.

The open-source flow (Yosys + nextpnr or OpenLane) means no vendor lock-in at any stage.

---

## 5. Conclusion

RideProtect-RV demonstrates that a production-quality vehicle telematics SoC can be built entirely with open-source tools, targeting an underserved market segment (2W/3W EVs) with a purpose-built peripheral set. The inclusion of formal verification from day one raises the confidence level beyond what conventional simulation-only flows provide, and the FPGA-first approach enables real-world deployment before ASIC commitment.

---

*RideProtect-RV Architecture Whitepaper v1.0 — © 2026*
