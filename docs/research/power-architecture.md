# Power Architecture

## Selected First-Board Topology

This topology is based on confirmed product requirements and WCH, TI, and
Everanalog primary documentation only. It does not use a reference-project
implementation as design evidence.

```text
USB-C VBUS
    |
VBUS_RAW -- TPS25200DRVR -- VDD5 ------------------------ HD3SS3220.VDD5
                            |                              TPS3898ADRYT.VCC
                            +-- EA3036CQBR VIN1/2/3, VCC

EA3036C CH1: 3V3 (3.32-V target) -> VDD33, VDD33A, VDDIO, HD3SS3220.VCC33
EA3036C CH2: 1V8 / VIO18 (DAC-adjustable) -> CH32 VIO18, SN74LVC16244A.VCC, I2C pull-ups
EA3036C CH3: 1V2 (1.218-V target) -> VDDK, VDD12A
```

`VIO18` is an externally supplied, DAC-adjustable common capture-voltage
domain from CH2. The current PCB names this net `1V8`; that name denotes the
default mode, not a restriction on the selected 1.80-V, 2.50-V, or 3.00-V
target. It directly powers the CH32H417 `VIO18` pins, the 16-channel capture
buffer, and the HD3SS3220 I2C pull-ups. It is never connected to `VDDIO`.
The 3.00-V mode is the 3.3-V and 5-V logic mode: SN74LVC16244A has a fixed
2.0-V `VIH(min)` when `VCC >= 2.7 V`, so 3.00 V is sufficient and preserves
margin below the fixed 3.32-V `VDDIO` rail.

```text
VDD33 >= VDD33A >= VDDIO >= VIO18
VDD33A >= VREFP
```

WEU6 does not expose `VREFP` or `VSSA`. Its internal `VREFP` is equal to
`VDD33A`, so `VREFP` is not a separate PCB rail or decoupling location on this
package.

## Rail Values and Passives

| Rail | Current EA3036C channel | Nominal output | Allocated load | Loads |
| --- | --- | ---: | ---: | --- |
| `3V3` / `VCC33` | CH1 | 3.32 V | 0.50 A | CH32 `VDD33`, `VDD33A`, `VDDIO`; HD3 `VCC33` |
| `1V8` / `VIO18` | CH2, including the existing DAC2 feedback connection | Product-defined | 0.60 A | CH32 `VIO18`, SN74LVC16244A, I2C pull-ups |
| `1V2` / `VDD12` | CH3 | 1.218 V | 0.45 A | CH32 `VDDK`, `VDD12A` |

Fit the following EA3036C network:

| Location | Fitted value |
| --- | --- |
| Each `VINx` | `10 uF`, X5R/X7R ceramic, immediately to local power ground |
| `VCC` | `1 uF`, X7R ceramic, immediately to ground |
| Each output | `22 uF`, X5R/X7R; effective capacitance must remain at least 10 uF at bias |
| Each `VOUTx -> FBx` | `220 pF`, C0G/X7R, in parallel with the upper feedback resistor |
| Each channel inductor | `SHC252012-1R5M`, 1.5 uH |
| `EN1` | Directly to `VDD5` |
| `EN2` | `3V3 -> R5 (10 kOhm) -> EN2`, with `C15 (100 nF)` from EN2 to GND |
| `EN3` | `3V3 -> R6 (10 kOhm) -> EN3`, with `C16 (100 nF)` from EN3 to GND |

The selected enable topology is therefore:

```text
VDD5 ------------------------------ EN1
3V3 --- R5 10 kOhm ---+------------ EN2
                       |
                     C15 100 nF
                       |
                      GND
3V3 --- R6 10 kOhm ---+------------ EN3
                       |
                     C16 100 nF
                       |
                      GND
```

Using `3V3`, rather than `VDD5`, as the source for both delayed enables makes
CH2 and CH3 wait for CH1's rail to begin rising. Equal RC networks intentionally
do not impose a CH2-to-CH3 order. No MCU GPIO, power-good signal, or CH2-to-EN3
cascade is part of this design.

