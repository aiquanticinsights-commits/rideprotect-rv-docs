# RideProtect-RV — Toolchain & Debugging Guide

**SoC Version:** v2.5.0 | **Document Version:** 1.0 | **Target Audience:** Embedded Software Developers

---

## 1. Toolchain Overview

| Tool | Purpose | Version Required | Installation |
|---|---|---|---|
| RISC-V GCC | Firmware compilation | 12.2.0+ | see §2.1 |
| GNU Make | Build system | 4.0+ | OS package |
| OpenOCD | JTAG debug bridge | 0.12.0+ | see §2.2 |
| GDB (riscv32) | Source-level debugger | 12.1+ | bundled with GCC |
| cocotb + Verilator | Simulation test | cocotb 1.8+ | pip install |
| Yosys | RTL synthesis / formal | 0.48+ | see §2.3 |
| SymbiYosys (sby) | Formal verification | 0.32+ | pip install |
| Python | Test scripts & tools | 3.10+ | OS package |

---

## 2. Installation

### 2.1 RISC-V GCC Toolchain

**Option A — Pre-built binaries (recommended):**
```bash
# Download from https://github.com/riscv-collab/riscv-gnu-toolchain/releases
wget https://github.com/riscv-collab/riscv-gnu-toolchain/releases/download/2024.12.20/riscv32-unknown-elf.gcc-14.2.0.tar.gz
tar xzf riscv32-unknown-elf.gcc-14.2.0.tar.gz
export PATH=$PWD/riscv32-unknown-elf/bin:$PATH
```

**Option B — Build from source:**
```bash
git clone https://github.com/riscv-collab/riscv-gnu-toolchain
cd riscv-gnu-toolchain
./configure --prefix=/opt/riscv --with-arch=rv32imc --with-abi=ilp32
make -j$(nproc)
export PATH=/opt/riscv/bin:$PATH
```

### 2.2 OpenOCD

```bash
# Ubuntu / Debian / WSL
sudo apt install openocd

# From source (for latest RISC-V support)
git clone https://github.com/riscv-collab/riscv-openocd
cd riscv-openocd
./configure --prefix=/usr/local --enable-ftdi --enable-jtag
make -j$(nproc)
sudo make install
```

### 2.3 Yosys + nextpnr

```bash
# Ubuntu / Debian / WSL
sudo apt install yosys nextpnr-xilinx

# Or build from source (for latest formal features)
git clone https://github.com/YosysHQ/yosys
cd yosys && make -j$(nproc) && sudo make install
```

---

## 3. Building Firmware

### 3.1 Quick Build

```bash
cd sw/boot
make clean
make
```

This produces:
- `firmware.elf` — ELF binary
- `firmware.hex` — Intel HEX (for SPI flash / simulation)
- `firmware.lst` — Disassembly listing

### 3.2 Makefile Targets

| Target | Description |
|---|---|
| `make` | Build firmware.hex |
| `make flash` | Program via OpenOCD (requires JTAG connected) |
| `make monitor` | Open serial console (9600 baud, /dev/ttyUSB0) |
| `make clean` | Remove build artifacts |
| `make debug` | Build with -O0 -g for debugging |

### 3.3 Compiler Flags

```
CFLAGS = -march=rv32imc -mabi=ilp32 -Os -ffreestanding -nostdlib
LDFLAGS = -T linker.ld -nostartfiles -lgcc
```

- `-Os` — optimize for size (8 KB SRAM budget)
- `-ffreestanding` — no libc dependency
- `-nostdlib` / `-nostartfiles` — bare-metal startup

---

## 4. FPGA Programming

### 4.1 Via Vivado Hardware Manager

1. Open Vivado → Open Hardware Manager → Auto Connect
2. Add Configuration Memory Device → select the SPI flash (N25Q128 or equivalent)
3. Select `firmware.hex` as the data file (or combine with bitstream)
4. Program

### 4.2 Via OpenOCD (SPI flash)

```bash
openocd -f interface/ftdi/jtag.cfg -f board/arty.cfg \
    -c "init; pld load 0 top.bit; exit"
```

### 4.3 Via TCL Build Script

```bash
cd fpga
vivado -source run_vivado.tcl
```

Produces `top.bit` in `fpga/`.

---

## 5. JTAG Debug

### 5.1 Hardware Setup

```
┌──────────────┐     JTAG cable     ┌──────────────────┐
│  Host PC     │◄──────────────────►│  RideProtect-RV   │
│  (OpenOCD    │   (6-pin header)   │  (FPGA board)     │
│   + GDB)     │                    │                   │
└──────────────┘                    └──────────────────┘
```

**Pinout (J9 connector on Arty):**
| Pin | Signal | Direction |
|---|---|---|
| 1 | TCK | Output from probe |
| 2 | GND | — |
| 3 | TMS | Output from probe |
| 4 | VREF (3.3V) | Input to probe |
| 5 | TDI | Output from probe |
| 6 | — | (nTRST, optional) |
| 7 | — | (not connected) |
| 8 | — | (not connected) |
| 9 | TDO | Input to probe |
| 10 | — | (not connected) |

