# RideProtect-RV — Build Environment Setup Guide

**Purpose:** Complete step-by-step environment setup so developers can build, simulate, and deploy from scratch.

---

## 1. Linux (Ubuntu 22.04 / 24.04)

### 1.1 Package Installation

```bash
# RISC-V cross-compiler
sudo apt-get install -y gcc-riscv64-linux-gnu

# Verilator 5 (Ubuntu 24.04 ships Verilator 5)
sudo apt-get install -y verilator

# For Ubuntu 22.04, install Verilator from source:
sudo apt-get install -y git perl python3 make autoconf g++ flex bison ccache
git clone https://github.com/verilator/verilator
cd verilator
git checkout v5.032
autoconf && ./configure && make -j$(nproc) && sudo make install
cd ..

# Yosys
sudo apt-get install -y yosys

# Utilities
sudo apt-get install -y bsdmainutils  # hexdump
sudo apt-get install -y make

# Optional: waveform viewer
sudo apt-get install -y gtkwave

# Optional: Vivado (see Section 4)
```

### 1.2 Verifying Installation

```bash
riscv64-linux-gnu-gcc --version | head -1
verilator --version
yosys --version
make --version
```

Expected output:
```
riscv64-linux-gnu-gcc (Ubuntu 13.x) 13.x.x
Verilator 5.032
Yosys 0.52
GNU Make 4.3
```

### 1.3 Building the SoC

```bash
cd rideprotect-rv
make firmware    # Build firmware (tests pass)
make sim         # Run simulation (15 tests)
make syn         # Yosys synthesis
```

---

## 2. Windows (WSL2)

### 2.1 Install WSL2

```powershell
# In PowerShell (Admin):
wsl --install -d Ubuntu-24.04
```

### 2.2 Setup in WSL

```bash
# Inside WSL terminal
sudo apt-get update
sudo apt-get install -y gcc-riscv64-linux-gnu verilator yosys bsdmainutils make
```

### 2.3 Clone and Build

```bash
cd /mnt/c/Projects/ChipDesign/rideprotect-rv
make firmware
make sim
make syn
```

### 2.4 Visual Studio Code Integration

```bash
# In WSL
code /mnt/c/Projects/ChipDesign/rideprotect-rv
```

Install VS Code extensions:
- `ms-vscode.cpptools` — C/C++ IntelliSense
- `zhwu95.riscv` — RISC-V assembly highlighting
- `mshr-h.veriloghdl` — Verilog/SystemVerilog support

---

## 3. macOS

```bash
# Install Homebrew: https://brew.sh
brew install riscv64-linux-gnu-binutils riscv64-linux-gnu-gcc
brew install verilator yosys
brew install coreutils  # for hexdump
```

---

## 4. Vivado Installation (for FPGA Bitstream)

### 4.1 Download

Download Vivado HL WebPACK from [AMD/Xilinx](https://www.amd.com/en/products/software/adaptive-socs-and-fpgas/vivado.html)

- **Free WebPACK** is sufficient for Artix-7 (all speed/temperature grades including automotive XA)
- **Required disk space:** ~40 GB (full installation) or ~15 GB (device-limited)
- **Devices to select:** Artix-7

### 4.2 Environment Setup

**Linux:**
```bash
source /tools/Xilinx/Vivado/2024.1/settings64.sh
# Add to ~/.bashrc for persistence
```

**Windows:**
```powershell
# Vivado adds itself to PATH during installation
# Or run from "Xilinx Vivado 2024.1" start menu shortcut
```

### 4.3 Verify

```bash
vivado -version
```

---

## 5. Docker Environment (Optional)

A Dockerfile for reproducible builds:

```dockerfile
FROM ubuntu:24.04
RUN apt-get update && apt-get install -y \
    gcc-riscv64-linux-gnu \
    verilator \
    yosys \
    bsdmainutils \
    make
WORKDIR /workspace
```

```bash
docker build -t rideprotect-rv .
docker run --rm -v $(pwd):/workspace rideprotect-rv make sim
```

---

## 6. Toolchain Details

| Tool | Version Required | Source |
|------|-----------------|--------|
| riscv64-linux-gnu-gcc | ≥ 12.x | Ubuntu apt / bootlin.com |
| Verilator | ≥ 5.0 | Ubuntu 24.04 apt / GitHub |
| Yosys | ≥ 0.30 | Ubuntu apt / YosysHQ |
| Vivado | ≥ 2022.1 | AMD/Xilinx website |
| Make | ≥ 4.0 | Ubuntu apt |
| hexdump (bsdmainutils) | any | Ubuntu apt |

---

## 7. Common Installation Issues

**Issue: `riscv64-linux-gnu-gcc: No such file or directory`**
→ Install the cross-compiler: `sudo apt-get install gcc-riscv64-linux-gnu`

**Issue: `verilator: command not found`**
→ Ubuntu 22.04 ships Verilator 4.x. Install Verilator 5 from source (see Section 1.1).

**Issue: `yosys: command not found`**
→ Install: `sudo apt-get install yosys`

**Issue: `/lib64/ld-linux-x86-64.so.2: No such file or directory`**
→ On WSL, install 64-bit support: `sudo apt-get install libc6-dev`
