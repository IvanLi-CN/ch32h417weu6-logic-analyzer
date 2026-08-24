# CH32H417WEU6 HSADC and VIO18 Operating Constraints

## Purpose

This note separates the documented steady-state supply constraints for HSADC
operation from the separate, unresolved startup and handover boundary for the
project's independently regulated external `VIO18` rail.

It is the normative source for the HSADC/VIO18 steady-state question. All
statements below apply after the rails have reached their selected voltages.

## Required Steady-State Rails

When HSADC is enabled on `CH32H417WEU6`, use the following rails:

| Rail | Required project value | Reason |
| --- | ---: | --- |
| `VDD33A` | `3.32 V` | WCH requires 3.0 V to 3.6 V for HSADC. |
| `VREFP` | Internal `VDD33A` | WEU6 does not expose a `VREFP` pin; WCH defines an unbonded `VREFP` as internally equal to `VDD33A`. |
| `VDDIO` | `3.32 V` | The HSADC table requires `VDDIO >= VREFP`; the family supply relation requires `VDD33A >= VDDIO`. |
| `VIO18` | `1.80 V`, `2.50 V`, or `3.00 V` | The HSADC table adds no separate `VIO18` value requirement. It remains the selected capture-I/O rail and must not exceed `VDDIO`. |

The first three rows therefore resolve to:

```text
VDD33A = VREFP(internal) = VDDIO = 3.32 V
VIO18 <= VDDIO
```

`VIO18` is not an HSADC reference rail. HSADC's documented conversion range is
`0 V` to `VDDIO` when `VREFP >= VDDIO`, which is the case above. Thus the
HSADC conversion range remains approximately `0 V` to `3.32 V` in every
selected capture-voltage mode; it does not shrink to 1.80 V or 2.50 V when
`VIO18` is changed.

In particular, enabling HSADC does not force `VIO18` to 3.3 V. The documented
steady-state HSADC condition is met with the selected 1.80-V, 2.50-V, and
3.00-V `VIO18` modes because the HSADC constraints name `VDD33A`, `VREFP`,
and `VDDIO`, while the general rail relationship remains `VIO18 <= VDDIO`.

## Capture-Voltage Modes

| Probe logic standard | `VIO18` and buffer `VCC` | `VDD33A`, `VREFP`, and `VDDIO` | HSADC conversion range |
| --- | ---: | ---: | ---: |
| 1.8 V | `1.80 V` | `3.32 V` | `0 V` to `3.32 V` |
| 2.5 V | `2.50 V` | `3.32 V` | `0 V` to `3.32 V` |
| 3.3 V or 5 V | `3.00 V` | `3.32 V` | `0 V` to `3.32 V` |

The `3.00-V` capture mode is the selected 3.3-V and 5-V threshold mode. The
project does not support a 1.2-V probe-logic mode.

This is a power-rail arrangement, not an added signal-processing stage. The
existing `SN74LVC16244A` remains the sole capture-input buffer, with its `VCC`
following the selected `VIO18` voltage; no additional series signal IC is
introduced.

## WEU6 Package Consequences

- WEU6 exposes neither `VREFP` nor `VSSA`; do not create PCB nets, decoupling
  capacitors, or pin assignments for those absent pins.
- Place the WCH-required `0.1 uF` decoupling capacitor at the exposed
  `VDD33A` pin. Its return joins the normal board ground system; it is not a
  separately routed `VSSA` return.
- The available WEU6 HSADC inputs are `PC0` through `PC3`. The current
  two-input port uses `PC2/HSADC2` and `PC3/HSADC3`. Those pads are in the
  high-speed `VIO18` GPIO domain when used digitally, but the HSADC electrical
  table names `VDDIO` and `VREFP`, not `VIO18`, as the conversion supply
  constraints.

## Evidence Boundary

The values above satisfy the published steady-state WCH rail inequalities and
HSADC electrical conditions. They do not approve the project's independently
regulated external `VIO18` topology, its interaction with the internal I/O
LDO, or any POR, reset, brownout, or voltage-transition sequence. Those
questions remain governed by [the external-VIO18 official analysis](ch32h417-external-vio18-dac-control-official-analysis.md).

Validate PC2/PC3 HSADC acquisition at each selected `VIO18` voltage on the
first board. That test confirms the intended mixed analog/digital use; it is
not a substitute for the missing external-rail handover specification.

## Primary Sources

- WCH, [CH32H417DS0 V1.8](https://www.wch-ic.com/downloads/CH32H417DS0_PDF.html):
  Section 1.4.3 (supply definitions and relationships), Section 1.4.45
  (GPIO supply domains), Table 3-52 (HSADC supply and conversion range), and
  the WEU6 pin table.
- WCH, [CH32H417RM V1.7](https://www.wch-ic.com/downloads/CH32H417RM_PDF.html):
  Section 2.1, which defines an unbonded `VREFP` as internally equal to
  `VDD33A`.
