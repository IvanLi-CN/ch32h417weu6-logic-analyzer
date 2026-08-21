# CH32H417WEU6 and CH32H417QEU6 Hardware Comparison

## Shared Core Capability

Both devices use the same CH32H417 dual-core family architecture, memory size,
USB 2.0 High-Speed support, USB 3.0 SuperSpeed support, Ethernet MAC plus
10/100M PHY, CAN, and the majority of the timer and conversion peripherals.

## Package and Peripheral Differences

| Item | CH32H417WEU6 | CH32H417QEU6 |
| --- | --- | --- |
| Package | QFN68, 8 mm x 8 mm, 0.4 mm pitch | QFN128, 12.3 mm x 12.3 mm, 0.35 mm pitch |
| GPIO | 50 | 95 |
| DAC | DAC2 only | DAC1 and DAC2 |
| OPA groups | OPA1 and OPA3 | OPA1, OPA2, and OPA3 |
| USB Type-C / PD and USB FS / OTG | Not present | Present |
| SerDes / SDIO | Not present | Present |
| UHSIF / FMC capability | Restricted as noted in the data sheet | Full feature set |

The packages are not footprint-compatible and are not pin-compatible. The
WEU6 package cannot be treated as a mechanical substitution for QEU6.

## Project Decision

Select `CH32H417WEU6`. The smaller QFN68 package fits the compact-board goal,
while the project needs the shared USB 3.0 capture capability rather than the
QEU6-only Type-C/PD, USB FS/OTG, SerDes, SDIO, or expanded I/O features.

Before finalizing any function marked optional in the WCH data sheet, check the
actual chip lot marking: some lots do not provide GPHA, Ethernet, SerDes, or
CAN.

Source: [WCH CH32H417/H416/H415 data sheet](https://www.wch-ic.com/downloads/CH32H417DS0_PDF.html)