The nominal RC time constant of each delayed enable is `1 ms`. EA3036C
Ver. 1.4, page 11, Section "Enable Control" says that each channel's startup
time can be programmed with an independent EN RC and illustrates `100 kOhm`
with `10 nF` to `100 nF`. Its page 5 electrical table gives only
`V_EN-L <= 0.4 V` and `V_EN-H >= 1.4 V`; it does not give an EN-threshold upper
bound, EN input timing parameters, or a guaranteed EN-to-output-stable time.
Consequently, `10 kOhm x 100 nF = 1 ms` records the nominal circuit constant,
not an official guarantee that `3V3` is fully settled before CH2 or CH3 turns
on. WCH specifies no mandatory three-rail order, so no additional sequence
circuit is selected.

## External VIO18 Evidence Boundary

External `VIO18` is a product requirement. WCH-RM v1.7 proves PA5 resets as
a floating GPIO input and DAC2 resets disabled, but the WCH data sheet and
reference manual only document external `VIO18` when it is fixed shorted to
`VDDIO`. They do not publish an independent-external-LDO application circuit
or a handover procedure for this DAC-adjustable rail. Therefore this document
does not prescribe an `XO` resistor, a CH2 feedback network, or a DAC startup
sequence. Keep the existing `VIO18` net name and CH2 feedback circuit unless a
separate official WCH source establishes a required change. See [the official
analysis](ch32h417-external-vio18-dac-control-official-analysis.md).

## CH32H417 Supply Rules

- `VCC33` satisfies WCH's 3.15-3.45-V high-speed-peripheral recommendation.
- `VDD12` is within the 1.17-1.27-V range. Its 1.218-V target leaves 18 mV
  above 1.2 V for copper drop.
- `VIO18` is externally supplied by product requirement and must not exceed
  `VDDIO`. WCH documents only a fixed `VDDIO`--`VIO18` short, not this
  independent external control range or startup behavior.
- With HSADC enabled, keep `VDD33A = VDDIO = 3.32 V`. WEU6's internal
  `VREFP` then also equals 3.32 V, while `VIO18` remains at the selected
  1.80-V, 2.50-V, or 3.00-V capture voltage. The HSADC conversion range stays
  at `0 V` to `VDDIO`; it does not follow `VIO18`. See [HSADC and VIO18
  operating constraints](ch32h417weu6-hsadc-vio18-constraints.md).
- Keep the CH32 combined `VDDIO` and `VIO18` supply-pin current below its
  200-mA absolute maximum. The 0.60-A CH2 allocation is rail capacity for the
  buffer and associated loads; it is not permission to draw 0.60 A through MCU
  supply pins.

For externally powered `VDD12A`, V3F firmware calls `PWR_VDD12ExternPower()`
after `SystemInit()`, after the 1.218-V rail is stable, and before waking V5F
or initializing USB 3.0. Use a 20-ms first-board guard before that call.

### Capture Output Enable

Tie all four active-low `OE#` inputs of SN74LVC16244A directly to `GND`. The
buffer is always enabled when its `VIO18` supply is valid; the current design
contains no `CAPTURE_OE_N` net, PB2 assignment, pull-up, or firmware-controlled
output gate.

TI `SN74LVC16244A` Rev. C, page 11, Section 9.4, Table 3 states that
`OE = L` makes `Y` follow `A`, while `OE = H` makes `Y` high impedance. A
direct GND connection is therefore the defined always-enabled state, not an
undefined reset state. The buffer and capture GPIOs share the external
`VIO18` rail; this connection neither drives an MCU output nor changes the
separate WCH external-`VIO18` handover evidence boundary.

## TPS25200DRVR Power Path

| Item | Value / connection |
| --- | --- |
| `IN` bypass | `0.1 uF`, 50 V X7R at IN-GND, plus `10 uF`, 25 V X5R/X7R local bulk |
| `OUT` bypass | `22 uF`, 16 V X5R/X7R at OUT-GND |
| `RILIM` | `75 kOhm`, 1%, 0603, direct to local ground |
| `EN` | `300 kOhm`, 1%, from `VBUS_RAW` to EN |
| `FAULT` | NC |
| Input TVS | Keep a DNP footprint on `VBUS_RAW`; do not populate it by default for a compliant 5-V host |

