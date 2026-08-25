# HD3SS3220 I2C GPIO Allocation: Reference Evidence

## Scope and evidence boundary

This note records the existing CH32H417WEU6 occupancy used to select the two
HD3SS3220 I2C signals. The supplied reference establishes occupancy only; the
final alternate-function and voltage-domain decision below is separately based
on the WCH data sheet and reference manual.

The two supplied project pages were checked. [Project 1](https://oshwhub.com/aaahaley/project_xkzwqync) currently exposes only a generic public project page and no CH32-specific schematic, PCB, netlist, or source code that can support a pin-allocation claim. No occupancy is inferred from it. [Project 2](https://oshwhub.com/calvaria/project_slbqmtir) identifies itself as the CH32H417WEU6 QFN-68 port and links the Calvaria repository; its public description says the design follows the original.

The positive evidence below comes from Calvaria's fixed Git commit [`70a2e6844b0649b4ac07f692926f342da991395c`](https://github.com/Calvariaa/CH32H417_Logic_Analyzer_Mini/tree/70a2e6844b0649b4ac07f692926f342da991395c):

- The [README pin-change table](https://github.com/Calvariaa/CH32H417_Logic_Analyzer_Mini/blob/70a2e6844b0649b4ac07f692926f342da991395c/README.md#L24-L29) names the WEU6 changes for VIO18 DAC, HSADC, UART, and LEDs.
- The actual EasyEDA Pro archive is [逻辑分析仪_2026-06-23-14-14.epro2](https://github.com/Calvariaa/CH32H417_Logic_Analyzer_Mini/blob/70a2e6844b0649b4ac07f692926f342da991395c/%E9%80%BB%E8%BE%91%E5%88%86%E6%9E%90%E4%BB%AA_backup/%E9%80%BB%E8%BE%91%E5%88%86%E6%9E%90%E4%BB%AA_2026-06-23-14-14.epro2). Its embedded `PCB-mini0v1` document has U1 component ID `adb716e7639dca71`; its `PAD_NET` records bind each U1 package pad to the PCB net. Its embedded `CH32H417WEU6` symbol maps those package-pad numbers to the signal names below. This is a direct symbol-plus-netlist join, not a conclusion drawn from package position.
- Firmware corroborates active functions: [hardware startup](https://github.com/Calvariaa/CH32H417_Logic_Analyzer_Mini/blob/70a2e6844b0649b4ac07f692926f342da991395c/CH32H417_Logic_Analyzer_1v0/CH32H417_Logic_Analyzer_APP/Common/hardware.c#L42-L60), [DAC and HSADC initialization](https://github.com/Calvariaa/CH32H417_Logic_Analyzer_Mini/blob/70a2e6844b0649b4ac07f692926f342da991395c/CH32H417_Logic_Analyzer_1v0/CH32H417_Logic_Analyzer_APP/Common/ch32h417_logic.c#L172-L225), [LED GPIO initialization](https://github.com/Calvariaa/CH32H417_Logic_Analyzer_Mini/blob/70a2e6844b0649b4ac07f692926f342da991395c/CH32H417_Logic_Analyzer_1v0/CH32H417_Logic_Analyzer_APP/Common/ch32h417_logic.c#L312-L335), [debug UART selection](https://github.com/Calvariaa/CH32H417_Logic_Analyzer_Mini/blob/70a2e6844b0649b4ac07f692926f342da991395c/CH32H417_Logic_Analyzer_1v0/CH32H417_Logic_Analyzer_APP/Common/Debug/debug.h#L24-L38), and [USART6 TX setup](https://github.com/Calvariaa/CH32H417_Logic_Analyzer_Mini/blob/70a2e6844b0649b4ac07f692926f342da991395c/CH32H417_Logic_Analyzer_1v0/CH32H417_Logic_Analyzer_APP/Common/Debug/debug.c#L143-L181).

## Occupied CH32 connections

### USB3 and USB2/SWD

| U1 pad | WEU6 symbol signal | Actual PCB net | Allocation result |
| --- | --- | --- | --- |
| 1 | SSTXB | `USBSS_TXN` | USB3 SuperSpeed transmit pair; occupied. |
| 2 | SSTXA | `USBSS_TXP` | USB3 SuperSpeed transmit pair; occupied. |
| 4 | SSRXB | `USBSS_RXN` | USB3 SuperSpeed receive pair; occupied. |
| 5 | SSRXA | `USBSS_RXP` | USB3 SuperSpeed receive pair; occupied. |
| 67 | PB8 / USB2DP / SWCLK | `USBHS_P` | USB2 D+ is routed here. The same symbol signal advertises SWCLK, so it is not an available SWD-only or I2C net in this reference. |
| 68 | PB9 / USB2DM / SWDIO | `USBHS_N` | USB2 D- is routed here. The same symbol signal advertises SWDIO, so it is not an available SWD-only or I2C net in this reference. |

The firmware starts USBSS at boot ([`hardware.c` lines 58-60](https://github.com/Calvariaa/CH32H417_Logic_Analyzer_Mini/blob/70a2e6844b0649b4ac07f692926f342da991395c/CH32H417_Logic_Analyzer_1v0/CH32H417_Logic_Analyzer_APP/Common/hardware.c#L58-L60)); USBHS also has a device initialization path ([`ch32h417_usbhs_device.c` lines 195-218](https://github.com/Calvariaa/CH32H417_Logic_Analyzer_Mini/blob/70a2e6844b0649b4ac07f692926f342da991395c/CH32H417_Logic_Analyzer_1v0/CH32H417_Logic_Analyzer_APP/Common/ch32h417_usbhs_device.c#L195-L218)). The PCB net binding, rather than the firmware alone, is the evidence for the physical USB assignment.

The application disables SWJ/SWD during startup ([`hardware.c` lines 42-49](https://github.com/Calvariaa/CH32H417_Logic_Analyzer_Mini/blob/70a2e6844b0649b4ac07f692926f342da991395c/CH32H417_Logic_Analyzer_1v0/CH32H417_Logic_Analyzer_APP/Common/hardware.c#L42-L49)). Do not treat `PA13` and `PA14` as SWD by name alone: the actual reference PCB binds them to capture signals below.

### 16-channel capture and capture clock

The firmware requests 16-bit UHSIF with pin remap 0 and clock remap 1 ([`ch32h417_uhsif.c` lines 25-40](https://github.com/Calvariaa/CH32H417_Logic_Analyzer_Mini/blob/70a2e6844b0649b4ac07f692926f342da991395c/CH32H417_Logic_Analyzer_1v0/CH32H417_Logic_Analyzer_APP/Common/ch32h417_uhsif.c#L25-L40); [configuration constants](https://github.com/Calvariaa/CH32H417_Logic_Analyzer_Mini/blob/70a2e6844b0649b4ac07f692926f342da991395c/CH32H417_Logic_Analyzer_1v0/CH32H417_Logic_Analyzer_APP/Common/ch32h417_uhsif.h#L34-L45)). The actual PCB assigns every bus bit as follows.

| U1 pad | WEU6 symbol signal | Actual PCB net |
| --- | --- | --- |
| 15 | PC0 / ADC10 / HSADC0 | `P_CLK` |
| 33 | PD10 | `UFP0` |
| 34 | PD11 | `UFP1` |
| 35 | PD12 | `UFP2` |
| 36 | PD13 | `UFP3` |
| 37 | PD14 | `UFP4` |
| 38 | PD15 | `UFP5` |
| 39 | PF0 | `UFP6` |
| 40 | PF1 | `UFP7` |
| 41 | PF2 | `UFP8` |
| 42 | PC6 | `UFP9` |
| 43 | PC7 | `UFP10` |
| 44 | PC8 | `UFP11` |
| 45 | PC9 | `UFP12` |
| 46 | PA13 | `UFP13` |
| 47 | PA14 | `UFP14` |
| 48 | PA15 | `UFP15` |

`PC6` is additionally touched by the startup code's SWPMI bypass setting ([`hardware.c` lines 47-49](https://github.com/Calvariaa/CH32H417_Logic_Analyzer_Mini/blob/70a2e6844b0649b4ac07f692926f342da991395c/CH32H417_Logic_Analyzer_1v0/CH32H417_Logic_Analyzer_APP/Common/hardware.c#L47-L49)); irrespective of that register setting, the PCB proves it is already the `UFP9` capture line.

### HSADC, VIO18 control, UART, and GPIO indicators

| U1 pad | WEU6 symbol signal | Actual PCB net | Firmware/reference evidence |
| --- | --- | --- | --- |
| 17 | PC2 / ADC12 / HSADC2 | `OUT_ADC0` | Firmware initializes PC2 for `ADC_CHANNEL0` ([lines 215-220](https://github.com/Calvariaa/CH32H417_Logic_Analyzer_Mini/blob/70a2e6844b0649b4ac07f692926f342da991395c/CH32H417_Logic_Analyzer_1v0/CH32H417_Logic_Analyzer_APP/Common/ch32h417_logic.c#L215-L220)). |
| 18 | PC3 / ADC13 / HSADC3 | `OUT_ADC1` | Firmware initializes PC3 for `ADC_CHANNEL1` ([lines 221-225](https://github.com/Calvariaa/CH32H417_Logic_Analyzer_Mini/blob/70a2e6844b0649b4ac07f692926f342da991395c/CH32H417_Logic_Analyzer_1v0/CH32H417_Logic_Analyzer_APP/Common/ch32h417_logic.c#L221-L225)). |
| 20 | PA5 / ADC5 / OP1_O1 / DAC2 | `FB` | PA5 is DAC2 in firmware ([lines 172-190](https://github.com/Calvariaa/CH32H417_Logic_Analyzer_Mini/blob/70a2e6844b0649b4ac07f692926f342da991395c/CH32H417_Logic_Analyzer_1v0/CH32H417_Logic_Analyzer_APP/Common/ch32h417_logic.c#L172-L190)); the README identifies it as the VIO18-adjustment DAC ([lines 24-27](https://github.com/Calvariaa/CH32H417_Logic_Analyzer_Mini/blob/70a2e6844b0649b4ac07f692926f342da991395c/README.md#L24-L27)). |
| 21 | PB0 / ADC8 / OP1_P0 / CMP_P0 | `LED1` | GPIO output for the USB-link indicator ([lines 312-335](https://github.com/Calvariaa/CH32H417_Logic_Analyzer_Mini/blob/70a2e6844b0649b4ac07f692926f342da991395c/CH32H417_Logic_Analyzer_1v0/CH32H417_Logic_Analyzer_APP/Common/ch32h417_logic.c#L312-L335)). |
| 22 | PB1 / ADC9 / OP1_N0 / CMP_N0 | `LED2` | GPIO output for the USB-transfer/firmware-run indicator ([lines 312-335](https://github.com/Calvariaa/CH32H417_Logic_Analyzer_Mini/blob/70a2e6844b0649b4ac07f692926f342da991395c/CH32H417_Logic_Analyzer_1v0/CH32H417_Logic_Analyzer_APP/Common/ch32h417_logic.c#L312-L335)). |
| 28, 66 | VIO18 | `VIO18` | Both dedicated VIO18 supply pins are on the adjustable rail. They are supply pins, not GPIO. |
| 53 | PD1 | `TX1` | The selected debug configuration is USART6 TX on PD1; it is TX-only (`USART_Mode_Tx`) ([`debug.h` lines 24-38](https://github.com/Calvariaa/CH32H417_Logic_Analyzer_Mini/blob/70a2e6844b0649b4ac07f692926f342da991395c/CH32H417_Logic_Analyzer_1v0/CH32H417_Logic_Analyzer_APP/Common/Debug/debug.h#L24-L38), [`debug.c` lines 143-181](https://github.com/Calvariaa/CH32H417_Logic_Analyzer_Mini/blob/70a2e6844b0649b4ac07f692926f342da991395c/CH32H417_Logic_Analyzer_1v0/CH32H417_Logic_Analyzer_APP/Common/Debug/debug.c#L143-L181)). |

The reference has no separate MCU `RX` PCB net and no RX initialization in this debug configuration. That is an evidence of the current reference wiring only; it is not permission to reuse an arbitrary other pad.

## I2C and remaining unconnected GPIO evidence

No application I2C bus is present in the reference. A literal search for `I2C`, `IIC`, `SDA`, and `SCL` across the tracked application source at the fixed commit returns no match, and the same search across the EasyEDA archive returns no named I2C net. The repository contains compiled vendor peripheral objects named `ch32h417_i2c.o`; those objects do not demonstrate an application I2C instance or a routed SDA/SCL pair and are not counted as use evidence.

The following MCU GPIO-labelled pads have an empty `PAD_NET` in the actual `PCB-mini0v1` U1 record: `PC1`, `PE13`, `PE14`, `PE15`, `PB10`, `PB11`, `PB12`, `PB13`, `PB14`, `PC10`, `PC11`, `PC12`, `PD0`, `PD2`, `PD3`, `PD4`, `PD5`, `PD6`, `PD7`, `PF3`, `PF4`, `PF5`, `PE0`, `PE1`, and `PF14`.

This means only that the reference PCB does not connect those pads. It does **not** establish their I2C alternate-function capability, boot/debug constraints, electrical compatibility with the chosen HD3SS3220 implementation, or their availability in the new compressed layout. The Ethernet PHY pins `MDIRN`, `MDIRP`, `MDITN`, and `MDITP` are also unconnected in the reference PCB, but they are dedicated PHY signals rather than candidates for an I2C GPIO allocation.

## Final allocation

Reference evidence proves that USB3, USB2/SWD, all 16 UHSIF capture bits plus
`P_CLK`, both HSADC inputs, PA5 DAC/VIO18 control, PD1 UART TX, and PB0/PB1
LEDs are already occupied. It also proves that the reference contains no
existing I2C bus.

The selected pair is:

| HD3SS3220 I2C signal | WEU6 QFN68 pad / GPIO | Peripheral function | Project connection |
| --- | --- | --- | --- |
| SCL | `26 / PB10` | `I2C2_SCL`, AF4 | `PB10 -> HD3SS3220.SCL/OUT2`; `4.7 kOhm` pull-up to `VIO18` |
| SDA | `27 / PB11` | `I2C2_SDA`, AF4 | `PB11 -> HD3SS3220.SDA/OUT1`; `4.7 kOhm` pull-up to `VIO18` |

The WCH alternate-function table exposes `I2C2_SCL/SDA` on PB10/PB11. The
reference manual classifies both as high-speed dynamic-`VIO18` pins, therefore
they cannot be pulled directly to fixed `VCC33`. Pull both lines up to `VIO18`
with 4.7 kOhm resistors instead. The HD3SS3220 specifies 1.65 V to 3.6 V for
its I2C-pin supply, which covers every project `VIO18` setting. I2C2 runs at
100 kHz; HD3SS3220 has 7-bit address `0x47` when `ADDR` is tied to GND.

This is a schematic decision made from the MCU alternate-function and
voltage-domain documentation plus the documented project occupancy. It is not
inferred from unconnected reference pads.

Sources: [WCH CH32H417 data sheet](https://www.wch-ic.com/downloads/CH32H417DS0_PDF.html),
[WCH CH32H417 EVT/reference-manual bundle](https://www.wch.cn/downloads/CH32H417EVT_ZIP.html),
[TI HD3SS3220 data sheet](https://www.ti.com/lit/ds/symlink/hd3ss3220.pdf).
