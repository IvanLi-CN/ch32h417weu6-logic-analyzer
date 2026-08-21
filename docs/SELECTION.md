# CH32H417WEU6 Logic Analyzer Hardware Selection

## Scope

This document is the current hardware selection baseline for a compact
16-channel logic analyzer. It is based on the project requirements, not on
matching a reference board's component names or propagation-delay values.

## Current Choices

| Function | Chosen part or architecture | Status | Basis |
| --- | --- | --- | --- |
| Main MCU | `CH32H417WEU6` | Selected | QFN68 supports the required USB 3.0 capture path while materially reducing board area compared with QEU6. |
| Capture voltage domain | One DAC-controlled `VIO18` rail, 1.84 V to 3.16 V | Selected | It sets the probe threshold and the MCU-side buffer output voltage together. |
| Digital input buffer | `SN74LVC16244ADGGR` (TI) | Selected for prototype | 16 non-inverting channels, 1.65 V to 3.6 V supply, 5.5 V-tolerant inputs, outputs limited to `VCC`, and no bus-hold. |
| Three-channel buck | `EA3036CQBR` / EA3036C | Selected for prototype | Provides the existing compact three-buck architecture; final rail allocation must be captured in the new schematic. |
| EA3036C output inductor | `SHC252012-1R5M`, 1.5 uH | Selected for prototype | Correct 1.5 uH value for EA3036C, low DCR, and ample current rating for the specified regulator load. |
| Analog amplifier | `OPA2356` | Conditional | Use only when retaining the high-bandwidth analog capture path. |

## Electrical Basis

### Digital Input Path

The digital path is `probe -> buffer -> CH32H417 GPIO`. The buffer supply and
the capture GPIO domain both use `VIO18`; consequently the output side is not
a fixed 3.3 V or 5 V domain. Probe inputs support 1.8 V, 2.5 V, 3.3 V, and 5 V
logic standards. The project does not support 1.2 V probe logic.

The `SN74LVC16244ADGGR` selection meets the hard electrical conditions. It
still requires board validation at `VIO18 = 1.84 V`, `2.5 V`, and `3.16 V` with
the intended RC/ESD network and actual MCU input loading.

Sources: [TI SN74LVC16244A data sheet](https://www.ti.com/lit/ds/symlink/sn74lvc16244a.pdf),
[TI product page for the exact ordering code](https://www.ti.com/product/SN74LVC16244A/part-details/SN74LVC16244ADGGR)

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

## Open Design Decisions

- Freeze the three EA3036C output rails and their load budgets in the new
  schematic. A 1.2 V *power* rail, if needed by the MCU or another circuit, is
  not a 1.2 V probe-logic requirement.
- Freeze the per-channel probe protection and RC values after checking their
  effect on short pulses and input capacitance.
- Determine whether the analog capture path remains in the compact product.
- Verify `SN74LVC16244ADGGR` procurement with the actual supplier before BOM
  release; a marketplace search result is not a traceable supply guarantee.

## Required Prototype Validation

- Capture 1.8 V, 2.5 V, 3.3 V, and 5 V probe signals at each applicable
  `VIO18` setting.
- Measure short-pulse observability, all-channel relative skew, and ringing at
  the MCU side of the buffer at the 200 MS/s target.
- Check EA3036C output ripple, startup, thermal rise, and the three-rail total
  power budget at realistic loads.
- Confirm the inductor's lot-specific data sheet, footprint, and reflow
  profile before a production order.

## Supporting Documents

- [Project requirements](research/project-requirements.md)
- [MCU package comparison](research/ch32h417-package-comparison.md)
- [EA3036C inductor selection](research/ea3036-inductor.md)
- [Analog front-end choice](research/analog-front-end.md)
