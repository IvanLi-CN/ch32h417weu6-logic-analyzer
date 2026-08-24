# HD3SS3220 Reliable Power-On Control

## Selected Circuit

Use `TPS3898ADRYT`, not `TPS3896ADRYT`, to control `HD3SS3220.ENn_CC`.

```text
VDD5 -------------------------- TPS3898.VCC
GND --------------------------- TPS3898.GND, TPS3898.ENABLE_N
VDD5 -- 0.1 uF ---------------- GND

VCC33 -- 422 kOhm, 0.1% --+---- TPS3898.SENSE
                           |
                         82.5 kOhm, 0.1%
                           |
                          GND

TPS3898.CT -- 47 nF ----------- GND
VDD5 -------- 12 kOhm ---------+---- HD3SS3220.ENn_CC
TPS3898.SENSE_OUT --------------+
```

TPS3898 uses the same active-low-enable timing polarity as TPS3896, but its
`SENSE_OUT` is open drain. Below the approximately 3.02-3.09-V `VCC33`
threshold, the open drain releases and the 12-kOhm resistor pulls `ENn_CC` to
the actual VDD5 value. After the rail is valid, the 47-nF capacitor creates
about 190 ms typical delay before the output pulls the active-low HD3 input
low.

This closes the high-level requirement that TPS3896 could not close:
HD3SS3220 requires `ENn_CC >= VDD5 - 0.3 V`, while a TPS3896 push-pull high
output is only specified at `0.8 x VCC`. Do not fit TPS3896 in this circuit.

## Unchanged Decisions

- All EA3036C `EN` pins connect directly to `VDD5`.
- TPS25200 `FAULT` remains NC.
- No controller `FAULT` or `EN` pin connects to another controller.
- `VDD5` must ramp at the HD3SS3220 pin in 25 ms or less. The supervisor does
  not remove that requirement.

## Measurement Gate

At cold start, hot plug, brownout, and cable removal, measure `VDD5`, `VCC33`,
and `ENn_CC` at the HD3SS3220 pins. Once VDD5 is in its valid operating range,
the open-drain pull-up must hold `ENn_CC` at least `VDD5 - 0.3 V` until both
rails are stable; then it must remain high at least 2 ms before dropping below
0.4 V.

Sources: [TI HD3SS3220 data sheet](https://www.ti.com/lit/ds/symlink/hd3ss3220.pdf)
and [TI TPS389x data sheet](https://www.ti.com/lit/ds/symlink/tps3896.pdf).
