# HD3SS3220 Direct-EA-Enable Timing

## Decision Boundary

All three EA3036C `EN` pins connect directly to `VBUS_5V_PROTECTED`, the
TPS25200 protected output. This is the selected always-on policy for the three
CH32H417 rails. There is no TPS25200/EA3036C `FAULT` or `EN` connection to an
HD3SS3220 enable pin.

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
a maximum EN-to-output start time that proves the required 2 ms interval.
Directly tying every EA EN to the protected 5 V input therefore does not by
itself justify `ENn_CC -> GND`.

Do not add an EA EN RC solely for HD3SS3220 timing. The project selects TI's
controlled-`ENn_CC` sequence through `TPS3898ADRYT`: its open drain releases
`ENn_CC` to the VDD5 pull-up while the rail is invalid, then sinks it low after
its delay. The exact circuit and acceptance measurement are in
[HD3SS3220 Reliable Power-On Control](hd3ss3220-reliable-power-on-control.md).

This is independent of the CH32H417 power-rail order, for which no fixed
sequence is required.

## Sources

- [TI HD3SS3220 data sheet, Rev. E, Sections 5.6 and 6.3.12](https://www.ti.com/lit/ds/symlink/hd3ss3220.pdf)
- [Everanalog EA3036C data sheet, Enable Control](https://datasheet.lcsc.com/szlcsc/2005300208_Everanalog-IC-design-EA3036C_C570857.pdf)
