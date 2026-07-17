# RideProtect-RV — Documentation Roadmap

**SoC Version:** v2.5.0 | **Document Version:** 1.0 | **Last Updated:** 2026-07-16

---

## 1. Document Inventory & Status

| # | Document | Audience | Source | Status | Last Updated | Notes |
|---|---|---|---|---|---|---|
| D01 | Product Brief (`product_brief.md`) | Customer / Architect | Markdown | ✅ Published | 2026-07-16 | 2–4 page marketing + technical summary |
| D02 | Chip Selection Guide (`chip_selection_guide.md`) | Customer / Architect | Markdown | ✅ Published | 2026-07-16 | Part-number scheme + ordering info |
| D03 | Architecture Whitepaper (`whitepaper_architecture.md`) | Customer / Architect | Markdown | ✅ Published | 2026-07-16 | Deep-dive into unique architectural innovations |
| D04 | Architecture (`architecture.md`) | Customer / Architect | Markdown + DOCX | ✅ Published | 2025-Q4 | SoC block diagram and bus topology |
| D05 | Bus Map (`bus_map.md`) | Customer / Architect | Markdown + DOCX | ✅ Published | 2025-Q4 | Full memory map with register offsets |
| D06 | Data Sheet (`datasheet.md`) | Hardware / Board | Markdown | ✅ Published | 2026-07-16 | Electrical & physical specifications |
| D07 | Hardware Design Guide (`hardware_design_guide.md`) | Hardware / Board | Markdown | ✅ Published | 2026-07-16 | Schematic checklist, power, layout guidelines |
| D08 | Reference Schematic (`reference_schematic.md`) | Hardware / Board | Markdown | ✅ Published | 2026-07-16 | System block diagram, pin connections, PCB guide |
| D09 | Integration Guide (`integration_guide.md`) | Hardware / Board | Markdown + DOCX | ✅ Published | 2025-Q4 | Adding/removing Wishbone peripherals |
| D10 | Power Architecture (`power_architecture.md`) | Hardware / Board | Markdown | ✅ Published | 2025-Q4 | Power distribution, clock gating, PMU |
| D11 | VexRiscv Migration Notes (`vexriscv_migration.md`) | Hardware / Board | Markdown + DOCX | ✅ Published | 2025-Q4 | PicoRV32 → VexRiscv migration record |
| D12 | User Manual / TRM (`RideProtect_RV_User_Manual.md`) | Embedded SW | Markdown + DOCX | ✅ Published | 2025-Q4 | Complete technical reference |
| D13 | Firmware Guide (`firmware_guide.md`) | Embedded SW | Markdown + DOCX | ✅ Published | 2025-Q4 | Firmware development reference |
| D14 | SDK & API Guide (`sdk_api_guide.md`) | Embedded SW | Markdown | ✅ Published | 2026-07-16 | Peripheral driver API reference |
| D15 | BSP & Porting Guide (`bsp_guide.md`) | Embedded SW | Markdown | ✅ Published | 2026-07-16 | Board support package, porting checklist |
| D16 | Toolchain & Debug Guide (`toolchain_debug_guide.md`) | Embedded SW | Markdown | ✅ Published | 2026-07-16 | Toolchain setup, JTAG/GDB debug, simulation |
| D17 | Quickstart (`quickstart.md`) | Embedded SW | Markdown + DOCX | ✅ Published | 2025-Q4 | Zero-to-blinky in 5 minutes |
| D18 | Setup Guide (`setup_guide.md`) | Embedded SW | Markdown + DOCX | ✅ Published | 2025-Q4 | Build environment setup |
| D19 | Project Report (`RideProtect_RV_Project_Report.md`) | Internal / Support | Markdown + DOCX | ✅ Published | 2025-Q4 | Comprehensive project report |
| D20 | Verification Plan (`verification_plan.md`) | Internal / Support | Markdown | ✅ Published | 2026-07-16 | Formal + simulation verification |
| D21 | Verification Summary (`VERIFICATION.md`) | Internal / Support | Markdown | ✅ Published | 2026-07-16 | One-page verification status |
| D22 | EV Features Plan (`ev_features_plan.md`) | Internal / Support | Markdown | ✅ Published | 2025-Q4 | EV-specific feature planning |
| D23 | Wireless Architecture (`rideprotect_wireless_architecture_plan.md`) | Internal / Support | Markdown | ✅ Published | 2025-Q4 | BLE helmet + mobile app architecture |
| D24 | Matrix Multiply Integration (`wb_matrix_mult_integration.md`) | Internal / Support | Markdown | ✅ Published | 2025-Q4 | Accelerator integration guide |
| D25 | v2.5 Implementation Plan (`v2.5_implementation_plan.md`) | Internal / Support | Markdown | ✅ Published | 2025-Q4 | Release implementation plan |
| D26 | Errata Sheet (`errata.md`) | Internal / Support | Markdown | ✅ Published | 2026-07-16 | Known issues & fixes |
| D27 | Documentation Roadmap (`documentation_roadmap.md`) | Internal / Support | Markdown | ✅ Published | 2026-07-16 | This document |
| D28 | README (`README.md`) | All | Markdown | ✅ Published | 2025-Q4 | Top-level project overview |
| D29 | README v2.5 (`README_v2.5.md`) | All | Markdown | ✅ Published | 2025-Q4 | v2.5 release highlights |
| D30 | Changelog (`CHANGELOG_v2.0_to_v2.5.md`) | All | Markdown | ✅ Published | 2025-Q4 | Changes from v2.0 to v2.5 |
| D31 | Project Status (`PROJECT_STATUS.md`) | All | Markdown | ✅ Published | 2025-Q4 | Current project status |

