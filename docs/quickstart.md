# RideProtect-RV — Quick-Start Guide

**Goal:** From zero to blinking an LED in under 5 minutes.

---

## 1. Prerequisites

```bash
sudo apt-get install -y gcc-riscv64-linux-gnu verilator yosys bsdmainutils make
```

## 2. Build and Run

```bash
git clone <repo-url> rideprotect-rv
cd rideprotect-rv
make firmware          # 2 seconds
make sim               # 10 seconds → see test results
make syn               # 30 seconds → resource estimate
```

## 3. Write Custom Firmware

Edit `sw/boot/test_prog.c`. Replace `main()` with:

```c
int main(void) {
    GPIO_DIR = 0x0F;                 // LEDs as outputs
    UART_DIV = 27; UART_CTRL = 0x03; // 115200 baud
    uart_puts("Blinking!\n");
    while (1) {
        GPIO_DATA_OUT = 0x0F;        // LEDs on
        delay(500000);
        GPIO_DATA_OUT = 0x00;        // LEDs off
        delay(500000);
    }
}
```

```bash
make firmware && make sim
```

## 4. Deploy to FPGA

```bash
make pnr-arty         # Vivado P&R (timing MET: WNS=2.869ns)
# Program build/soc_top.bit via Vivado Hardware Manager
# Public docs: https://aiquanticinsights-commits.github.io/rideprotect-rv-docs/
screen /dev/ttyUSB1 115200  # Should show "Blinking!"
```

## 5. Key Addresses (RAM Map)

```
0x0000_0000  SRAM (8 KB code+data)
0x2000_0000  GPIO         0x2000_1000  UART
0x2000_2000  Timer        0x2000_3000  INTC
0x2000_4000  Watchdog     0x2000_5000  SPI
0x2000_6000  I2C          0x2000_7000  CAN
0x2000_8000  DMA          0x2000_9000  PWM Advanced
0x2000_A000  RTC          0x2000_B000  Debug/JTAG mailbox
```

## 6. Quick Make Targets

| Command | What |
|---------|------|
| `make firmware` | Build firmware |
| `make sim` | Build + run simulation |
| `make lint` | Verilator lint check |
| `make syn` | Yosys synthesis |
| `make clean` | Remove build artifacts |

## 7. Where to Find Help

| Document | File |
|----------|------|
| User Manual | `docs/RideProtect_RV_User_Manual.md` |
| Project Report | `docs/RideProtect_RV_Project_Report.md` |
| Firmware Guide | `docs/firmware_guide.md` |
| Integration Guide | `docs/integration_guide.md` |
| Setup Guide | `docs/setup_guide.md` |
| Register Reference | `docs/bus_map.md` |
| Architecture | `docs/architecture.md` |
