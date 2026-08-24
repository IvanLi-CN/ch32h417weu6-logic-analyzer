# HD3SS3220 UFP Port Selection

## Decision

The USB-C port uses the non-L `HD3SS3220` as the Type-C CC controller and
reversible SuperSpeed 2:1 mux. The analyzer is a fixed USB device-side
UFP/Sink port with one USB 3.2 Gen 1 link to the host. The part's 10 Gbps
capability supplies link margin; it does not change the CH32H417 link rate.

Use I2C mode. The project needs a real MCU management connection to the
Type-C controller, while remaining a fixed UFP/Sink port. I2C is used only
for controller status and configuration; it does not add source mode,
DisplayPort, audio/debug accessory behavior, or USB Power Delivery
negotiation.

## Fixed Configuration

| Function | Required connection | Reason |
| --- | --- | --- |
| Port role | `PORT` to GND | Selects UFP-only mode. |
| Control mode | `ADDR` to GND | Selects I2C mode at 7-bit address `0x47`. |
| I2C master pins | `PB10` as `I2C2_SCL` (AF4) and `PB11` as `I2C2_SDA` (AF4) | New-project allocation; verify against the saved LCEDA schematic before routing. |
| I2C electrical connection | Connect `PB10` directly to `SCL/OUT2` and `PB11` directly to `SDA/OUT1`; add one `4.7 kOhm`, 5%, `0402` pull-up from each line to external `VIO18` | HD3SS3220 specifies a 1.65 V to 3.6 V I2C supply range. The existing VIO18 circuit must remain in that range while I2C is active; no level-shifter is used. |
| CC termination | `CC1` and `CC2` to the receptacle CC contacts; no external `Rd` resistors | The controller supplies the UFP `Rd` internally: 4.6 kOhm to 5.6 kOhm, 5.1 kOhm typical. Do not fit the two external 5.1 kOhm CC pull-downs used by a discrete-CC design. |
| VBUS sense | One `910 kOhm`, 1% resistor from `VBUS_RAW` to `VBUS_DET` | `VBUS_RAW` is the connector VBUS node before TPS25200. The pin senses 4 V to 28 V through this resistor; it is not a VBUS supply input. |
| 5 V supply | `VDD5` from `VBUS_5V_PROTECTED` | This is the TPS25200 output and must be 4.5 V to 5.5 V. |
| 3.3 V supply | `VCC33` from the EA3036C fixed 3.3 V output | Required range is 3.0 V to 3.6 V. |
| CC controller enable | `TPS3898ADRYT.SENSE_OUT` drives `ENn_CC`; add the selected 12 kOhm pull-up from `VDD5` and the supervisor's `VCC33` divider/CT network. | This implements the HD3SS3220 controlled-enable condition without altering the direct EA3036C enable policy. |
| Mux enable | Hold `ENn_MUX` low in normal operation | Enables the SuperSpeed mux. A permanently enabled port may tie it to GND. |
| Orientation status | Pull `DIR` up to `VCC33` with 200 kOhm | `DIR` is open drain and TI specifies this pull-up for proper operation. No MCU handling is needed for mux orientation. |
| Source-current selector | `CURRENT_MODE` NC | It is a don't-care input in UFP mode. |

In I2C mode, `SDA/OUT1` becomes SDA and `SCL/OUT2` becomes SCL. The exact
connection is:

```text
CH32 PB10 (I2C2_SCL, AF4) -- HD3SS3220.SCL/OUT2 -- 4.7 kOhm -> VIO18
CH32 PB11 (I2C2_SDA, AF4) -- HD3SS3220.SDA/OUT1 -- 4.7 kOhm -> VIO18
```

The two I2C lines use the same `VIO18` rail as PB10/PB11. Do not pull either
line up to fixed `VCC33`, because PB10/PB11 are not fixed-3.3-V GPIO. Configure
I2C2 at `100 kHz`.

`INT_N/OUT3`, `ID`, and `VCONN_FAULT_N` have no allocated system function in
this fixed UFP design. Leave `INT_N/OUT3` open and poll controller status over
I2C; this avoids allocating a third GPIO.

## Power and Protection Boundary

`HD3SS3220` implements Type-C CC attach/orientation and the SuperSpeed mux. It
does not implement USB PD messaging. The product therefore remains a 5 V sink
that must stay within the upstream host's advertised Type-C current capability.
A future requirement for a negotiated PD contract would require a separate PD
controller and is outside this design.

This selection does not remove the USB-C VBUS protection or current-limit
design. The exact boundary is `USB-C VBUS -> VBUS_RAW -> TPS25200 ->
VBUS_5V_PROTECTED`; `VBUS_DET` senses the raw side and `VDD5` uses the
protected side. The VBUS TVS decision remains separate from HD3SS3220.

## SuperSpeed Link

Connect the CH32H417 SuperSpeed TX/RX pairs to `TXp/TXn` and `RXp/RXn`, and
the receptacle's two reversible SuperSpeed lane sets to `TX1/TX2` and
`RX1/RX2`, following the TI UFP reference circuit. Connect the CH32H417 TX
pair directly to `TXp/TXn`. Fit four `100 nF` AC-coupling capacitors, one in
each conductor of the Type-C-facing `TX1p/n` and `TX2p/n` paths; TI permits
75 nF to 200 nF. Do not add a second series-coupling pair between the CH32H417
and `TXp/TXn`, and do not add coupling capacitors to the receive paths.

The mux does not require transmit-pair length to match receive-pair length;
only the two conductors within each differential pair require matching. Apply
the project USB 3 routing rules through the connector, mux, and MCU path.

## Prototype Checks

- Enumerate at USB SuperSpeed in both plug orientations.
- Check attach, detach, and VBUS removal in fixed UFP/Sink mode.
- Confirm that no external CC `Rd` resistors are populated.
- Verify I2C2 register access at every selected `VIO18` voltage and only after
  each voltage transition has settled.
- Confirm the VBUS power path stays within the current level advertised by the
  connected host. The <=1.10-A full-capture mode may start only after I2C reports
  Medium or High current; I2C status does not change the TPS25200 current
  limit by itself.

The complete project-specific netlist and BOM are in
[HD3SS3220 fixed-UFP peripheral review](hd3ss3220-ufp-peripheral-review.md).
The selected power-good circuit is specified in
[HD3SS3220 Reliable Power-On Control](hd3ss3220-reliable-power-on-control.md).

Sources: [TI HD3SS3220 data sheet](https://www.ti.com/lit/ds/symlink/hd3ss3220.pdf)
and [WCH CH32H417 data sheet](https://www.wch-ic.com/downloads/CH32H417DS0_PDF.html).