`75 kOhm` is an E24 common resistance value. With TI's equation and a 1%
resistor it gives an approximately `1.175-1.420 A` limit. The maximum remains
below the 1.5-A Type-C Medium advertisement. Its lower guaranteed limit sets
the normal full-capture design budget to `1.10 A` maximum; `1.15 A` is not an
acceptable operating target. The previous `68 kOhm` selection is rejected
because its maximum limit is 1.561 A.

```text
3.32 V x 0.50 A + 1.218 V x 0.45 A + 3.00 V x 0.60 A = 4.01 W
```

At 4.65 V and 80% conversion efficiency, the buck input is about 1.08 A.
Reserve **1.10 A maximum normal full-capture input current** after including
direct 5-V loads and board loss. Full capture must remain disabled until
HD3SS3220 reports Medium or High source current.

## HD3SS3220 Power-On Control

Use `TPS3898ADRYT`, not `TPS3896ADRYT`:

```text
VDD5 -------------------------- TPS3898ADRYT.VCC
GND --------------------------- TPS3898ADRYT.GND, ENABLE_N
VDD5 -- 0.1 uF ---------------- GND

VCC33 -- 422 kOhm, 0.1% --+---- SENSE
                           |
                         82.5 kOhm, 0.1%
                           |
                          GND

CT -- 47 nF X7R --------- GND
VDD5 -- 12 kOhm, 1% -----+---- HD3SS3220.ENn_CC
SENSE_OUT (open drain) --+
```

TPS3898A has the required active-low-enable timing polarity and an open-drain
`SENSE_OUT`. Below the 3.02-3.09-V `VCC33` threshold, the output is high
impedance and the 12-kOhm resistor establishes `ENn_CC = VDD5`. After `VCC33`
is good, the 47-nF capacitor gives about 190 ms typical delay before it sinks
`ENn_CC` low. The 12-kOhm pull-up demands 0.458 mA at 5.5 V.

TPS3896 is not valid here: its push-pull high output cannot meet HD3SS3220's
`ENn_CC >= VDD5 - 0.3 V` requirement. `HD3SS3220.VDD5` must still ramp in
25 ms or less. TPS25200 `FAULT` and all EA3036C `EN` pins do not connect to
the HD3SS3220 enable network.

## Required First-Board Measurements

1. At MCU pins, verify `VDD12` remains 1.17-1.27 V under USB3 and full capture
   load, and call `PWR_VDD12ExternPower()` only after it is stable.
2. Verify the finalized external `VIO18` circuit against an official WCH
   application document before assigning a startup or transition procedure.
   Do not infer PA5 reset behavior from the DAC driver API.
3. Test the required capture thresholds at each selected external `VIO18`
   voltage with all buffer `OE#` pins permanently tied to `GND`.
4. At HD3SS3220 pins, verify `VDD5 >= 4.5 V`, VDD5 ramp no slower than 25 ms,
   and `ENn_CC` remains at least `VDD5 - 0.3 V` until both rails are stable,
   then remains high at least 2 ms before dropping below 0.4 V.
5. Measure VDD5 current, TPS25200/EA3036C temperature, and `VIO18` ripple at
   all-channel switching load. Normal full-capture input current must stay
   below 1.10 A.

## Sources

- [WCH CH32H417 data sheet](https://www.wch-ic.com/downloads/CH32H417DS0_PDF.html)
- [WCH CH32H417 external-VDD12 SDK example](https://github.com/openwch/ch32h417/tree/main/EVT/EXAM/PWR/VDD12_ExternPower)
- [Everanalog EA3036C data sheet](https://datasheet.lcsc.com/datasheet/pdf/d83e9c52d5fbdb916908193b63b8ae60.pdf?productCode=C570857)
- [TI TPS25200 data sheet](https://www.ti.com/lit/ds/symlink/tps25200.pdf)
- [TI HD3SS3220 data sheet](https://www.ti.com/lit/ds/symlink/hd3ss3220.pdf)
- [TI TPS389x data sheet](https://www.ti.com/lit/ds/symlink/tps3896.pdf)
- [TI SN74LVC16244A data sheet](https://www.ti.com/lit/ds/symlink/sn74lvc16244a.pdf)
