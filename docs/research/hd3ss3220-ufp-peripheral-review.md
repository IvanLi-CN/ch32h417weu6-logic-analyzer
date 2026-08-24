# HD3SS3220 Fixed-UFP Peripheral Review

## Scope

This is the exact `HD3SS3220` external circuit required for the project's
single USB-C device port:

- `HD3SS3220` in I2C mode, fixed UFP/Sink role;
- one CH32H417WEU6 USB 3.2 Gen 1 device link;
- host-supplied 5 V VBUS, with the project power path
  `USB-C VBUS -> TPS25200 -> 5 V protected rail -> board rails`;
- one board `3.3 V` rail;
- no USB PD or source role.

The USB-C circuit below follows the TI data sheet; the MCU pin and
voltage-domain allocation also uses the WCH documentation. `VBUS_RAW` and
`VBUS_5V_PROTECTED` are project net names defined here to make the TPS25200
boundary unambiguous:

```text
USB-C receptacle VBUS -> VBUS_RAW -> TPS25200 IN
TPS25200 OUT          -> VBUS_5V_PROTECTED -> EA3036C and HD3SS3220 VDD5
```

`VBUS_RAW` is the connector-facing 5 V VBUS node before the series TPS25200
power path. It is the project's instance of the data sheet's `system VBUS`
node for `VBUS_DET`.

