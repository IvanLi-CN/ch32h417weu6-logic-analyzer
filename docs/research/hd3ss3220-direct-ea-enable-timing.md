# HD3SS3220 and EA3036C Enable Timing

## Decision Boundary

EA3036C `EN1` is driven from `VBUS_5V_PROTECTED`, the TPS25200 protected
output, through `100 kOhm`, with `1 uF` from EN1 to GND. `EN2` and `EN3` each
use a local `3V3`-referenced `10 kOhm / 100 nF` RC network. This is the
selected always-on policy for the three CH32H417 rails: CH1 is passively
delayed from the protected input and the other two channels are passively
released after `3V3` starts rising. There is no TPS25200/EA3036C `FAULT` or
`EN` connection to an HD3SS3220 enable pin.

The relevant power paths are:

```text
TPS25200 OUT -> VBUS_5V_PROTECTED -> HD3SS3220 VDD5
                                       -> EA3036C VIN
EA3036C fixed 3.3 V -----------------> HD3SS3220 VCC33
```

## HD3SS3220 Requirement

`DIR` is pulled to `VCC33`, and the I2C lines are pulled to `VIO18`; both are
non-failsafe HD3SS3220 pins supplied from something other than `VDD5`.

TI permits `ENn_CC` to remain low during power-on only when `VDD5` is stable at
least 2 ms before `VCC33` is valid. If that cannot be guaranteed, TI requires
`ENn_CC` to stay high while both supplies ramp, then be asserted low after both
supplies are stable. `ENn_CC` is active low.

## Consequence for This Board

EA3036C specifies that an EN RC can program startup time, but does not specify
an EN-threshold upper bound or a maximum EN-to-output-stable time that proves
the required 2 ms interval. The selected EN1 RC therefore does not by itself
justify `ENn_CC -> GND`.

The EA EN RC topology is selected for startup separation, not as a
component-specification proof of the HD3SS3220 timing requirement. The current
netlist retains `ENn_CC -> GND`; it must be qualified at the HD3SS3220 pins for
all required power events. A controlled `ENn_CC` sequence through TPS3898ADRYT
is documented as an unpopulated alternative in
[HD3SS3220 Reliable Power-On Control](hd3ss3220-reliable-power-on-control.md).

This is independent of the CH32H417 power-rail order, for which no fixed
sequence is required.

## Sources

- [TI HD3SS3220 data sheet, Rev. E, Sections 5.6 and 6.3.12](https://www.ti.com/lit/ds/symlink/hd3ss3220.pdf)
- [Everanalog EA3036C data sheet, Enable Control](https://datasheet.lcsc.com/szlcsc/2005300208_Everanalog-IC-design-EA3036C_C570857.pdf)
