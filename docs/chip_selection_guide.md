# RideProtect-RV — Chip Selection & Ordering Guide

**Document Version:** 1.0 | **SoC Version:** v2.5.0

---

## Part Number Nomenclature

As an FPGA-prototyped SoC with a planned ASIC migration, RideProtect-RV uses the following part-number scheme:

```
RP-RV-{SPEED}-{TEMP}-{PACKAGE}-{OPTION}
```

| Field | Values | Description |
|---|---|---|
| `RP` | — | RideProtect product family |
| `RV` | — | RISC-V CPU core |
| `{SPEED}` | `50` | Max system clock (MHz) on -1 speed grade |
| | `80` | Max system clock (MHz) on -2 speed grade |
| `{TEMP}` | `C` | Commercial (0°C to +85°C) |
| | `I` | Industrial (−40°C to +100°C) |
| | `E` | Extended (−40°C to +125°C) |
| `{PACKAGE}` | `A7-35` | Arty A7-35T (xc7a35ticsg324-1L) |
| | `A7-100` | Nexys A7-100T (xc7a100tcsg324-1) |
| `{OPTION}` | (blank) | Base configuration |
| | `-W` | Wireless (BLE helmet module enabled) |
| | `-S` | Security (immobilizer + encrypted boot) |

**Examples:**
- `RP-RV-50-C-A7-35` — 50 MHz, commercial temp, Arty A7-35T
- `RP-RV-80-I-A7-100-W` — 80 MHz, industrial temp, Nexys A7-100T with wireless support

---

## FPGA Target Selection

### For Evaluation & Development

| Board | FPGA | Speed Grade | LUTs | BRAM | DSP | Price Range | Best For |
|---|---|---|---|---|---|---|---|
| Arty A7-35T | xc7a35ticsg324-1L | -1L | 20,800 | 50 (18K) | 90 | ~$99 | Basic evaluation, cost-sensitive |
| Nexys A7-100T | xc7a100tcsg324-1 | -1 | 63,400 | 135 (18K) | 240 | ~$269 | Full peripheral set, expansion PMODs |

**Resource utilization (Arty A7-35T, v2.5.0):** ~11,500 LUTs (~55%), ~28 BRAM (~56%), ~12 DSP (~13%) — room for customer-specific accelerators.

### For Production (ASIC)

| Target | Technology | VDD Core | Speed | Status |
|---|---|---|---|---|
| SkyWater 130nm | SKY130A | 1.8V | TBD (post-layout) | Planned for v3.0 |

---

## Configuration Options

### Base Configuration (all variants)
- VexRiscv RV32IMC CPU
- 8 KB SRAM
- GPIO (32-bit), UART, SPI, I2C, Timer/PWM
- Interrupt Controller, Watchdog, RTC
- Fault Logger, Clock Output

### EV Option (recommended for 2W/3W applications)
Adds: Motor PWM, Advanced PWM, Hall Sensor, QEI, Resolver, BMS ADC, BMS I2C, BMS Balancer, BMS Temperature, ADC12, PMU

### Connectivity Option
Adds: CAN 2.0B, USB 2.0 ULPI with DMA

### Security Option
Adds: Immobilizer with challenge-response, watchdog with secure reset

### Accelerator Option
Adds: Matrix Multiply (4x4), DMA controller

---

## How to Order

### FPGA Bitstream
1. Clone the repository: `git clone https://github.com/aiquanticinsights-commits/RideProtect-SOC-Automobile.git`
2. Checkout v2.5: included in master
3. Run the build script: `cd fpga && vivado -source run_vivado.tcl` (Arty A7-35T) or modify TCL for Nexys A7-100T
4. Program the board via Vivado Hardware Manager

### Pre-built Bitstreams
Pre-compiled `.bit` files are available for:
- **Arty A7-35T** — `build/rideprotect_rv/rideprotect_rv.bit`
- **Nexys A7-100T** — build from source (see `fpga/run_vivado.tcl`, select `NEXYS_A7` target)

### ASIC / Commercial License
Contact [your-email / website] for:
- SkyWater 130nm GDS delivery
- Custom configuration and integration support
- Commercial licensing terms

---

## Revision History

| Rev | Date | Changes |
|---|---|---|
| 1.0 | 2026-07-16 | Initial v2.5.0 release |

---

*RideProtect-RV — Chip Selection & Ordering Guide v1.0*
