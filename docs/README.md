# Documentation Index

## Current Decision

- [Hardware Selection](SELECTION.md): current component and architecture
  choices, open decisions, and the required prototype validation.

## Research

- [Project Requirements](research/project-requirements.md): confirmed capture,
  voltage-domain, and layout constraints.
- [CH32H417 Package Comparison](research/ch32h417-package-comparison.md):
  hardware comparison of WEU6 and QEU6 and the reason WEU6 is selected.
- [CH32H417QEU6 Reference Power Topology](research/ch32h417-qeu6-reference-power-topology.md):
  supplied-reference analysis of the `AVCC`/`VDD33A`/`VREFP`/`VDDIO` path,
  the L4 ferrite-bead boundary, and the required HSADC validation.
- [Analog Front End](research/analog-front-end.md): OPA2356 and OPA2365
  comparison for the optional analog capture path.
- [EA3036C Inductor Selection](research/ea3036-inductor.md): rationale for
  the 1.5 uH output inductor.
- [CH32H417 Power Sequencing](research/ch32h417-power-sequencing.md): MCU
  supply constraints and EA3036C EN startup control.
- [Power Architecture](research/power-architecture.md): CH1 `3V3`, CH2
  external-`VIO18`, and CH3 `1V2` mapping; local `3V3`-referenced EA3036C
  enables; TPS25200 values; and the hardware-only HD3SS3220 power-on controller.
- [CH32H417WEU6 HSADC and VIO18 Operating Constraints](research/ch32h417weu6-hsadc-vio18-constraints.md):
  fixed 3.32-V HSADC supplies, selectable 1.80/2.50/3.00-V capture `VIO18`,
  and the remaining external-rail handover boundary.
- [TPS25200 Peripheral Selection](research/tps25200-peripherals.md):
  project-specific capacitors, current-limit resistor, enable, fault, TVS,
  and layout choices.
- [Logic Capture and HSADC Layout Rules](research/logic-adc-layout.md):
  project-specific routing, length-matching, analog placement, and prototype
  validation rules.
- [External VIO18 Official Evidence](research/vio18-external-handover-official-evidence.md):
  WCH-only evidence defining what is and is not established for external
  `VIO18` operation.
- [CH32H417 External VIO18 DAC-Control Official Analysis](research/ch32h417-external-vio18-dac-control-official-analysis.md):
  complete WCH-only reset, register, and transition analysis for the existing
  independent external rail.
- [CH32H417WEU6 External VIO18 Reset and EA3036C Enable Strategy](research/ch32h417-vio18-en-gated-dac-assessment.md):
  PA5/DAC2 reset evidence, the selected non-MCU enable network, and its
  documented timing boundary.
- [CH32H417 Official WCH Source Register](research/ch32h417-official-source-register.md):
  exact official data sheet, reference manual, and EVT sources used.

## Datasheets

- [WCH CH32H417 Datasheet Source Note](datasheets/wch-ch32h417-datasheet/README.md)
- [Everanalog EA3036C Datasheet Source Note](datasheets/everanalog-ea3036c-datasheet/README.md)
- [TI SN74LVC16244A Datasheet Source Note](datasheets/ti-sn74lvc16244a-datasheet/README.md)
- [TI OPA2356 Datasheet Source Note](datasheets/ti-opa2356-datasheet/README.md)
- [TI OPA2365 Datasheet Source Note](datasheets/ti-opa2365-datasheet/README.md)
- [TI TPS389x Datasheet Source Note](datasheets/ti-tps3896-datasheet/README.md)

## Reference Manuals

- [WCH CH32H417 Reference Manual Source Note](manuals/wch-ch32h417-reference-manual/README.md)

## Supplied Evidence

- [SHC252012 Electrical Characteristics](evidence/README.md)
