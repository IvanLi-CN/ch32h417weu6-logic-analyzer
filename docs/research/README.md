# Research Index

- [Power Architecture](power-architecture.md): current EA3036C channel map:
  CH1 `3V3`, CH2 external `VIO18`, and CH3 `1V2`, plus the local EN network.
- [CH32H417 Power Sequencing](ch32h417-power-sequencing.md): WCH rail
  constraints and the independent EA3036C EN timing boundary.
- [CH32H417WEU6 HSADC and VIO18 Operating Constraints](ch32h417weu6-hsadc-vio18-constraints.md):
  normative HSADC steady-state rails: fixed 3.32-V `VDD33A`/`VREFP`/`VDDIO`
  and selectable 1.80/2.50/3.00-V `VIO18`.
- [CH32H417QEU6 参考工程的模拟与 I/O 供电拓扑](ch32h417-qeu6-reference-power-topology.md):
  QEU6 reference project's LDO and ferrite-bead topology, WCH HSADC supply
  constraints, and the pin-level validation boundary.
- [CH32H417WEU6 外部 VIO18 复位行为与 EA3036C 使能策略](ch32h417-vio18-en-gated-dac-assessment.md): WCH 官方资料证明 PA5/DAC2 的高阻复位状态；当前 EN2/EN3 使用 `3V3` RC，不采用 MCU、PGOOD 或级联控制。
- [SN74LVC16244A 输入网络与 200 MS/s 评估](sn74lvc16244a-input-network-200mhz.md): 当前 1.8-V 输入网络的 `OE#` 直接接地、弱下拉、串联阻尼、ESD 与采样边界。
- [LMHaley CH32H417 USB3.0 逻辑分析仪采集记录](oshwhub-aaahaley-ch32h417-usb3-logic-analyzer.md): 改进版工程的页面资料、软件与硬件改动、评论和复用边界。
- [CH32H417 Logic Analyzer Firmware Sources](ch32h417-firmware-sources.md): public firmware candidates, WEU6 fit, IAP/host workflows, and porting risks.
