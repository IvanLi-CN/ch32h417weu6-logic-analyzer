# Power Architecture Source Audit

## Closed Decisions

This audit uses manufacturer primary documentation only. It does not use a
reference project, its schematic, firmware, BOM, or layout as design evidence.

| Decision | Primary-source basis | Result |
| --- | --- | --- |
| Main fixed rail | WCH permits 3.15-3.45 V for high-speed peripheral use. | `VCC33 = 3.32 V` supplies `VDD33`, `VDD33A`, `VDDIO`, `VREFP`, and HD3SS3220 `VCC33`. |
| Core/USB3 rail | WCH specifies `VDDK`/`VDD12A = 1.17-1.27 V` and recommends external 1.2 V plus 10-50 mV. | One `1.218-V` EA3036C channel supplies both. |
| Capture-voltage architecture | Product requirement mandates external `VIO18`; WCH permits `VIO18 = 1.1-3.6 V` provided it does not exceed `VDDIO`. TI specifies SN74LVC16244A operation at 1.65-3.6 V with 5.5-V-tolerant inputs. | External `VIO18` remains a product requirement; this document does not prescribe its CH3 feedback values or control range. |
| Reset-time VIO18 control | WCH specifies the `XO` resistor range selecting the internal-LDO reset default and documents an LDO-off software mode, but does not publish external-supply handover circuitry or PA5 reset behavior. | No `XO` value or DAC/CH3 startup sequence is selected from these sources. |
| TPS25200 current limit | TI's worst-case equations with a 1% resistor give approximately 1.175-1.420 A for 75 kOhm. | Use common E24 `75 kOhm`; do not use 68 kOhm because its worst-case limit exceeds 1.5 A. Normal full-capture budget is `<= 1.10 A`. |
| HD3SS3220 enable | HD3 requires a VDD5-referenced high `ENn_CC`; TPS3896 has push-pull output while TPS3898 has an 18-V-rated open drain and the needed active-low-enable polarity. | Use `TPS3898ADRYT` plus a 12-kOhm VDD5 pull-up. |

## Important Boundaries

- The 1.2-V rail is MCU and USB3 core power. It does not introduce a 1.2-V
  probe-logic mode.
- `VIO18` is an external rail by product requirement. The currently available
  WCH sources do not prove an external/internal-LDO handover, so no such
  sequence is a frozen design conclusion.
- The CH32 combined `VDDIO` and `VIO18` supply-pin current limit remains 200
  mA absolute maximum. Buffer current belongs to CH3, not to the MCU pins.

## Verification Scope

The remaining source gap is an official WCH document defining external VIO18
operation. Until it exists, the schematic must retain its existing VIO18/CH3
implementation without modifications derived from an assumed handover. Other
first-board requirements are listed in [Power Architecture](power-architecture.md).

## Primary Sources

- [WCH CH32H417 data sheet](https://www.wch-ic.com/downloads/CH32H417DS0_PDF.html)
- [WCH CH32H417 SDK](https://github.com/openwch/ch32h417)
- [EA3036C data sheet](https://datasheet.lcsc.com/datasheet/pdf/d83e9c52d5fbdb916908193b63b8ae60.pdf?productCode=C570857)
- [TPS25200 data sheet](https://www.ti.com/lit/ds/symlink/tps25200.pdf)
- [HD3SS3220 data sheet](https://www.ti.com/lit/ds/symlink/hd3ss3220.pdf)
- [TPS389x data sheet](https://www.ti.com/lit/ds/symlink/tps3896.pdf)
- [SN74LVC16244A data sheet](https://www.ti.com/lit/ds/symlink/sn74lvc16244a.pdf)
