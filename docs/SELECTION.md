# CH32H417WEU6 Logic Analyzer Hardware Selection

## Scope

This document is the current hardware selection baseline for a compact
16-channel logic analyzer. It is based on the project requirements, not on
matching a reference board's component names or propagation-delay values.

## Current Choices

| Function | Chosen part or architecture | Status | Basis |
| --- | --- | --- | --- |
| Main MCU | `CH32H417WEU6` | Selected | QFN68 supports the required USB 3.0 capture path while materially reducing board area compared with QEU6. |
| USB-C CC controller and SuperSpeed mux | `HD3SS3220`, I2C-managed UFP/Sink | Selected | One IC provides fixed device-side CC handling, automatic cable orientation, and a 10 Gbps-capable mux for the required 5 Gbps link. |
| Capture voltage domain | Existing schematic-defined external `VIO18` shared by MCU and buffer | Product requirement | The product requires external `VIO18` from EA3036C CH2; no new CH2 feedback, `XO`, or handover circuit is selected by this document. |
| Digital input buffer | `SN74LVC16244ADGGR` (TI) | Selected for prototype | 16 non-inverting channels, 1.65 V to 3.6 V supply, 5.5 V-tolerant inputs, outputs limited to `VCC`, and no bus-hold. |
| Three-channel buck | `EA3036CQBR` / EA3036C | Selected for prototype | CH1 is the fixed 3.32-V `3V3` rail, CH2 is the DAC-adjustable external `VIO18` rail, and CH3 is the fixed 1.218-V `1V2` rail. |
| EA3036C output inductor | `SHC252012-1R5M`, 1.5 uH | Selected for prototype | Correct 1.5 uH value for EA3036C, low DCR, and ample current rating for the specified regulator load. |
| HD3SS3220 power-good control | `TPS3898ADRYT` | Selected for prototype | Its open drain releases `ENn_CC` to the VDD5 pull-up until the fixed 3.3 V mux rail is valid. |
| Analog amplifier | `OPA2356` | Conditional | Use only when retaining the high-bandwidth analog capture path. |
| Future hardware SPI expansion | `SPI1`: PF4/NSS, PF5/SCK, PD7/MOSI, PF3/MISO | Reserved | A complete AF5 full-duplex port remains free, with adjacent PD6 reserved as optional expansion GPIO. It uses the selected external `VIO18` logic domain. |

## Electrical Basis

### Digital Input Path

The digital path is `probe -> buffer -> CH32H417 GPIO`. The buffer and capture
GPIO domain share the product-required external `VIO18` rail. Probe inputs
support 1.8 V, 2.5 V, 3.3 V, and 5 V logic standards. The project does not
support 1.2 V probe logic. This document does not prescribe changes to the
existing CH2 feedback or DAC connection.

The `SN74LVC16244ADGGR` selection meets the hard electrical conditions. It
requires board validation at each selected external `VIO18` voltage with the
intended RC/ESD network and actual MCU input loading.

All four active-low `OE#` inputs are tied directly to `GND`. The buffer is
therefore always enabled when its `VIO18` supply is valid; no `CAPTURE_OE_N`
net, PB2 assignment, pull-up, or firmware output gate is part of the design.
TI `SN74LVC16244A` Rev. C, page 11, Section 9.4, Table 3 specifies that
`OE = L` makes `Y` follow `A`, while `OE = H` makes `Y` high impedance.

When the optional HSADC path is retained, `VDD33A` and `VDDIO` remain on the
fixed 3.32-V rail. WEU6 has no external `VREFP` pin; its internal `VREFP`
equals `VDD33A`. The selected capture `VIO18` rail remains 1.80 V, 2.50 V, or
3.00 V and does not set the HSADC conversion range. See [HSADC and VIO18
operating constraints](research/ch32h417weu6-hsadc-vio18-constraints.md).