### 5.2 OpenOCD Configuration

Create `openocd.cfg`:
```
adapter driver ftdi
adapter speed 5000
transport select jtag

set CHIPNAME riscv
jtag newtap $CHIPNAME cpu -irlen 5 -expected-id 0x10001FFF

target create $CHIPNAME.cpu riscv -chain-position $CHIPNAME.cpu
$CHIPNAME.cpu configure -work-area-phys 0x00000000 -work-area-size 0x2000

init
halt
```

### 5.3 Debug Session

```bash
# Terminal 1 — Start OpenOCD
openocd -f openocd.cfg

# Terminal 2 — Connect GDB
riscv32-unknown-elf-gdb sw/boot/firmware.elf
(gdb) target remote localhost:3333
(gdb) monitor reset halt
(gdb) load
(gdb) continue
```

### 5.4 Debug Commands (GDB)

```gdb
# Halt execution
monitor halt

# Reset and halt
monitor reset halt

# Load firmware
load

# Set breakpoint
break main
break *0x00000010   # hardware breakpoint via wb_debug

# Examine memory
x/32wx 0x10000000   # dump GPIO registers

# Modify memory
set {int}0x10000000 = 0x01    # set GPIO output

# Continue execution
continue

# Step
stepi
```

---

## 6. Simulation Debug

### 6.1 Running Simulation Tests

```bash
cd sim/testbench
make            # Runs all cocotb tests
```

### 6.2 Viewing Waveforms

Tests produce `.vcd` files in `sim/testbench/`. View with GTKWave:
```bash
gtkwave sim/testbench/dump.vcd
```

### 6.3 UART Console Log

Firmware UART output is captured to `sim/testbench/uart_log.txt` during simulation.

### 6.4 Test Selection

```bash
make test TEST=test_soc_boot    # Run single test
make test TEST=test_gpio        # GPIO register test
make test TEST=test_uart_echo   # UART loopback test
```

---

## 7. Formal Verification

### 7.1 Running Formal Proofs

```bash
# All module-level proofs
cd formal
sby -f wb_dma.sby
sby -f wb_gpio.sby
# ... (7 non-CDC modules)

# CDC modules (smtbmc recommended)
sby -f wb_usb.sby --engine smtbmc
sby -f wb_debug.sby --engine smtbmc
```

### 7.2 Interpreting Results

- **PASS** — All assertions hold for all reachable states (bounded + inductive)
- **FAIL** — Counterexample found (trace in `engine_0/trace.vcd`)
- **ERROR** — Toolchain limitation (e.g., dual-clock CDC modules under ABC PDR)

---

## 8. Troubleshooting

### 8.1 Build Issues

| Symptom | Likely Cause | Fix |
|---|---|---|
| `riscv32-unknown-elf-gcc: not found` | Toolchain not in PATH | Add GCC to PATH or install |
| `undefined reference to _start` | Missing startup.S | Compile with startup.S in source list |
| `section .text exceeds SRAM` | Firmware too large (8 KB limit) | Reduce code size, use -Os |
| `linker.ld: no such file` | Wrong working directory | Run from sw/boot |

### 8.2 JTAG / Debug Issues

| Symptom | Likely Cause | Fix |
|---|---|---|
| `JTAG scan chain interrogation failed` | No FPGA bitstream loaded | Program FPGA first |
| `TDO seems to be stuck` | JTAG clock too fast | Reduce adapter speed to 1000 kHz |
| `cannot halt CPU` | wb_debug not in bus map | Verify SoC includes debug peripheral |
| `No symbols in firmware.elf` | Debug info stripped | Build with `make debug` |

### 8.3 Simulation Issues

| Symptom | Likely Cause | Fix |
|---|---|---|
| `Module not found` | Verilator not installed | Install Verilator 5.0+ |
| `UART timeout` | Firmware not reaching test case | Check firmware output in console log |
| `Test FAILED` | Assertion in testbench failed | Examine waveform in .vcd |

---

## 9. Common Workflows

### 9.1 Edit-Build-Test Cycle

```bash
# Edit firmware
vim sw/boot/test_prog.c

# Build
cd sw/boot && make

# Simulate
cd ../../sim/testbench && make

# Debug if fails
# → Open .vcd in GTKWave
# → Add UART debug prints
```

### 9.2 Edit-Build-Flash-Run Cycle

```bash
# Edit firmware
vim sw/boot/test_prog.c

# Build
cd sw/boot && make

# Flash via JTAG
make flash

# Open console
make monitor
```

---

## 10. Revision History

| Rev | Date | Description |
|---|---|---|
| 1.0 | 2026-07-16 | Initial toolchain & debug guide |

---

*RideProtect-RV Toolchain & Debugging Guide v1.0*