Primary sources throughout: [TI HD3SS3220 data sheet, Rev. E](https://www.ti.com/lit/ds/symlink/hd3ss3220.pdf)
and [WCH CH32H417 data sheet](https://www.wch-ic.com/downloads/CH32H417DS0_PDF.html).

## Must-Fit Circuit

| Ref. | Connection or value | Pin disposition | Evidence |
| --- | --- | --- | --- |
| `U1.VDD5` | Connect to `VBUS_5V_PROTECTED`. Fit `C1 = 100 nF`, 10 V, X7R, `0402` from `VDD5` to GND at U1. | Required 5 V CC-controller supply. | TI specifies `VDD5 = 4.5 V` to `5.5 V`; the UFP reference shows 100 nF local supply decoupling. See Sections 5.3 and 7.2.4, Table 7-3. |
| `U1.VCC33` | Connect to the board `3.3 V` rail. Fit `C2 = 100 nF`, 6.3 V, X7R, `0402` from `VCC33` to GND at U1. | Required mux supply. | TI specifies `VCC33 = 3.0 V` to `3.6 V`; the UFP reference shows 100 nF local supply decoupling. See Sections 4, 5.3, and 7.2.4. |
| `U1.GND` pins 13, 28 and exposed pad | Connect directly to GND. | Required. | Section 4 requires the thermal pad to connect to GND. |
| `U1.CC1`, `U1.CC2` | Connect directly to the matching USB-C receptacle `CC1` and `CC2` contacts. Fit **no external CC pull-down resistors**. | Required. | In UFP mode the device provides internal `Rd = 4.6 kOhm` to `5.6 kOhm`, 5.1 kOhm typical. See Sections 4 and 5.5. |
| `R1` | `910 kOhm`, 1%, `0402`, from `VBUS_RAW` to `U1.VBUS_DET`. | Required. | TI requires one resistor from system VBUS to `VBUS_DET` for UFP attach/detach detection. The 900 kOhm nominal UFP reference value is shown in Figure 7-3. A 910 kOhm nominal part is at the high end of the 880 kOhm to 910 kOhm recommended range, and its +1% value remains below the 920 kOhm electrical maximum. See Sections 4, 5.3, 5.5, 6.3.11, and 7.2.4. |
| `U1.PORT` | Tie directly to GND. | Required fixed-role configuration. | `PORT = L` selects UFP. The value is sampled when `ENn_CC` is low and `VDD5` is active. See Section 4. |
| `U1.ADDR` | Tie directly to GND. | Required I2C configuration. | Low `ADDR` selects I2C mode at the 7-bit address `0x47`. See Sections 4 and 6.3.10. |
| `U1.CURRENT_MODE` | Leave electrically NC. | Required disposition for this fixed UFP use. | The pin is explicitly a don't-care in UFP mode; its DFP current-advertisement resistors do not apply. See Section 4. |
| `U1.ENn_MUX` | Tie directly to GND. | Required for an always-active USB 3 mux. | It is active low: low is normal operation and high is shutdown. See Section 4. |
| `U1.DIR` | Fit `R2 = 200 kOhm`, 1%, `0402` from `DIR` to `VCC33`. Do not drive the pin from the MCU. | Required pull-up; the MCU need not consume the signal. | `DIR` is an open-drain orientation output and TI states that a 200 kOhm pull-up **must** be installed for proper operation. The mux uses orientation internally. See Sections 4 and 7.2.4, Table 7-3. |
| `U1.ENn_CC` | Connect to `TPS3898ADRYT.SENSE_OUT`; fit the selected 12 kOhm pull-up from `VDD5`, divider from `VCC33`, and 47 nF CT capacitor. Do not connect to any TPS25200 or EA3036C `FAULT`/`EN` pin. | Required to enable CC operation. | This implements TI's controlled-enable requirement for the non-failsafe pull-ups. See Section 6.3.12 and [Reliable Power-On Control](hd3ss3220-reliable-power-on-control.md). |
| `U1.TXp/TXn` | Connect directly from the CH32H417 transmitter pair to `TXp` and `TXn`. Do not fit an AC-coupling capacitor in this MCU-facing segment. | Required SuperSpeed transmit path. | TI's UFP reference circuit connects the system `SSTXP/SSTXN` pair directly to `TXp/TXn`; its AC-coupling capacitors are on the Type-C-facing TX outputs. See Figure 7-3 and Table 7-3. |
| `U1.RXp/RXn` | Connect directly to the CH32H417 receiver pair. Do not add an HD3SS3220 AC-coupling capacitor here. | Required SuperSpeed receive path. | TI states that RX pairs are biased by the host receiver; its UFP coupling requirement is TX pairs only. See Section 7.2.4, Table 7-3. |
| `U1.TX1p/n`, `U1.TX2p/n` | Fit `C3` through `C6`, each `100 nF`, 10 V, X7R, `0402`, one in every conductor between these U1 TX outputs and the matching USB-C SuperSpeed TX lane. | Required Type-C-facing transmit paths. | TI's UFP reference shows four 100 nF AC-coupling capacitors: one in each `TX1p/n` and `TX2p/n` conductor. The permitted value range is 75 nF to 200 nF. See Figure 7-3 and Table 7-3. |
| `U1.RX1p/n`, `U1.RX2p/n` | Connect directly to the corresponding USB-C SuperSpeed RX lane pairs. Do not add AC-coupling capacitors. | Required Type-C-facing receive paths. | The receiver supplies the required common-mode bias. See Table 7-3 and Figure 7-3. |
| `U1.SDA/OUT1`, `U1.SCL/OUT2` | Connect directly to CH32H417WEU6 `PB11/I2C2_SDA(AF4)` and `PB10/I2C2_SCL(AF4)` respectively. Fit `R3`, `R4` = `4.7 kOhm`, 5%, `0402`, from each line to external `VIO18`. | Required I2C bus. | The HD3SS3220 I2C-pin supply range is 1.65 V to 3.6 V. The existing VIO18 circuit must remain in that range while I2C is active. |

`C3` and `C4` are the two conductors of one Type-C-facing TX lane, and `C5`
and `C6` are the two conductors of the other Type-C-facing TX lane. This is
four capacitors total, exactly as shown in the TI UFP reference circuit. Do not
fit a second series-coupling set between the CH32H417 transmitter and
`U1.TXp/TXn`, and do not fit coupling capacitors in the receive paths.

## Project Power-On Constraint

The required 200 kOhm `DIR` pull-up is to the `VCC33` rail, and `DIR` is one
of the data sheet's non-failsafe pins. The data sheet therefore requires the
following condition for a direct `ENn_CC -> GND` connection:

```text
VDD5 is stable for at least 2 ms before VCC33 reaches its valid level.
```

For this project, `VDD5` is `VBUS_5V_PROTECTED`, the TPS25200 output. `VCC33`
is the EA3036C fixed 3.3 V output. All three EA3036C `EN` pins connect directly
to `VBUS_5V_PROTECTED`; no EA EN RC network is used. This is the correct
always-on policy for the CH32H417 rails, but the EA3036C data sheet gives no
maximum start-up timing that proves the 2 ms HD3SS3220 requirement.

Consequently, this project uses TI's controlled `ENn_CC` method rather than a
permanent GND connection. `TPS3898ADRYT` releases the input to its VDD5
pull-up during both supply ramps and sinks it only after `VCC33` passes the
chosen threshold and the CT delay expires. This is a local HD3SS3220 enable decision, not a
connection from a TPS25200 or EA3036C `FAULT`/`EN` pin.

Evidence: [Sections 5.6 and 6.3.12](https://www.ti.com/lit/ds/symlink/hd3ss3220.pdf).

## I2C GPIO Allocation and Firmware Boundary

The allocated MCU pins are fixed as follows:

| Function | WEU6 QFN68 pad / GPIO | Alternate function | Electrical side |
| --- | --- | --- | --- |
| I2C clock | `26 / PB10` | `I2C2_SCL`, AF4 | Direct to `U1.SCL/OUT2`, pulled up to `VIO18` |
| I2C data | `27 / PB11` | `I2C2_SDA`, AF4 | Direct to `U1.SDA/OUT1`, pulled up to `VIO18` |

`PB10` and `PB11` are `VIO18`-domain pins, so the I2C pull-ups must also use
external `VIO18`, not fixed `VCC33`. The HD3SS3220 specifies a 1.65 V to 3.6 V
supply range for its SDA/SCL pins. The existing VIO18 circuit must stay inside
that range while I2C is active, so the direct two-wire connection needs no
active level shifter.

Configure I2C2 as a standard-mode `100 kHz` master and use the HD3SS3220
7-bit address `0x47`. This is a firmware change, but it does not change the
USB data-path firmware or add USB PD behavior. `INT_N/OUT3` is left NC; the
firmware polls status over I2C, so no third GPIO is required. Do not start an
I2C transfer while firmware changes `VIO18`;
complete the voltage transition and wait for the rail to settle before the
next controller access.

## Intentionally Unused Pins and Parts

| Item | Required disposition for this project | Reason |
| --- | --- | --- |
| `INT_N/OUT3` | Leave NC. Do not fit a pull-up. | In I2C mode it is the optional interrupt output. Firmware polls status instead, so no third GPIO is needed. |
| `ID` | Leave NC. | It asserts only when the port is a source or a DRP acting as source, neither of which exists here. |
| `VCONN_FAULT_N` | Leave NC. | VCONN is supplied only in DFP mode or DRP acting as DFP. The fixed port is UFP/Sink. |
| External `Rd` on `CC1` and `CC2` | Do not fit. | The `HD3SS3220` already supplies UFP `Rd`; adding external 5.1 kOhm resistors would parallel it. |
| `CURRENT_MODE` pull-up resistor | Do not fit. | The 500 kOhm and 10 kOhm options select source current in DFP mode only; `CURRENT_MODE` is don't-care in UFP mode. |
| I2C pull-up to fixed `VCC33` | Do not fit. | The bus operates directly at the current `VIO18` voltage. The only I2C pull-ups are `R3` and `R4` to `VIO18`. |
| VCONN bulk capacitor | Do not fit for this fixed UFP port. | TI specifies the 10 uF to 200 uF VCONN bulk capacitance only when VCONN is on; VCONN is not sourced in UFP mode. |
| USB PD controller or VBUS source circuitry | Do not fit. | The project is a 5 V UFP/Sink and does not require PD or source role. The HD3SS3220 data sheet explicitly separates those functions from the fixed UFP use. |

The non-L `HD3SS3220` can detect UFP audio/debug accessories by default.
Leaving `INT_N/OUT3` NC means the analyzer deliberately takes no asynchronous
system action on that indication; I2C polling remains available if firmware
needs the controller state.

## Schematic-Level Netlist

```text
USB-C CC1  -------------------------- U1.CC1
USB-C CC2  -------------------------- U1.CC2

USB-C VBUS -------------------------- VBUS_RAW ---------------- TPS25200 IN
VBUS_RAW ---- R1 910 kOhm, 1% ------- U1.VBUS_DET
TPS25200 OUT ------------------------ VBUS_5V_PROTECTED ------- U1.VDD5
U1.VDD5 ---- C1 100 nF -------------- GND
Board 3.3 V ------------------------- U1.VCC33
U1.VCC33 -- C2 100 nF -------------- GND

VBUS_5V_PROTECTED -------------------- EA3036C EN1
VBUS_5V_PROTECTED -------------------- EA3036C EN2
VBUS_5V_PROTECTED -------------------- EA3036C EN3

U1.PORT ----------------------------- GND
U1.ADDR ----------------------------- GND (I2C 7-bit address 0x47)
U1.CURRENT_MODE --------------------- NC
U1.ENn_MUX -------------------------- GND
U1.ENn_CC --------------------------- TPS3898ADRYT.SENSE_OUT, 12 kOhm pull-up to VDD5
U1.DIR ---- R2 200 kOhm ------------- U1.VCC33

PB10 (I2C2_SCL, AF4) ---------------- U1.SCL/OUT2
PB11 (I2C2_SDA, AF4) ---------------- U1.SDA/OUT1
PB10 ---- R3 4.7 kOhm, 5% ----------- VIO18
PB11 ---- R4 4.7 kOhm, 5% ----------- VIO18
U1.INT_N/OUT3 ----------------------- NC

CH32H417 SSTX+ ---------------------- U1.TXp
CH32H417 SSTX- ---------------------- U1.TXn
CH32H417 SSRX+ ---------------------- U1.RXp
CH32H417 SSRX- ---------------------- U1.RXn

U1.TX1p ---- C3 100 nF -------------- matching USB-C TX1 conductor
U1.TX1n ---- C4 100 nF -------------- matching USB-C TX1 conductor
U1.TX2p ---- C5 100 nF -------------- matching USB-C TX2 conductor
U1.TX2n ---- C6 100 nF -------------- matching USB-C TX2 conductor
U1.RX1/RX2 -------------------------- matching USB-C SuperSpeed RX lane pairs
U1.GND pins and exposed pad --------- GND
```

## Primary Evidence Index

- [Section 4, Pin Functions](https://www.ti.com/lit/ds/symlink/hd3ss3220.pdf):
  fixed role selection, `VBUS_DET`, `DIR`, enables, GPIO/I2C selection, and
  all SuperSpeed pin roles.
- [Sections 5.3 and 5.5](https://www.ti.com/lit/ds/symlink/hd3ss3220.pdf):
  supply ranges, `VBUS_DET` resistor range, internal UFP `Rd`, open-drain
  pull-up value, and VCONN capacitor condition.
- [Sections 6.3.10 to 6.3.12](https://www.ti.com/lit/ds/symlink/hd3ss3220.pdf):
  GPIO output functions, VBUS attach/detach detection, and the required
  VDD5/VCC33 power-on timing.
- [Section 7.2.4 and Table 7-3](https://www.ti.com/lit/ds/symlink/hd3ss3220.pdf):
  TI's UFP reference circuit, 100 nF TX coupling, 200 kOhm `DIR` pull-up, and
  900 kOhm nominal `VBUS_DET` connection.
