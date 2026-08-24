# Documentation Index

## Current Decision

- [Hardware Selection](SELECTION.md): current component and architecture
  choices, open decisions, and the required prototype validation.

## Research

- [Project Requirements](research/project-requirements.md): confirmed capture,
  voltage-domain, and layout constraints.
- [CH32H417 Package Comparison](research/ch32h417-package-comparison.md):
  hardware comparison of WEU6 and QEU6 and the reason WEU6 is selected.
- [Analog Front End](research/analog-front-end.md): OPA2356 and OPA2365
  comparison for the optional analog capture path.
- [EA3036C Inductor Selection](research/ea3036-inductor.md): rationale for
  the 1.5 uH output inductor.
- [CH32H417 Power Sequencing](research/ch32h417-power-sequencing.md): MCU
  supply constraints and EA3036C EN startup control.
- [Power Architecture](research/power-architecture.md): selected 3.32 V and
  1.218 V rails, external-`VIO18` evidence boundary, TPS25200 conditional
  values, and the hardware-only HD3SS3220 power-on controller.
- [TPS25200 Peripheral Selection](research/tps25200-peripherals.md):
  project-specific capacitors, current-limit resistor, enable, fault, TVS,
  and layout choices.
- [Logic Capture and HSADC Layout Rules](research/logic-adc-layout.md):
  project-specific routing, length-matching, analog placement, and prototype
  validation rules.
- [External VIO18 Official Evidence](research/vio18-external-handover-official-evidence.md):
  WCH-only evidence defining what is and is not established for external
  `VIO18` operation.

## Datasheets

- [WCH CH32H417 Datasheet Source Note](datasheets/wch-ch32h417-datasheet/README.md)
- [Everanalog EA3036C Datasheet Source Note](datasheets/everanalog-ea3036c-datasheet/README.md)
- [TI SN74LVC16244A Datasheet Source Note](datasheets/ti-sn74lvc16244a-datasheet/README.md)
- [TI OPA2356 Datasheet Source Note](datasheets/ti-opa2356-datasheet/README.md)
- [TI OPA2365 Datasheet Source Note](datasheets/ti-opa2365-datasheet/README.md)
- [TI TPS389x Datasheet Source Note](datasheets/ti-tps3896-datasheet/README.md)

## Supplied Evidence

- [SHC252012 Electrical Characteristics](evidence/README.md)
