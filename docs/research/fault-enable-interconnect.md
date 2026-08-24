# Fault and Enable Interconnect

## Scope and Decision

This note treats `FATURE` as `FAULT`.

For the selected fixed-UFP/Sink USB-C architecture, do **not** directly wire a
fault-status output from one of these devices to an enable input on another:

```text
USB-C VBUS -> TPS25200 -> VBUS_5V_PROTECTED -> EA3036C -> board rails
                                                 |
                                                 +-> HD3SS3220 VDD5
EA3036C 3.3 V ------------------------------------> HD3SS3220 VCC33
```

The only polarity-compatible pair is `TPS25200.FAULT` to an
`EA3036C.ENx` node. It is deliberately **not used** in this project. It would
change the downstream-rail restart policy after a TPS25200 fault but add no
required protection: the TPS25200 already limits or disconnects its protected
5-V output, and the EA3036C input is downstream of that output. The TPS25200
FAULT output also does not assert during output-voltage-clamp (OVC) operation,
so it is not a complete input-overvoltage interlock.

## Pin Roles

| Device | Pin | Direction and active state | Project disposition |
| --- | --- | --- | --- |
| TPS25200 | `EN` | Input; high enables the eFuse power switch. | `300 kOhm` pull-up to `VBUS_RAW`; no cross-chip connection. |
| TPS25200 | `FAULT` | Active-low open-drain output for overcurrent, overtemperature, or OVLO; it does not assert in OVC mode. | Leave NC; no reliable downstream-MCU observation function is selected. |
| EA3036C | `EN1`, `EN2`, `EN3` | Inputs; high enables the respective buck channel. No external fault-status output exists. | All three connect directly to `VBUS_5V_PROTECTED`. No EN RC and no cross-chip control is selected. |
| HD3SS3220 | `ENn_CC` | Input; low enables the CC controller. | No connection to TPS25200 or EA3036C control pins. Connect to the TPS3898A open-drain output with a VDD5 pull-up; the supervisor releases it only after `VCC33` is valid. |
| HD3SS3220 | `ENn_MUX` | Input; low enables the SuperSpeed mux. | Tie to GND in this fixed UFP design. |
| HD3SS3220 | `VCONN_FAULT_N` | Active-low open-drain output only for VCONN overcurrent. | Leave NC: a fixed UFP/Sink does not source VCONN. |

## Cross-Chip Matrix

| Source | Target | Electrical result | Decision |
| --- | --- | --- | --- |
| `TPS25200.FAULT` | `EA3036C.ENx` | A low FAULT can disable a high-active EN input. | Logic polarity matches, but do not connect in this project. |
| `TPS25200.FAULT` | `HD3SS3220.ENn_CC` or `ENn_MUX` | A low FAULT enables, rather than disables, the target. | Do not connect. |
| `HD3SS3220.VCONN_FAULT_N` | `TPS25200.EN` or `EA3036C.ENx` | A low output could disable a high-active enable input. | Do not connect. It never asserts in the fixed UFP/Sink role. |
| `HD3SS3220.VCONN_FAULT_N` | `HD3SS3220.ENn_CC` or `ENn_MUX` | A low fault output enables the target. | Do not connect. |
| `EA3036C.ENx` | either HD3SS3220 `ENn` input | EA EN is high-active while both HD3 enables are low-active. | Do not connect. |
| `TPS25200.EN` | `EA3036C.ENx` | Both are high-active, but the eFuse EN sits on raw VBUS while EA EN must follow the protected rail. | Do not connect. |

## Why `TPS25200.FAULT -> EA3036C.ENx` Is Not Used

`TPS25200.FAULT` is an open-drain sink. A dedicated hardware-hiccup design
could pull an EA3036C EN node low with it, provided that EN is pulled up from
`VBUS_5V_PROTECTED` and the FAULT sink-current limit is respected. That is a
different power-policy design, not a free connection in the selected circuit.

All three EA3036C enables are intentionally direct-connected to
`VBUS_5V_PROTECTED`. This establishes the selected always-on CH32H417 power
policy; it is not an HD3SS3220 timing guarantee. The TPS25200 does not assert
FAULT in OVC mode, so a FAULT-to-EN connection cannot serve as a general
input-overvoltage shutdown policy.

An earlier documentation proposal to pull FAULT up to `VIO18` and read it with
the MCU is withdrawn. `VIO18` and the MCU are both downstream of TPS25200 OUT,
so they cannot reliably observe a fault that removes or collapses OUT.

If a future requirement explicitly asks for hardware shutdown of every
EA3036C output upon a TPS25200 fault, design that as a separate interlock with
proper domain isolation and validate the resulting restart waveform. It is not
needed for the current protected USB-powered analyzer.

## Required Net Connections

```text
TPS25200 EN       <- 300 kOhm <- VBUS_RAW
TPS25200 FAULT    <- NC

EA3036C EN1       <- VBUS_5V_PROTECTED
EA3036C EN2       <- VBUS_5V_PROTECTED
EA3036C EN3       <- VBUS_5V_PROTECTED

HD3SS3220 ENn_CC  <- TPS3898A.SENSE_OUT, with 12 kOhm pull-up to VDD5
HD3SS3220 ENn_MUX <- GND
HD3SS3220 VCONN_FAULT_N <- NC
```

## Primary Sources

- [TI TPS25200 data sheet, Rev. F](https://www.ti.com/lit/ds/symlink/tps25200.pdf), pin functions and Sections 7.3.1 and 7.3.4.
- [Everanalog EA3036C data sheet](https://datasheet.lcsc.com/szlcsc/2005300208_Everanalog-IC-design-EA3036C_C570857.pdf), electrical characteristics and Enable Control section.
- [TI HD3SS3220 data sheet, Rev. E](https://www.ti.com/lit/ds/symlink/hd3ss3220.pdf), pin functions and Sections 6.3.9 and 6.3.12.
- [HD3SS3220 Reliable Power-On Control](hd3ss3220-reliable-power-on-control.md): selected TPS3898A circuit.
