# TPD4E05U06 USB-C SuperSpeed ESD Routing

## Decision

For a full USB-C SuperSpeed port, two `TPD4E05U06DQAR` devices protect the
receptacle's eight SuperSpeed conductors. One DQA package has four independent
shunt ESD channels: `D1+`, `D1-`, `D2+`, and `D2-`. Assign `D1+/D1-` to one
complete differential pair and `D2+/D2-` to a second complete differential
pair.

The device is a shunt TVS, not a series component. Route each protected signal
through its I/O pad without a branch stub, and connect both GND pins to the
ground plane with the shortest practical return path.

## USB-C Receptacle Assignment

For the Type-C receptacle, group each DQA package by the connector's physical
A or B contact side. This keeps the connector-to-TVS segment short and avoids
unnecessary crossing. The A/B contact groups are not the same thing as the two
logical plug orientations:

| TVS | DQA channel pair | USB-C contact / net | HD3SS3220 port-side signal |
| --- | --- | --- | --- |
| `U_ESD_A` | `D1+` / `D1-` | `A2/A3`, `TXP1/TXN1` | `TX2p/TX2n` through the Type-C-facing coupling capacitors |
| `U_ESD_A` | `D2+` / `D2-` | `A11/A10`, `RXP2/RXN2` | `RX1n/RX1p` |
| `U_ESD_B` | `D1+` / `D1-` | `B2/B3`, `TXP2/TXN2` | `TX1p/TX1n` through the Type-C-facing coupling capacitors |
| `U_ESD_B` | `D2+` / `D2-` | `B11/B10`, `RXP1/RXN1` | `RX2n/RX2p` |

Thus, one TVS protects the physical A side (`TX1` plus `RX2`) and the other
protects the physical B side (`TX2` plus `RX1`). Do not interpret those two
pairs as one logical plug orientation. The mux selects the corresponding TX
and RX paths internally.

The project uses a complete 1/2 mapping: physical `CC1` goes to controller
`CC2`, physical `CC2` goes to controller `CC1`, and the TX/RX relationships in
the table change with it. The receive P/N order is also reversed at the
controller. This is the observed netlist mapping, not a license to alter one
CC or lane connection independently. The authoritative complete table is in
[HD3SS3220 fixed-UFP peripheral review](hd3ss3220-ufp-peripheral-review.md#project-port-mapping).

Do not use one `TPD4E05U06DQAR` for the complete port: it protects only four
of the eight SuperSpeed conductors. A reversed plug would leave one selected
TX or RX pair unprotected.

## Path Order

The TVS devices go immediately behind the USB-C connector. On transmit paths,
the path is connector -> TVS -> 100 nF AC-coupling capacitor -> HD3SS3220 TX
output. On receive paths, the path is connector -> TVS -> HD3SS3220 RX input.
The TVS placement is therefore connector-facing relative to the transmit
coupling capacitors.

## Layout Constraints

- Place both TVS packages as close to the receptacle as possible; keep all
  trace length between connector contacts and the TVS pads minimal.
- Connect DQA pins `3` and `8` to the ground plane with low-inductance paths;
  use nearby ground vias and do not share a long narrow return trace.
- Preserve each differential pair through the TVS: `D1+` with `D1-`, and
  `D2+` with `D2-`. The selected fabrication process does not support a
  controlled-impedance or pair-length-matching acceptance gate; retain short,
  direct, symmetric routing where practical and do not create a branch stub.
- The four channels are electrically symmetric. The A/B grouping is a layout
  choice for short, direct routing, not a special TX-versus-RX requirement.

The board's USB 2.0 and USB 3.0 paths are best-effort prototype links. Passing
means attach, enumeration, and sustained transfer in both plug orientations;
it is not a USB-IF compliance or guaranteed 5-Gbps claim.

## Sources

- [TI TPDxE05U06 data sheet, Rev. O](https://www.ti.com/lit/ds/symlink/tpd4e05u06.pdf): DQA pinout, four identical ESD channels, connector-near placement, and layout guidance.
- [TI HD3SS3220 data sheet, Rev. E](https://www.ti.com/lit/ds/symlink/hd3ss3220.pdf): Figure 7-3 USB-C SuperSpeed lane naming and the UFP port-side TX/RX assignment.
