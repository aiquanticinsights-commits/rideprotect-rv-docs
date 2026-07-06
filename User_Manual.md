# RideProtect-RV SoC — User Manual

**Document Version:** 1.0  
**Date:** 2026-07-04  
**SoC Version:** Phase 5 (RV32IMC + 13 peripherals)

---

## Table of Contents

1. [Introduction](#1-introduction)
2. [Getting Started](#2-getting-started)
3. [Building the Firmware](#3-building-the-firmware)
4. [Running Simulation](#4-running-simulation)
5. [FPGA Deployment](#5-fpga-deployment)
6. [Peripheral Programming Guide](#6-peripheral-programming-guide)
7. [Register Reference](#7-register-reference)
8. [JTAG Debug](#8-jtag-debug)
9. [Memory Map Reference](#9-memory-map-reference)
10. [Troubleshooting](#10-troubleshooting)
11. [FAQ](#11-faq)

---

## 1. Introduction

RideProtect-RV is a RISC-V-based SoC designed for vehicle telematics applications. It integrates a VexRiscv CPU with 13 peripherals over a Wishbone B4 shared bus.

### 1.1 System Overview

- **CPU:** VexRiscv RV32IMC (5-stage pipeline, 4 KB instruction cache)
- **Clock:** 50 MHz system clock (divided from 100 MHz input on FPGA)
- **Bus:** Wishbone B4 shared bus with 2 masters (CPU + DMA) and 13 slaves
- **Memory:** 8 KB on-chip SRAM (code + data)
- **Debug:** IEEE 1149.1 JTAG via external TAP controller

### 1.2 Peripheral Summary

| # | Peripheral | Address | Purpose |
|---|-----------|---------|---------|
| 1 | SRAM | 0x0000_0000 | Code and data memory (8 KB) |
| 2 | GPIO | 0x2000_0000 | 32-bit general-purpose I/O |
| 3 | UART | 0x2000_1000 | Serial communication (115200 baud) |
| 4 | Timer | 0x2000_2000 | Timer/PWM/Capture |
| 5 | INTC | 0x2000_3000 | 16-source interrupt controller |
| 6 | WDT | 0x2000_4000 | Watchdog timer |
| 7 | SPI | 0x2000_5000 | SPI master (4 CS, dual/quad mode) |
| 8 | I2C | 0x2000_6000 | I2C master |
| 9 | CAN | 0x2000_7000 | CAN 2.0B controller |
| 10 | DMA | 0x2000_8000 | Memory-to-memory DMA |
| 11 | PWM Adv | 0x2000_9000 | 4-channel advanced PWM |
| 12 | RTC | 0x2000_A000 | Real-time clock with alarm |
| 13 | Debug | 0x2000_B000 | JTAG debug + mailbox |

---

## 2. Getting Started

### 2.1 Prerequisites

| Tool | Version | Purpose |
|------|---------|---------|
| `riscv64-linux-gnu-gcc` | ≥ 12.x | RISC-V cross-compiler |
| `verilator` | ≥ 5.0 | RTL simulation |
| `yosys` | ≥ 0.30 | RTL synthesis |
| `vivado` | ≥ 2022.1 | FPGA P&R (optional for sim only) |
| `make` | ≥ 4.0 | Build system |
| `hexdump` | any | Binary-to-hex conversion |

**WSL2 (Windows users):** The recommended approach is to build from an Ubuntu 22.04/24.04 WSL2 instance. All paths in this manual assume a Linux/WSL environment.

### 2.2 Quick Start

```bash
# Clone the repository
git clone <repo-url> rideprotect-rv
cd rideprotect-rv

# Build firmware
make firmware

# Run lint check
make lint

# Run simulation
make sim

# Run Yosys synthesis
make syn

# Clean build artifacts
make clean
```

### 2.3 Repository Structure

```
rideprotect-rv/
├── Makefile              — Build system (firmware, sim, lint, syn, pnr)
├── rtl/                  — RTL sources (.v)
├── sim/                  — Simulation testbench
├── sw/boot/              — Firmware source (test_prog.c, startup.s, linker.ld)
├── fpga/                 — FPGA constraints + TCL scripts
├── docs/                 — Documentation
└── build/                — Generated artifacts (firmware.hex, sim.log, etc.)
```

---

## 3. Building the Firmware

### 3.1 Building from Source

```bash
make firmware
```

This produces:
- `build/firmware.elf` — ELF binary
- `build/firmware.bin` — Raw binary
- `build/firmware.hex` — Hex dump for BRAM initialization
- `build/firmware.dis` — Disassembly listing
- `build/firmware.map` — Memory map

### 3.2 Writing Custom Firmware

Firmware source files are in `sw/boot/`:

| File | Purpose |
|------|---------|
| `test_prog.c` | Main program (replace with your code) |
| `startup.s` | Boot code, IRQ handler, irq_counter |
| `linker.ld` | Memory layout |

**To write custom firmware:**

1. Edit `sw/boot/test_prog.c` — replace `main()` with your application code
2. If needed, update `startup.s` for custom IRQ handling
3. Run `make firmware`

**Example: Blink LEDs**

```c
#define GPIO_BASE   0x20000000
#define GPIO_DATA_OUT (*(volatile unsigned int *)(GPIO_BASE + 0x00))
#define GPIO_DIR      (*(volatile unsigned int *)(GPIO_BASE + 0x08))

void delay(volatile int count) {
    while (count--);
}

int main(void) {
    GPIO_DIR = 0x0000000F;     // First 4 pins as output (LEDs)
    while (1) {
        GPIO_DATA_OUT = 0x0000000F;  // All LEDs on
        delay(500000);
        GPIO_DATA_OUT = 0x00000000;  // All LEDs off
        delay(500000);
    }
}
```

### 3.3 Firmware Constraints

- **Maximum size:** 8,192 bytes (fits in 8 KB SRAM)
- **Linker origin:** 0x00000000
- **Stack top:** 0x00001FFC (top of SRAM minus 4 bytes)
- **No libc:** Use `-nostdlib -nostartfiles -ffreestanding`
- **IRQ vector:** 0x00000010 (mtvec)
- **No MMU:** All addresses are physical

### 3.4 Preprocessor Symbols

The firmware is compiled with `-DSYNTHESIS` NOT defined (the firmware runs identically in sim and on hardware). The `SYNTHESIS` define is only used during FPGA synthesis to enable FPGA-specific logic (BUFG, clock divider).

---

## 4. Running Simulation

### 4.1 Basic Simulation

```bash
make sim
```

This:
1. Builds a Verilator model from all RTL sources + `sim_top.v`
2. Compiles the C++ testbench (`sim/sim_main.cpp`)
3. Runs simulation for 5,000,000 cycles
4. Prints test results

**Output:**
```
=== >>> JTAG TAP TEST PASSED <<< ===
=== >>> ECHO TEST PASSED <<< ===
=== >>> TIMER INTERRUPT TEST PASSED <<< ===
=== >>> GPIO INTERRUPT TEST PASSED <<< ===
=== >>> DMA TEST SEEN <<< ===
=== >>> PWM ADV TEST SEEN <<< ===
=== >>> RTC TEST SEEN <<< ===
=== >>> DEBUG TEST SEEN <<< ===
=== >>> BUS TIMEOUT TEST PASSED <<< ===
=== >>> Phase 5 simulation complete <<< ===
```

### 4.2 Viewing Waveforms

The simulation generates `build/rideprotect_rv.fst` (FST format, trace depth 99).

Open with:
```bash
gtkwave build/rideprotect_rv.fst
```

Key signals to observe:

| Signal | Path | Purpose |
|--------|------|---------|
| clk | top→clk | System clock |
| rst_n | top→rst_n | Reset |
| uart_tx | top→uart_tx | UART output |
| gpio | top→gpio | GPIO pins |
| irq | top→irq | Interrupt line |
| pwm_out | top→pwm_out | Timer PWM |
| sck | top→sck | SPI clock |
| sda | top→sda | I2C data |
| can_tx | top→can_tx | CAN output |
| pwm_adv_out | top→pwm_adv_out | Advanced PWM |
| jtag_tck | top→jtag_tck | JTAG clock |

### 4.3 Simulation Testbench Details

The testbench (`sim/sim_main.cpp`) provides:

- **JTAG test:** Bit-bangs JTAG protocol to verify IDCODE and debug mailbox read/write
- **UART receiver:** Samples TX line at 16× baud, assembles bytes, parses test output
- **GPIO stimulus:** Drives gpio[0] high during GPIO IRQ test
- **Capture stimulus:** Pulses capture_in during timer capture test
- **PWM measurement:** Counts high/low samples for duty cycle verification
- **Echo test:** Sends 'A' via UART RX and verifies loopback on TX

### 4.4 Modifying Simulation Parameters

Edit `sim/sim_main.cpp`:

```cpp
// Change simulation length (line ~176)
for (int cycles = 0; cycles < 5000000; cycles++) {

// Change baud rate divisor (line ~33)
const int BIT_TIME = 27 * 16;  // 27 = UART divisor from firmware
```

---

## 5. FPGA Deployment

### 5.1 Target Boards

| Board | Part Number | Make Target |
|-------|-------------|-------------|
| Arty A7-35T | xc7a35ticsg324-1L | `make pnr-arty` |
| Nexys A7-100T | xc7a100tcsg324-1 | `make pnr-nexys` |

### 5.2 Full Deployment Flow

```bash
# Full flow: firmware + Yosys synthesis + Vivado P&R
make fpga-arty

# Or step by step:
make firmware         # Step 1: Build firmware
make syn              # Step 2: Yosys synthesis (resource estimation)
make pnr-arty         # Step 3: Vivado P&R → build/soc_top.bit
```

### 5.3 Bitstream Programming

**Using Vivado GUI:**
1. Open Vivado Hardware Manager
2. Connect Arty A7 via USB
3. Auto-connect → Add Configuration Memory Device
4. Select bitstream: `build/soc_top.bit`
5. Program

**Using openFPGALoader:**
```bash
openFPGALoader -b arty_a35t build/soc_top.bit
```

**Using Vivado batch:**
```bash
vivado -mode batch -source <(echo "
    open_hw_manager;
    connect_hw_server;
    open_hw_target;
    current_hw_device [lindex [get_hw_devices] 0];
    refresh_hw_device -update_hw_probes false [current_hw_device];
    set_property PROGRAM.FILE {build/soc_top.bit} [current_hw_device];
    program_hw_devices [current_hw_device];")
```

### 5.4 Hardware Verification

**Serial monitor (115200 8-N-1):**
```bash
screen /dev/ttyUSB1 115200
# or
minicom -D /dev/ttyUSB1 -b 115200
# or Windows:
# putty COM3 115200 8-N-1
```

Expected output:
```
RideProtect-RV Phase 5
Timer: PWM enabled (25% duty)
  period=0x0000C350
  duty=0x000030D4
...
```

### 5.5 Pin Connections

**Arty A7-35T Pin Map:**

| SoC Signal | Arty Pin | Connector |
|------------|----------|-----------|
| uart_tx | D10 | USB-UART (FTDI) |
| uart_rx | A9 | USB-UART (FTDI) |
| gpio[0:3] | H4,J3,J4,K1 | LD0-LD3 (LEDs) |
| gpio[4:7] | A8,C11,C10,A10 | SW0-SW3 (switches) |
| pwm_out | G17 | Pmod JA pin 1 |
| capture_in | K17 | Pmod JA pin 2 |
| wdt_reset | H15 | Pmod JA pin 3 |
| pwm_adv_out[0:1] | G13,H14 | Pmod JA pins 4,7 |
| jtag_tck/tms/tdi/tdo | D3,K15,J17,H16 | Pmod JD-2 (MRCC), Pmod JB-6, Pmod JB-4, Pmod JA-10 |
| sck/mosi/miso | A14,A15,B14 | Pmod JB pins 1-3 |
| cs_n[0:3] | A16,C15,B17,A18 | Pmod JB pins 4,7,8,10 |
| io2/io3 | F16,E16 | Pmod JB pins 9-10 |
| sda/scl | K18,L18 | Pmod JC pins 1-2 |
| jtag_trst | K16 | Pmod JC pin 4 / Digilent ja[7] |
| can_tx/rx | D14,F15 | Pmod JD pins 1-2 |
| pwm_adv_out[2:3] | D15,C17 | Pmod JD pins 3-4 |
| fault_n | D17 | Pmod JD pin 7 |
| irq | J15 | Pmod JC pin 3 / Digilent jb[7] |
| rst_n | D4 | BTN0 |
| clk | E3 | 100 MHz oscillator |

---

## 6. Peripheral Programming Guide

### 6.1 GPIO Programming

```c
// Memory-mapped register definitions
#define GPIO_BASE    0x20000000
#define GPIO_DATA_OUT (*(volatile unsigned int *)(GPIO_BASE + 0x00))
#define GPIO_DATA_IN  (*(volatile unsigned int *)(GPIO_BASE + 0x04))
#define GPIO_DIR      (*(volatile unsigned int *)(GPIO_BASE + 0x08))
#define GPIO_IRQ_EN   (*(volatile unsigned int *)(GPIO_BASE + 0x10))
#define GPIO_IRQ_EDGE (*(volatile unsigned int *)(GPIO_BASE + 0x14))
#define GPIO_IRQ_STAT (*(volatile unsigned int *)(GPIO_BASE + 0x18))

// Set pins 0-7 as output, rest as input
GPIO_DIR = 0x000000FF;

// Write output
GPIO_DATA_OUT = 0xAAAAAAAA;

// Read input
unsigned int pins = GPIO_DATA_IN;

// Configure pin 0 rising edge interrupt
GPIO_IRQ_EDGE = 0x00000000;  // Rising edge
GPIO_IRQ_EN = 0x00000001;    // Enable pin 0 IRQ

// Clear interrupt status
GPIO_IRQ_STAT = 0xFFFFFFFF;  // W1C
```

### 6.2 UART Programming

```c
#define UART_BASE    0x20001000
#define UART_DATA    (*(volatile unsigned char *)(UART_BASE + 0x00))
#define UART_STATUS  (*(volatile unsigned char *)(UART_BASE + 0x04))
#define UART_CTRL    (*(volatile unsigned char *)(UART_BASE + 0x08))
#define UART_DIV     (*(volatile unsigned short *)(UART_BASE + 0x0C))

#define UART_TX_READY 0x01
#define UART_RX_READY 0x02

// Initialize: 115200 baud @ 50 MHz
UART_DIV = 27;           // Divisor = 50e6 / (16 × 115200) ≈ 27.12
UART_CTRL = 0x03;        // TX + RX enable

// Transmit a character
void uart_tx(char c) {
    while (!(UART_STATUS & UART_TX_READY));
    UART_DATA = c;
}

// Receive a character
char uart_rx(void) {
    while (!(UART_STATUS & UART_RX_READY));
    return UART_DATA;
}

// Print a string
void uart_puts(const char *s) {
    while (*s) uart_tx(*s++);
}
```

**Baud rate table (@ 50 MHz):**

| Baud Rate | Divisor | Actual | Error |
|-----------|---------|--------|-------|
| 9600 | 326 | 9580 | −0.21% |
| 19200 | 163 | 19172 | −0.15% |
| 38400 | 81 | 38580 | +0.47% |
| 57600 | 54 | 57870 | +0.47% |
| 115200 | 27 | 115740 | +0.47% |
| 230400 | 13 | 240385 | +4.33% (marginal) |

### 6.3 Timer/PWM Programming

```c
#define TIMER_BASE    0x20002000
#define TIMER_CTRL    (*(volatile unsigned int *)(TIMER_BASE + 0x00))
#define TIMER_PERIOD  (*(volatile unsigned int *)(TIMER_BASE + 0x04))
#define TIMER_DUTY    (*(volatile unsigned int *)(TIMER_BASE + 0x08))
#define TIMER_COUNTER (*(volatile unsigned int *)(TIMER_BASE + 0x0C))
#define TIMER_CAPTURE (*(volatile unsigned int *)(TIMER_BASE + 0x10))
#define TIMER_IRQ_CTRL (*(volatile unsigned int *)(TIMER_BASE + 0x14))
#define TIMER_IRQ_FLAG (*(volatile unsigned int *)(TIMER_BASE + 0x18))

// Configure 25% PWM at 1 kHz (50 MHz / 50000)
TIMER_PERIOD = 50000;
TIMER_DUTY = 12500;       // 25% of period
TIMER_CTRL = 0x03;        // Enable timer + PWM output

// Read current counter
unsigned int cnt = TIMER_COUNTER;

// Enable overflow interrupt
TIMER_IRQ_CTRL = 0x01;    // Bit 0 = overflow IRQ
INTC_ENABLE = 0x7FF;      // Enable all interrupt sources

// Clear interrupt flag (in IRQ handler)
TIMER_IRQ_FLAG = 0x01;    // W1C overflow flag

// Read capture register (latched on capture_in rising edge)
unsigned int cap = TIMER_CAPTURE;
```

### 6.4 Interrupt Controller Programming

```c
#define INTC_BASE    0x20003000
#define INTC_STATUS  (*(volatile unsigned int *)(INTC_BASE + 0x00))
#define INTC_ENABLE  (*(volatile unsigned int *)(INTC_BASE + 0x04))
#define INTC_PENDING (*(volatile unsigned int *)(INTC_BASE + 0x08))
#define INTC_SOURCE  (*(volatile unsigned int *)(INTC_BASE + 0x0C))

// Enable interrupts from specific peripherals
INTC_ENABLE = (1 << 0) |   // GPIO
              (1 << 2) |   // Timer
              (1 << 7);    // DMA

// Enable machine external interrupts in CPU
__asm__ volatile ("csrsi mstatus, 8");   // Set MIE bit
__asm__ volatile ("li t0, %0; csrw mie, t0" :: "i"(1 << 11));  // MEI enable

// Read pending interrupts
unsigned int pending = INTC_PENDING;

// Get highest-priority pending source
unsigned int source = INTC_SOURCE;

// Clear pending interrupt
INTC_PENDING = (1 << source);  // W1C
```

### 6.5 Watchdog Programming

```c
#define WDT_BASE     0x20004000
#define WDT_CTRL     (*(volatile unsigned int *)(WDT_BASE + 0x00))
#define WDT_COMPARE  (*(volatile unsigned int *)(WDT_BASE + 0x04))
#define WDT_PET      (*(volatile unsigned int *)(WDT_BASE + 0x08))

// Configure 1-second timeout (50 MHz clock)
WDT_COMPARE = 50000000;
WDT_CTRL = 0x03;           // Enable + reset output on timeout

// Pet the watchdog (must be called before timeout)
WDT_PET = 0x01;            // Any value works

// To read current settings
unsigned int ctrl = WDT_CTRL;      // Returns 0x03 if enabled
```

### 6.6 SPI Programming

```c
#define SPI_BASE     0x20005000
#define SPI_CTRL     (*(volatile unsigned int *)(SPI_BASE + 0x00))
#define SPI_DIV      (*(volatile unsigned int *)(SPI_BASE + 0x04))
#define SPI_DATA     (*(volatile unsigned int *)(SPI_BASE + 0x08))
#define SPI_STATUS   (*(volatile unsigned int *)(SPI_BASE + 0x0C))
#define SPI_CONF     (*(volatile unsigned int *)(SPI_BASE + 0x10))
#define SPI_CS_SEL   (*(volatile unsigned int *)(SPI_BASE + 0x14))

// Initialize SPI (mode 0, 1 MHz @ 50 MHz)
SPI_CTRL = 0x01;           // Enable, CPOL=0, CPHA=0
SPI_DIV = 50;              // SCK = 50 MHz / 50 = 1 MHz

// Transfer a byte (write triggers transfer)
SPI_DATA = 0xAA;           // TX + start transfer
while (SPI_STATUS & 0x01); // Wait for busy=0
unsigned int rx = SPI_DATA; // Read received data

// Select chip
SPI_CS_SEL = 0x01;         // CS0 active

// Configure for dual mode
SPI_CONF = 0x01;           // Bit 0 = dual_mode

// Word size options (CONTROL bits 3-4):
// 00 = 8-bit, 01 = 16-bit, 10 = 32-bit
```

### 6.7 I2C Programming

```c
#define I2C_BASE     0x20006000
#define I2C_CTRL     (*(volatile unsigned int *)(I2C_BASE + 0x00))
#define I2C_DIV      (*(volatile unsigned int *)(I2C_BASE + 0x04))
#define I2C_DATA     (*(volatile unsigned int *)(I2C_BASE + 0x08))
#define I2C_STATUS   (*(volatile unsigned int *)(I2C_BASE + 0x0C))

// Initialize I2C (100 kHz @ 50 MHz)
I2C_CTRL = 0x01;           // Enable
I2C_DIV = 250;             // SCL = 50 MHz / (2 × 250) = 100 kHz

// Write to I2C device at address 0x3C
// I2C_DATA format: [15:8] = address|R/W, [7:0] = data
I2C_DATA = (0xA5 << 8) | (0x3C << 1) | 0;  // Write
while (I2C_STATUS & 0x02);  // Wait for busy=0
if (I2C_STATUS & 0x01) {
    // ACK error
}

// Read from I2C device
I2C_DATA = (0x3C << 1) | 1;  // Read
while (I2C_STATUS & 0x02);
unsigned int data = I2C_DATA & 0xFF;
```

### 6.8 CAN 2.0B Programming

```c
#define CAN_BASE     0x20007000
#define CAN_CTRL     (*(volatile unsigned int *)(CAN_BASE + 0x00))
#define CAN_BIT_TIM  (*(volatile unsigned int *)(CAN_BASE + 0x04))
#define CAN_TX_ID    (*(volatile unsigned int *)(CAN_BASE + 0x08))
#define CAN_TX_D0    (*(volatile unsigned int *)(CAN_BASE + 0x0C))
#define CAN_TX_D1    (*(volatile unsigned int *)(CAN_BASE + 0x10))
#define CAN_TX_DLC   (*(volatile unsigned int *)(CAN_BASE + 0x14))
#define CAN_TX_REQ   (*(volatile unsigned int *)(CAN_BASE + 0x18))
#define CAN_RX_ID    (*(volatile unsigned int *)(CAN_BASE + 0x1C))
#define CAN_RX_D0    (*(volatile unsigned int *)(CAN_BASE + 0x20))
#define CAN_RX_D1    (*(volatile unsigned int *)(CAN_BASE + 0x24))
#define CAN_STATUS   (*(volatile unsigned int *)(CAN_BASE + 0x30))

// Initialize CAN (loopback mode)
CAN_CTRL = 0x03;            // Enable + loopback
CAN_BIT_TIM = 0x00042004;   // BRP=4, TSEG1=4, TSEG2=4

// Transmit a CAN frame
CAN_TX_ID = 0x1AA00000;     // 29-bit extended ID
CAN_TX_D0 = 0xDEADBEEF;     // Data bytes 0-3
CAN_TX_D1 = 0x00000000;     // Data bytes 4-7
CAN_TX_DLC = 0x04;          // 4 data bytes
CAN_TX_REQ = 0x01;          // Trigger transmission

// Wait for completion
while (CAN_STATUS & 0x02);  // Wait for tx_busy=0

// Read received frame
unsigned int rx_id = CAN_RX_ID;
unsigned int rx_d0 = CAN_RX_D0;
unsigned int status = CAN_STATUS;
```

### 6.9 DMA Programming

```c
#define DMA_BASE     0x20008000
#define DMA_SRC_ADDR (*(volatile unsigned int *)(DMA_BASE + 0x00))
#define DMA_DST_ADDR (*(volatile unsigned int *)(DMA_BASE + 0x04))
#define DMA_COUNT    (*(volatile unsigned int *)(DMA_BASE + 0x08))
#define DMA_CTRL     (*(volatile unsigned int *)(DMA_BASE + 0x0C))
#define DMA_STATUS   (*(volatile unsigned int *)(DMA_BASE + 0x10))
#define DMA_IRQ_EN   (*(volatile unsigned int *)(DMA_BASE + 0x14))

// Copy 4 words from SRAM address 0x800 to 0x900
DMA_SRC_ADDR = 0x00000800;  // Source (in SRAM)
DMA_DST_ADDR = 0x00000900;  // Destination (in SRAM)
DMA_COUNT = 4;              // 4 words
DMA_CTRL = 0x0F;            // Start + src_inc + dst_inc + irq_en

// Wait for completion
while (DMA_STATUS & 0x01);  // Wait for busy=0
unsigned int status = DMA_STATUS;
if (status & 0x02) {
    // Transfer complete (done flag)
}

// Clear status
DMA_CTRL = 0x00;
```

### 6.10 Advanced PWM Programming

```c
#define PWM_BASE      0x20009000
#define PWM_CH0_CTRL    (*(volatile unsigned int *)(PWM_BASE + 0x00))
#define PWM_CH0_PERIOD  (*(volatile unsigned int *)(PWM_BASE + 0x04))
#define PWM_CH0_DUTY    (*(volatile unsigned int *)(PWM_BASE + 0x08))
#define PWM_CH0_COUNTER (*(volatile unsigned int *)(PWM_BASE + 0x0C))
#define PWM_CH1_CTRL    (*(volatile unsigned int *)(PWM_BASE + 0x10))
#define PWM_CH1_PERIOD  (*(volatile unsigned int *)(PWM_BASE + 0x14))
#define PWM_CH1_DUTY    (*(volatile unsigned int *)(PWM_BASE + 0x18))
#define PWM_GLOBAL_CTRL (*(volatile unsigned int *)(PWM_BASE + 0x40))
#define PWM_IRQ_ENABLE  (*(volatile unsigned int *)(PWM_BASE + 0x44))
#define PWM_IRQ_FLAG    (*(volatile unsigned int *)(PWM_BASE + 0x48))

// Configure channel 0 for 25% PWM
PWM_CH0_PERIOD = 50000;
PWM_CH0_DUTY = 12500;
PWM_CH0_CTRL = 0x01;         // Enable channel 0
PWM_GLOBAL_CTRL = 0x01;      // Global enable

// Center-aligned mode
PWM_CH0_CTRL = 0x05;         // Enable + center_align

// Inverted polarity
PWM_CH0_CTRL = 0x03;         // Enable + polarity_invert

// Fault handling (fault_n = 0 latches all outputs to zero)
// Clear fault after fault_n returns high:
PWM_GLOBAL_CTRL = 0x03;      // global_en + fault_clear
PWM_GLOBAL_CTRL = 0x01;      // Back to normal operation
```

### 6.11 RTC Programming

```c
#define RTC_BASE     0x2000A000
#define RTC_SEC      (*(volatile unsigned int *)(RTC_BASE + 0x00))
#define RTC_MIN      (*(volatile unsigned int *)(RTC_BASE + 0x04))
#define RTC_HOUR     (*(volatile unsigned int *)(RTC_BASE + 0x08))
#define RTC_DAY      (*(volatile unsigned int *)(RTC_BASE + 0x0C))
#define RTC_MONTH    (*(volatile unsigned int *)(RTC_BASE + 0x10))
#define RTC_YEAR     (*(volatile unsigned int *)(RTC_BASE + 0x14))
#define RTC_ALARM    (*(volatile unsigned int *)(RTC_BASE + 0x18))
#define RTC_CTRL     (*(volatile unsigned int *)(RTC_BASE + 0x1C))

// Set time to 2026-07-04 12:30:00
RTC_CTRL = 0x01;          // Enable RTC
RTC_SEC = 0x00;
RTC_MIN = 0x1E;           // 30 minutes
RTC_HOUR = 0x0C;          // 12 hours
RTC_DAY = 0x04;           // 4th
RTC_MONTH = 0x07;         // July
RTC_YEAR = 0x2026;

// Set alarm for 12:30:01
RTC_ALARM = (0x0C << 16) | (0x1E << 8) | 0x01;

// Enable alarm interrupt
RTC_CTRL = 0x07;          // enable + alarm_en + irq_en

// Read back
unsigned int hour = RTC_HOUR;
unsigned int min = RTC_MIN;
unsigned int sec = RTC_SEC;
```

### 6.12 Debug Module Programming

```c
#define DEBUG_BASE    0x2000B000
#define DEBUG_CTRL    (*(volatile unsigned int *)(DEBUG_BASE + 0x00))
#define DEBUG_STATUS  (*(volatile unsigned int *)(DEBUG_BASE + 0x04))
#define DEBUG_MAILBOX (*(volatile unsigned int *)(DEBUG_BASE + 0x40))

// Write to debug mailbox
DEBUG_MAILBOX = 0xDEADBEEF;

// Read back
unsigned int val = DEBUG_MAILBOX;

// Check debug status
unsigned int status = DEBUG_STATUS;
// Bit 0 = CPU halted
// Bit 8 = breakpoint active

// Issue CPU halt
DEBUG_CTRL = 0x01;

// Issue CPU reset
DEBUG_CTRL = 0x02;

// Issue external IRQ
DEBUG_CTRL = 0x04;
```

---

## 7. Register Reference

### 7.1 GPIO — 0x2000_0000

| Offset | Name | 31 | 30 | ... | 1 | 0 | Access | Reset |
|--------|------|----|----|-----|---|---|--------|-------|
| 0x00 | DATA_OUT | — output data bits — | R/W | 0 |
| 0x04 | DATA_IN | — synchronized input bits — | RO | — |
| 0x08 | DIR | — direction (1=out, 0=in) — | R/W | 0 |
| 0x10 | IRQ_ENABLE | — per-pin IRQ enable — | R/W | 0 |
| 0x14 | IRQ_EDGE | — edge (0=rise, 1=fall) — | R/W | 0 |
| 0x18 | IRQ_STATUS | — sticky (W1C) — | RC/W1C | 0 |

### 7.2 UART — 0x2000_1000

| Offset | Bit | Name | Description |
|--------|-----|------|-------------|
| 0x00 | [7:0] | DATA | Write = TX data, Read = RX data |
| 0x04 | [0] | TX_READY | 1 = TX buffer empty, ready for next byte |
| 0x04 | [1] | RX_READY | 1 = RX data available |
| 0x08 | [0] | TX_EN | Enable UART transmitter |
| 0x08 | [1] | RX_EN | Enable UART receiver |
| 0x0C | [15:0] | DIV | Baud rate divisor |

### 7.3 Timer — 0x2000_2000

| Offset | Bit | Name | Description |
|--------|-----|------|-------------|
| 0x00 | [0] | EN | Timer enable |
| 0x00 | [1] | PWM_EN | PWM output enable |
| 0x00 | [2] | CAPTURE_EN | Input capture enable |
| 0x04 | [31:0] | PERIOD | Counter wraps at this value |
| 0x08 | [31:0] | DUTY | PWM compare value |
| 0x0C | [31:0] | COUNTER | Current counter (RO) |
| 0x10 | [31:0] | CAPTURE | Latched counter value (RO) |
| 0x14 | [0] | IRQ_OVF | Overflow interrupt enable |
| 0x14 | [1] | IRQ_CAP | Capture interrupt enable |
| 0x18 | [0] | IRQ_OVF_FLAG | Overflow flag (W1C) |
| 0x18 | [1] | IRQ_CAP_FLAG | Capture flag (W1C) |

### 7.4 INTC — 0x2000_3000

| Offset | Bit | Name | Description |
|--------|-----|------|-------------|
| 0x00 | [15:0] | SOURCE_STATUS | Raw source status (RO) |
| 0x04 | [15:0] | ENABLE | Interrupt enable mask |
| 0x08 | [15:0] | PENDING | Pending interrupts (W1C) |
| 0x0C | [3:0] | SOURCE_VECTOR | Highest-priority pending source (RO) |

### 7.5 Watchdog — 0x2000_4000

| Offset | Bit | Name | Description |
|--------|-----|------|-------------|
| 0x00 | [0] | EN | Watchdog enable |
| 0x00 | [1] | RESET_EN | Assert wdt_reset on timeout |
| 0x04 | [31:0] | COMPARE | Timeout threshold |
| 0x08 | — | PET | Write to pet (resets counter) |
| 0x14 | [0] | IRQ_EN | Interrupt enable |
| 0x18 | [0] | IRQ_FLAG | Interrupt flag (W1C) |

### 7.6 SPI — 0x2000_5000

| Offset | Bit | Name | Description |
|--------|-----|------|-------------|
| 0x00 | [0] | EN | SPI enable |
| 0x00 | [1] | CPOL | Clock polarity |
| 0x00 | [2] | CPHA | Clock phase |
| 0x00 | [4:3] | WORD_SIZE | 00=8-bit, 01=16-bit, 10=32-bit |
| 0x04 | [31:0] | DIV | Clock divider |
| 0x08 | [31:0] | DATA | Write = TX, Read = RX |
| 0x0C | [0] | BUSY | Transfer in progress (RO) |
| 0x0C | [1] | RX_READY | RX data available (RO) |
| 0x10 | [0] | DUAL_MODE | Enable dual/quad mode |
| 0x10 | [1] | LSB_FIRST | LSB-first bit order |
| 0x14 | [3:0] | CS_SELECT | Chip select mask |

### 7.7 I2C — 0x2000_6000

| Offset | Bit | Name | Description |
|--------|-----|------|-------------|
| 0x00 | [0] | EN | I2C enable |
| 0x04 | [31:0] | DIV | Clock divider |
| 0x08 | [7:0] | DATA | Transmit/receive data |
| 0x08 | [15:8] | ADDR | Slave address + R/W bit |
| 0x0C | [0] | ACK_ERR | No acknowledge received (RO) |
| 0x0C | [1] | BUSY | Transaction in progress (RO) |
| 0x0C | [2] | RX_READY | Receive data available (RO) |
| 0x0C | [3] | IRQ | Interrupt flag (RO) |

### 7.8 CAN — 0x2000_7000

| Offset | Bit | Name | Description |
|--------|-----|------|-------------|
| 0x00 | [0] | EN | CAN enable |
| 0x00 | [1] | LOOPBACK | Internal loopback mode |
| 0x04 | [15:0] | BRP | Baud rate prescaler |
| 0x04 | [19:16] | TSEG1 | Time segment 1 |
| 0x04 | [23:20] | TSEG2 | Time segment 2 |
| 0x08 | [28:0] | TX_ID | 29-bit extended TX identifier |
| 0x0C | [31:0] | TX_DATA0 | TX data bytes 0-3 |
| 0x10 | [31:0] | TX_DATA1 | TX data bytes 4-7 |
| 0x14 | [3:0] | TX_DLC | Data length code |
| 0x18 | — | TX_REQ | Write 1 to start transmission |
| 0x1C | [28:0] | RX_ID | 29-bit extended RX identifier (RO) |
| 0x20 | [31:0] | RX_DATA0 | RX data bytes 0-3 (RO) |
| 0x24 | [31:0] | RX_DATA1 | RX data bytes 4-7 (RO) |
| 0x28 | [0] | IRQ_FLAG | Interrupt flag (W1C) |
| 0x2C | [0] | IRQ_EN | Interrupt enable |
| 0x30 | [0] | BUS_OFF | Bus-off status (RO) |
| 0x30 | [1] | TX_BUSY | Transmission in progress (RO) |
| 0x30 | [2] | RX_READY | Receive data available (RO) |

### 7.9 DMA — 0x2000_8000

| Offset | Bit | Name | Description |
|--------|-----|------|-------------|
| 0x00 | [31:0] | SRC_ADDR | Source address |
| 0x04 | [31:0] | DST_ADDR | Destination address |
| 0x08 | [15:0] | WORD_COUNT | Number of words to transfer |
| 0x0C | [0] | START | Write 1 to start transfer |
| 0x0C | [1] | SRC_INC | Increment source address |
| 0x0C | [2] | DST_INC | Increment destination address |
| 0x0C | [3] | IRQ_EN | Interrupt on completion |
| 0x10 | [0] | BUSY | Transfer in progress (RO) |
| 0x10 | [1] | DONE | Transfer complete (W1C) |
| 0x10 | [2] | ERR | Transfer error (W1C) |
| 0x14 | [0] | IRQ_EN | Interrupt enable |

### 7.10 PWM Advanced — 0x2000_9000

**Per-channel registers (ch0: offset 0x00, ch1: 0x10, ch2: 0x20, ch3: 0x30):**

| Offset (ch0) | Bit | Name | Description |
|-------------|-----|------|-------------|
| 0x00 | [0] | CH_EN | Channel enable |
| 0x00 | [1] | POLARITY | Output polarity invert |
| 0x00 | [2] | CENTER_ALIGN | Center-aligned mode |
| 0x04 | [15:0] | PERIOD | PWM period |
| 0x08 | [15:0] | DUTY | PWM duty value |
| 0x0C | [15:0] | COUNTER | Current counter (RO) |

**Global registers:**

| Offset | Bit | Name | Description |
|--------|-----|------|-------------|
| 0x40 | [0] | GLOBAL_EN | Enable all channels |
| 0x40 | [1] | FAULT_CLEAR | Clear fault latch |
| 0x44 | [3:0] | IRQ_ENABLE | Per-channel IRQ enable |
| 0x48 | [3:0] | IRQ_FLAG | Per-channel IRQ flags (W1C) |

### 7.11 RTC — 0x2000_A000

| Offset | Bits | Field | Range | Description |
|--------|------|-------|-------|-------------|
| 0x00 | [7:0] | SEC | 0-59 | Seconds |
| 0x04 | [7:0] | MIN | 0-59 | Minutes |
| 0x08 | [7:0] | HOUR | 0-23 | Hours |
| 0x0C | [7:0] | DAY | 1-31 | Day of month |
| 0x10 | [7:0] | MONTH | 1-12 | Month |
| 0x14 | [31:0] | YEAR | — | Year (full 32-bit) |
| 0x18 | [7:0] | ALARM_SEC | 0-59 | Alarm seconds match |
| 0x18 | [15:8] | ALARM_MIN | 0-59 | Alarm minutes match |
| 0x18 | [23:16] | ALARM_HOUR | 0-23 | Alarm hours match |
| 0x1C | [0] | EN | — | RTC enable |
| 0x1C | [1] | ALARM_EN | — | Alarm enable |
| 0x1C | [2] | IRQ_EN | — | Interrupt enable |
| 0x1C | [8] | ALARM_IRQ | — | Alarm interrupt status (W1C) |
| 0x1C | [9] | OVF_IRQ | — | 1 Hz tick interrupt status (W1C) |

### 7.12 Debug — 0x2000_B000

| Offset | Bit | Name | Description |
|--------|-----|------|-------------|
| 0x00 | [0] | HALT | Halt CPU |
| 0x00 | [1] | RESET | Reset CPU |
| 0x00 | [2] | EXT_IRQ | Assert external IRQ |
| 0x04 | [0] | HALTED | CPU halted (RO) |
| 0x04 | [8] | BP_ACTIVE | Breakpoint active (RO) |
| 0x08-0x14 | [31:0] | BP_ADDR | Breakpoint address registers |
| 0x18 | [31:0] | BP_CTRL | Breakpoint control |
| 0x40-0xBC | [31:0] | MAILBOX | 32-word mailbox |

---

## 8. JTAG Debug

### 8.1 JTAG Interface

| Signal | Direction | Description |
|--------|-----------|-------------|
| jtag_tck | Input | Test clock |
| jtag_tms | Input | Test mode select |
| jtag_tdi | Input | Test data in |
| jtag_tdo | Output | Test data out |
| jtag_trst | Input | Test reset (active-low) |

### 8.2 TAP Controller

The JTAG TAP controller implements IEEE 1149.1 with 4 instructions:

| Instruction | Code | DR Size | Function |
|------------|------|---------|----------|
| IDCODE | 0x01 | 32 bits | Read device ID |
| DEBUG | 0x10 | 40 bits | Debug module access |
| SAMPLE | 0x02 | 1 bit | Bypass |
| BYPASS | 0x1F | 1 bit | Bypass |

**IDCODE value:** 0x00000001

### 8.3 DEBUG Instruction Protocol

The 40-bit DEBUG data register format:
```
Bit 39      — cmd_valid (1 = new command)
Bit 38      — r_w (0 = write, 1 = read)
Bits 37:32  — addr[5:0] (debug register word address)
Bits 31:0   — data[31:0] (write data or read result)
```

**Write sequence:**
1. Select-IR → Shift-IR (0x10) → Update-IR → Run-Test/Idle
2. Select-DR → Capture-DR → Shift-DR (40 bits: {1, 0, addr, data}) → Update-DR
3. Wait 10 TCK cycles for CDC propagation

**Read sequence (2 scans):**
1. Select-IR → Shift-IR (0x10) → Update-IR → Run-Test/Idle
2. Scan 1: Shift-DR (40 bits: {1, 1, addr, 0}) → Update-DR (command issued)
3. Wait 10 TCK cycles for CDC propagation + read data
4. Scan 2: Shift-DR (40 bits: {0, 0, 0, 0}) → capture and read out previous result on TDO

### 8.4 OpenOCD Configuration

An OpenOCD configuration file is provided at `fpga/openocd_rideprotect.cfg`:

```tcl
interface ftdi
ftdi_vid_pid 0x0403 0x6010
ftdi_layout_init 0x0098 0x008b
adapter_khz 1000
transport select jtag

jtag newtap rideprotect tap -irlen 5 -expected-id 0x00000001
target create rideprotect.tap jtag -chain-position rideprotect.tap
```

Usage:
```bash
openocd -f fpga/openocd_rideprotect.cfg
```

### 8.5 JTAG Bit-Bang from Host (Python Example)

```python
import gpiod

# JTAG pins on a Raspberry Pi or FTDI
TCK = 11
TMS = 10
TDI = 9
TDO = 8

def jtag_tick():
    gpio.output(TCK, 1); time.sleep(0.000001)
    gpio.output(TCK, 0); time.sleep(0.000001)

def jtag_shift_bits(count, data):
    tdo = 0
    for i in range(count):
        tdo |= (gpio.input(TDO) << i)
        gpio.output(TMS, 1 if (i == count-1) else 0)
        gpio.output(TDI, (data >> i) & 1)
        jtag_tick()
    return tdo

# Reset TAP
for _ in range(6): gpio.output(TMS, 1); jtag_tick()
gpio.output(TMS, 0); jtag_tick()

# Read IDCODE
gpio.output(TMS, 1); jtag_tick()
gpio.output(TMS, 1); jtag_tick()
gpio.output(TMS, 0); jtag_tick()
jtag_shift_bits(5, 0b00001)  # IDCODE instruction
gpio.output(TMS, 1); jtag_tick()
gpio.output(TMS, 0); jtag_tick()
print(f"IDCODE = {jtag_shift_bits(32, 0):08X}")
```

---

## 9. Memory Map Reference

### 9.1 Complete Address Map

| Base | End | Region | Access |
|------|-----|--------|--------|
| 0x0000_0000 | 0x0000_1FFF | SRAM (8 KB) | R/W |
| 0x2000_0000 | 0x2000_0FFF | GPIO | R/W |
| 0x2000_1000 | 0x2000_1FFF | UART | R/W |
| 0x2000_2000 | 0x2000_2FFF | Timer | R/W |
| 0x2000_3000 | 0x2000_3FFF | INTC | R/W |
| 0x2000_4000 | 0x2000_4FFF | Watchdog | R/W |
| 0x2000_5000 | 0x2000_5FFF | SPI | R/W |
| 0x2000_6000 | 0x2000_6FFF | I2C | R/W |
| 0x2000_7000 | 0x2000_7FFF | CAN | R/W |
| 0x2000_8000 | 0x2000_8FFF | DMA | R/W |
| 0x2000_9000 | 0x2000_9FFF | PWM Adv | R/W |
| 0x2000_A000 | 0x2000_AFFF | RTC | R/W |
| 0x2000_B000 | 0x2000_BFFF | Debug | R/W |
| All other | — | Bus timeout | Returns 0 after 255 cycles |

### 9.2 Boot Sequence

1. CPU reset vector = 0x00000000
2. First instruction at 0x00000000 jumps to `_startup` label
3. Stack pointer initialized to 0x00001FFC (top of SRAM − 4)
4. mtvec set to `irq_handler` at 0x00000010
5. Machine external interrupts enabled (mie[11], mstatus.MIE)
6. `main()` called

### 9.3 Interrupt Vector Table

| Address | Function |
|---------|----------|
| 0x00000000 | Reset vector → _startup |
| 0x00000004 | (reserved) |
| 0x00000010 | mtvec → irq_handler |

---

## 10. Troubleshooting

### 10.1 Build Issues

**Problem: `riscv64-linux-gnu-gcc: command not found`**

Install the RISC-V toolchain:
```bash
# Ubuntu/Debian
sudo apt-get install gcc-riscv64-linux-gnu

# Arch Linux
sudo pacman -S riscv64-linux-gnu-gcc
```

**Problem: `verilator: command not found`**

Install Verilator 5:
```bash
# Ubuntu 24.04
sudo apt-get install verilator

# Ubuntu 22.04 (PPA)
sudo add-apt-repository ppa:ubuntu-toolchain-r/test
sudo apt-get install verilator
```

**Problem: `yosys: command not found`**

```bash
sudo apt-get install yosys
```

**Problem: `hexdump: command not found`**

```bash
sudo apt-get install bsdmainutils
# or
sudo apt-get install xxd  # alternative
```

### 10.2 Simulation Issues

**Problem: Simulation hangs or times out**

1. Check that `make firmware` completed successfully
2. Verify `build/firmware.hex` exists and is non-empty
3. Increase MAX_SIM_TIME in `sim/sim_main.cpp` (line ~176)
4. Check for infinite loops in firmware

**Problem: UART output is garbled**

1. Verify UART divisor matches baud rate (27 for 115200 @ 50 MHz)
2. Check clock frequency — simulation runs at 50 MHz
3. Ensure BIT_TIME constant in `sim_main.cpp` matches divisor:
   `const int BIT_TIME = 27 * 16;` (27 = divisor, 16 = oversampling)

**Problem: JTAG test fails**

1. Ensure `jtag_trst` is held low during TAP reset sequence
2. Check CDC propagation: need 10+ TCK cycles between command and read
3. Verify two-scan read protocol is followed correctly
4. Check that mailbox is not being modified by CPU simultaneously

**Problem: Waveform file not generated**

1. Verify `Verilated::traceEverOn(true)` is set in testbench
2. Check disk space in `build/` directory
3. FST file can be large (~100 MB for 5M cycles)

### 10.3 Firmware Issues

**Problem: Firmware doesn't fit in SRAM**

- Firmware max size: 8,192 bytes
- Current firmware: ~5,124 bytes
- Reduce code size: use `-Os` optimization, remove unused functions
- Enable compressed instructions: `-march=rv32imc_zicsr` (saves 15-20%)

**Problem: IRQ handler not firing**

1. Verify INTC_ENABLE has the correct source bit set
2. Verify mie CSR has bit 11 set (machine external interrupt)
3. Verify mstatus.MIE bit is set (bit 3)
4. Check peripheral-specific IRQ enable register
5. Check peripheral-specific IRQ flag is being set

**Problem: Watchdog reset fires unexpectedly**

1. Ensure WDT_PET is written before timeout
2. Check WDT_COMPARE value is appropriate for your timing
3. WDT is enabled by default after reset — configure it early

### 10.4 FPGA Issues

**Problem: Vivado fails during synthesis**

1. Verify Xilinx Vivado is installed and in PATH
2. Check that `SYNTHESIS` define is present (`-verilog_define SYNTHESIS`)
3. Ensure `build/firmware.hex` is copied to project root
4. Check for XDC constraint violations (unconstrained paths)

**Problem: Bitstream programs but no UART output**

1. Check UART baud rate match (115200 8-N-1)
2. Verify TX/RX pin names match the XDC constraints
3. Check FTDI USB driver is installed
4. Measure clock output on test pin (if available)

**Problem: Timing closure not met**

1. The current design achieves timing closure at 50 MHz (WNS=2.869ns). If adding peripherals degrades timing, try:
2. Reduce clock frequency by changing divider in soc_top.v
3. Add pipeline stages to long combinatorial paths
4. Use `report_timing` to identify critical paths

### 10.5 Common Error Messages

| Error | Likely Cause | Solution |
|-------|-------------|----------|
| `Error: export of x-bit` | Unused CPU output bits | Benign warning; can be ignored |
| `Warning: unconnected port` | CPU bus bits not used | Benign; CPU drives full 32-bit bus |
| `TIMESCALEMOD` | Missing timescale | Suppressed with `-Wno-TIMESCALEMOD` |
| `UNOPTFLAT` | Combinatorial loop | Check for latch inference |
| `Bus timeout` at 0x30000000 | Unmapped address access | Intentional test; returns 0 |

---

## 11. FAQ

**Q: What microcontroller does this SoC replace?**
A: The RideProtect-RV is designed as an open-source replacement for ARM Cortex-M4/M7-based RideProtect telematics SoCs.

**Q: Can I use this in a commercial product?**
A: All RTL is open-source (check individual file headers for license). VexRiscv is MIT-licensed. Third-party files may have different licenses.

**Q: Why Wishbone instead of AXI?**
A: Wishbone B4 is simpler to implement and debug, with sufficient performance for this class of SoC. AXI4 would add 3× more signals without meaningful benefit at 50 MHz.

**Q: How do I add a new peripheral?**
A: 1) Create a Wishbone B4 slave module 2) Add it to `wb_shared_bus.v` address decoder 3) Add to `soc_top.v` instantiation 4) Add to Makefile RTL_SRCS 5) Write firmware test 6) Add XDC constraints if it has external pins

**Q: What is the maximum clock frequency?**
A: Currently 50 MHz (limited by the toggle flip-flop divider + BUFG). With an MMCM/PLL, the Artix-7 can run the VexRiscv at 100+ MHz. The combinatorial paths in wb_debug (2,206 LCs) are the current limiting factor.

**Q: Can I run without Vivado?**
A: Yes. Simulation (`make sim`) and synthesis (`make syn`) work with open-source tools. Only bitstream generation requires Vivado.

**Q: How do I debug firmware?**
A: Two methods: 1) UART output (simplest) 2) JTAG debug via OpenOCD (with FTDI cable). The simulation testbench also provides waveform tracing.

**Q: What is the power consumption?**
A: Not yet measured. On Arty A7-35T, the entire SoC uses ~5,878 LCs — expect <100 mW typical.

**Q: Can I use this with FreeRTOS or Zephyr?**
A: The VexRiscv supports machine-mode interrupts and can run a simple RTOS. However, the current firmware is bare-metal (no scheduler, no heap). RTOS porting is future work.

**Q: Why are GPIO bits 31:8 unassigned?**
A: The Arty A7-35T has limited pinout (4 LEDs + 4 switches). The RTL supports all 32 bits identically — bits 8-31 are available for expansion on PMOD headers.

**Q: How is the SRAM initialized with firmware?**
A: In simulation, `sim_top.v` uses `$readmemh("firmware.hex", mem)`. In FPGA, the Vivado flow copies firmware.hex to the project root, and the initial block in `wb_sram.v` (under `ifdef SYNTHESIS`) reads it via `$readmemh` for BRAM initialization.

**Q: The bus timeout returns 0 for unmapped addresses — is this safe?**
A: Yes. The timeout counter (255 cycles) ensures the CPU never hangs on stray pointer accesses. Returning 0 is safe for reads; writes are silently dropped. For production, a bus error exception or MPU-based protection is recommended.