---

## 2. Coverage by Audience

### Customers & System Architects (8 docs)
| Doc | Priority | What It Covers |
|---|---|---|
| D01 Product Brief | ⭐ Essential | Marketing + technical overview |
| D02 Chip Selection Guide | ⭐ Essential | Part numbers + ordering |
| D03 Architecture Whitepaper | ⭐ Essential | Technical deep-dive |
| D04 Architecture | ⭐ Essential | Block diagram + bus topology |
| D05 Bus Map | ⭐ Essential | Memory map |
| D09 Integration Guide | ★ Nice-to-have | Peripheral customization |
| D12 User Manual / TRM | ⭐ Essential | Complete reference |
| D30 Changelog | ★ Nice-to-have | Version differences |

### Hardware & Board Designers (5 docs)
| Doc | Priority | What It Covers |
|---|---|---|
| D06 Data Sheet | ⭐ Essential | Electrical specs |
| D07 Hardware Design Guide | ⭐ Essential | Schematic checklist, layout |
| D08 Reference Schematic | ⭐ Essential | Pin connections, block diagram |
| D09 Integration Guide | ⭐ Essential | Peripheral add/remove |
| D10 Power Architecture | ★ Nice-to-have | Power distribution |

### Embedded Software Developers (8 docs)
| Doc | Priority | What It Covers |
|---|---|---|
| D12 User Manual / TRM | ⭐ Essential | Complete reference |
| D13 Firmware Guide | ⭐ Essential | Firmware development |
| D14 SDK & API Guide | ⭐ Essential | Driver API reference |
| D15 BSP & Porting Guide | ⭐ Essential | Board porting |
| D16 Toolchain & Debug Guide | ⭐ Essential | Build, JTAG, simulation |
| D17 Quickstart | ⭐ Essential | First-run guide |
| D18 Setup Guide | ⭐ Essential | Environment setup |
| D24 Matrix Multiply Integration | ★ Nice-to-have | Accelerator usage |

### Internal / Support Teams (10 docs)
| Doc | Priority | What It Covers |
|---|---|---|
| D19 Project Report | ★ Reference | Full project documentation |
| D20 Verification Plan | ⭐ Essential | Formal + simulation coverage |
| D21 Verification Summary | ⭐ Essential | Quick status reference |
| D22 EV Features Plan | ⭐ Essential | Feature planning |
| D23 Wireless Architecture | ★ Nice-to-have | BLE + app architecture |
| D24 Matrix Multiply Integration | ★ Nice-to-have | Accelerator doc |
| D25 v2.5 Implementation Plan | ★ Reference | Release plan |
| D26 Errata Sheet | ⭐ Essential | Known issues |
| D27 Documentation Roadmap | ⭐ Essential | This document |
| D31 Project Status | ★ Reference | Status overview |

---

## 3. Future Documentation Plans

### Planned for v3.0 (ASIC Migration)

| Document | Description | Priority |
|---|---|---|
| ASIC Data Sheet | Updated electrical specs for SKY130A (1.8V core, different I/O) | High |
| ASIC Application Note | Analog integration: PLL, ADC, I/O pad selection | High |
| Migration Guide | Soft core → hard core differences, timing changes | High |
| Power Estimation Guide | Dynamic/leakage power for ASIC | Medium |
| Reliability Report | ESD, latch-up, EM, IR drop analysis | Medium |

### Planned Enhancements (no version target)

| Document | Description | Priority |
|---|---|---|
| Tutorial — Motor Control | Step-by-step BLDC/PMSM motor control with RideProtect-RV | Medium |
| Tutorial — BMS Design | Building a BMS with BMS ADC, balancer, temp sensors | Medium |
| Tutorial — CAN Bootloader | Firmware update over CAN bus | Low |
| Functional Safety Manual | ISO 26262 ASIL decomposition strategy | Medium |
| Security White Paper | Immobilizer protocol, encryption, secure boot | Low |
| Application Note — Solar EV | Integration with MPPT and solar charging | Low |

---

## 4. Document Source Files

Documents are maintained in two formats:

- **Markdown (`.md`):** Primary format, version-controlled in Git
- **Word (`.docx`):** Secondary format, generated from Markdown via Pandoc

### Conversion Workflow

```bash
# Convert Markdown → DOCX
pandoc docs/sdk_api_guide.md -o docs/sdk_api_guide.docx \
    --reference-doc=tools/pandoc/template.docx
```

---

## 5. Document Conventions

| Convention | Meaning |
|---|---|
| ✅ Published | Finalized and included in v2.5.0 |
| 🚧 Draft | Under review or not yet finalized |
| 📝 Planned | Scheduled for a future release |
| Item | No priority / not started |

---

## 6. Revision History

| Rev | Date | Description |
|---|---|---|
| 1.0 | 2026-07-16 | Initial documentation roadmap for v2.5.0 |

---

*RideProtect-RV Documentation Roadmap v1.0*