Sources: [TI SN74LVC16244A data sheet](https://www.ti.com/lit/ds/symlink/sn74lvc16244a.pdf),
[TI product page for the exact ordering code](https://www.ti.com/product/SN74LVC16244A/part-details/SN74LVC16244ADGGR)

### USB-C Device Port

The USB-C port is a fixed UFP/Sink device port. `HD3SS3220` is used in I2C
mode with `PORT` and `ADDR` tied to GND; its 7-bit I2C address is `0x47`. It
internally presents the UFP CC pull-downs, so external 5.1 kOhm CC pull-downs
must not be fitted. I2C is controller management only; it does not add USB PD
or require the WEU6 USBPD peripheral.

Allocate WEU6 QFN68 pad `26/PB10` as `I2C2_SCL` (AF4) and pad `27/PB11` as
`I2C2_SDA` (AF4). Connect PB10 directly to `SCL/OUT2` and PB11 directly to
`SDA/OUT1`; fit `4.7 kOhm`, 5%, `0402` pull-ups from both lines to external
`VIO18`. The HD3SS3220 I2C-pin supply range is 1.65 V to 3.6 V; the existing
external rail must remain in that range whenever I2C is active. Run I2C2 at
100 kHz.

The project-specific peripheral is `VBUS_RAW -> 910 kOhm -> VBUS_DET`,
`TPS25200 OUT -> VDD5`, and `EA3036C 3.32 V -> VCC33`, with a 200 kOhm
pull-up from open-drain `DIR` to fixed 3.3 V. `EA3036C.EN1` connects directly
to `VDD5`; `EN2` and `EN3` each use the selected `3V3`-referenced
`10 kOhm / 100 nF` RC network. TPS25200 `FAULT` remains NC.

This local passive EA enable policy is valid for the CH32H417 rails, which have
no mandatory three-rail sequence. The matching EN2/EN3 RC values intentionally
do not impose a CH2-to-CH3 order, and no MCU, PGOOD, or cascade is selected.
It does not satisfy the separate HD3SS3220 power-on requirement. `ENn_CC` is therefore controlled by `TPS3898ADRYT`,
which monitors `VCC33` and releases this active-low input to the VDD5 pull-up
during ramp. The
exact divider, delay capacitor, and release test are fixed in
[Power Architecture](research/power-architecture.md). `HD3SS3220` is not a
USB PD protocol controller, which matches the current 5 V host-powered
analyzer port.

The part supports USB 3.1 Gen 1/Gen 2 signaling up to 10 Gbps. The board link
remains limited by the CH32H417 USB 3.2 Gen 1 interface. See
[HD3SS3220 fixed-UFP peripheral review](research/hd3ss3220-ufp-peripheral-review.md)
for the complete connection constraints and BOM.

Sources: [TI HD3SS3220 data sheet](https://www.ti.com/lit/ds/symlink/hd3ss3220.pdf)
and [WCH CH32H417 data sheet](https://www.wch-ic.com/downloads/CH32H417DS0_PDF.html)

### Future Hardware SPI Expansion

Reserve a full `SPI1` port for future hardware expansion: `PF4` as NSS/CS0,
`PF5` as SCK, `PD7` as MOSI, and `PF3` as MISO, all in AF5. Reserve adjacent
`PD6` as `EXP_GPIO0` for a peripheral interrupt, reset, or extra software
chip-select. These are all `VIO18` pins, so the expansion interface uses the
currently selected external capture-I/O domain; it is not a fixed 3.3 V or 5 V
port.

The reservation requires only PCB access to `GND`, `VIO18`, the four SPI nets,
and `EXP_GPIO0`. Do not fit a connector or additional components until a
specific expansion device and its power budget are selected. The MCU SPI limit
is 75 MHz, subject to board-level validation for the actual future topology.
See [hardware SPI expansion reservation](research/spi-expansion-allocation.md)
for the pin, electrical, and firmware constraints.

### Power and Inductor

EA3036C is a three-channel synchronous buck regulator with 1.5 MHz nominal
switching and 1 A continuous load per channel. Its application guidance allows
1.0 uH to 2.2 uH and uses 1.5 uH in the typical circuit. The selected
`SHC252012-1R5M` is a 2.5 mm x 2.0 mm x 1.2 mm package class, not an 0806
part. Treat it as a `252012` / 1008 footprint part.

The supplied table lists 53 mOhm maximum DCR, 3.2 A heat-rating current, and
3.6 A saturation current for this exact inductor. Its current-column labels
are internally inconsistent, so the lower shown values are used as the
conservative design input. Before production release, obtain the supplier's
data sheet and confirm its `Isat` test definition.

Sources: [EA3036C data sheet](https://datasheet.lcsc.com/datasheet/pdf/d83e9c52d5fbdb916908193b63b8ae60.pdf?productCode=C570857),
[SHC252012 electrical evidence](evidence/shc252012-electrical-characteristics.png)

The power rail, current budget, TPS25200 peripheral values, and HD3SS3220
power-good circuit are selected together in
[Power Architecture](research/power-architecture.md). That document is the
source of truth for the non-`VIO18` rails. The external `VIO18` circuit remains
the existing schematic implementation; see the evidence boundary there.

## Open Design Decisions

- Freeze the per-channel probe protection and RC values after checking their
  effect on short pulses and input capacitance.
- Determine whether the analog capture path remains in the compact product.
- Verify `SN74LVC16244ADGGR` procurement with the actual supplier before BOM
  release; a marketplace search result is not a traceable supply guarantee.

## Required Prototype Validation

- Validate the existing external `VIO18` implementation against a WCH source
  that explicitly documents external supply operation before changing its
  feedback network or startup behavior.
- Measure short-pulse observability, all-channel relative skew, and ringing at
  the MCU side of the buffer at the 200 MS/s target.
- With HSADC enabled, verify PC2/PC3 acquisition and its `0 V` to 3.32-V
  conversion range at each selected `VIO18` mode.
- Check EA3036C output ripple, startup, thermal rise, and the selected
  3.32-V `3V3` / external-`VIO18` / 1.218-V `1V2` rail budget at realistic loads.
- Confirm the TPS3898-controlled `ENn_CC` is high during every supply ramp and
  remains high for at least 2 ms after both HD3SS3220 supply rails are valid.
- Verify `VDD5` remains above 4.5 V at HD3SS3220 and that full-rate capture is
  not enabled on a Default-current Type-C source.
- Enumerate USB SuperSpeed in both Type-C plug orientations and validate
  UFP/Sink attach and detach behavior.
- Confirm the inductor's lot-specific data sheet, footprint, and reflow
  profile before a production order.

## Supporting Documents

- [Project requirements](research/project-requirements.md)
- [MCU package comparison](research/ch32h417-package-comparison.md)
- [EA3036C inductor selection](research/ea3036-inductor.md)
- [Power architecture](research/power-architecture.md)
- [HD3SS3220 fixed-UFP peripheral review](research/hd3ss3220-ufp-peripheral-review.md)
- [Hardware SPI expansion reservation](research/spi-expansion-allocation.md)
- [Analog front-end choice](research/analog-front-end.md)
