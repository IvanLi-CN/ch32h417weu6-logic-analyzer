# USB-C 烧录转接板

> 当前有效规范以本文为准；实现覆盖与当前状态见
> `./IMPLEMENTATION.md`，关键决策理由见 `./HISTORY.md`。

## 背景 / 问题陈述

CH32H417WEU6 将 USB2 D+、D- 分别复用到 `PB8/SWCLK` 与
`PB9/SWDIO`。主板 USB-C 口是固定 UFP/Sink，正常连接 USB 主机时，
主机 USB2 PHY 与 WCH-Link 两线 SDI 下载接口不能共同连接到这对引脚。

本规格定义一个独立烧录转接板。它在烧录期间将目标板 USB-C 口变成受控的
5 V 电源入口和 WCH-Link SDI 接口；拔除后，主板 USB-C 口恢复为普通
USB 3.2 Gen 1 设备口。

## 目标 / 非目标

### Goals

- 不修改主板，不增加主板下载排针、0 Ohm 跳线或 USB2 信号切换器。
- 通过目标板现有 USB-C 口完成供电和 WCH-Link 两线 SDI 下载。
- 让正常 USB 主机数据路径与烧录路径在物理上互斥。
- 将 USB-C CC、电源和烧录信号的职责分离，避免形成 USB 数据透传路径。

### Non-goals

- 不支持在插入烧录转接板时进行正常 USB 数据通信、USB 枚举或 USB 3
  SuperSpeed 传输。
- 不实现 USB PD 的电压协商，不从输入端请求高于 5 V 的 VBUS。
- 不向主板提供 WCH-Link 的 3.3 V 或 5 V 下载器电源输出。
- 不更改主板的 HD3SS3220 固定 UFP/Sink 配置或 USB-C 电源路径。

## 范围（Scope）

### In scope

- 一块独立小板，包含一个 USB-C 母座、一个 USB-C 公头和一个
  WCH-Link 下载器连接器。
- USB-C 母座的 5 V 输入和 CC Sink 检测。
- USB-C 公头的 5 V Source 输出、CC Source 宣告、过流/反灌保护和
  WCH-Link 信号转接。
- 工装自身和目标板的上电、烧录、拔除验收。

### Out of scope

- 主板 EDA 工程的改动。
- 目标板运行时通过该工装进行 USB 传输。
- WCH-Link 本体、USB 线和外部电源适配器的选型。

## 需求（Requirements）

### MUST

- 转接板必须只有如下唯一的烧录信号映射：

  ```text
  target USB-C D+ -> WCH-Link SWCLK -> CH32 PB8
  target USB-C D- -> WCH-Link SWDIO -> CH32 PB9
  target GND      -> WCH-Link GND
  ```

- USB-C 母座必须仅作为工装的 5 V 电源输入（Sink）。其 USB2 D+/D-、
  SuperSpeed 对和 SBU 不得连接到 USB-C 公头或 WCH-Link。
- USB-C 公头必须作为目标板的普通 5 V Source；其 CC 实现必须符合
  Source 角色的 attach/current-advertisement 规则。
- 公头 VBUS 必须来自母座的 5 V 输入，经具备反灌保护和限流能力的
  电源路径后输出。不得将两个端口的 VBUS 直接短接。
- 公头输出启用前，工装必须已确认母座端存在合规 5 V Source，且输入端
  可供给的电流不小于公头对目标板宣告和允许的电流。
- WCH-Link 连接器必须提供 `GND`、`SWCLK` 和 `SWDIO`；不得把
  WCH-Link 的 `3.3V` 或 `5V` 电源输出接到目标板或工装 VBUS。
- 在目标板未上电、转接板 VBUS 未启用或未连接 WCH-Link 的情况下，
  `SWCLK` 和 `SWDIO` 不得经保护结构反向给目标板供电。实现必须使用
  经验证的高阻隔离/上电时序，而不能假设下载器输出天然高阻。
- D+ 与 D- 在 USB-C 公头的双面触点上必须按 USB2 同名网络正确并接，
  以支持任一插入方向。

### SHOULD

- 母座输入应接受固定 5 V、至少 Medium-current 宣告的 Type-C Source；
  工装应拒绝或保持公头 VBUS 关闭，直至输入能力满足输出策略。
- 公头 VBUS 的稳态电流能力应覆盖目标板的启动和 WCH 下载；若希望在
  工装连接时运行完整采集负载，必须覆盖主板定义的 1.10 A 最大预算。
- WCH-Link 下载器连接器应有明确丝印，至少标注 `GND`、`CLK`、`DIO` 与
  插接方向；不得出现会诱导连接目标 3.3 V 的电源针脚标注。
- 应为公头 VBUS、WCH-Link `SWCLK` 与 `SWDIO` 保留可测量测试点。

### COULD

- 可增加仅表示“输入有效/公头 VBUS 已使能”的状态指示，不得把该指示
  作为安全控制的唯一依据。
- 可增加机械外壳或定位结构，防止使用者把转接板误当作普通 USB 数据转接器。

## 功能与行为规格

### Core flows

1. 外部 Type-C 5 V Source 插入工装母座。工装以 Sink 角色完成 CC attach，
   检查输入电压与可用电流。
2. 输入满足要求后，工装以 Source 角色在公头向目标板提供受保护的 5 V。
   目标板沿现有 `VBUS_RAW -> TPS25200 -> VBUS_5V_PROTECTED` 路径上电。
3. 目标板电源稳定后，WCH-Link 经 `SWCLK`/`SWDIO` 连接，执行两线 SDI
   下载。USB2 线不连接到任何 USB Host PHY。
4. 烧录结束后，先停止 WCH-Link 会话，再拔除转接板。目标板重新连接普通
   USB-C 主机时，USB2 和 USB3 路径按主板原设计工作。

### Edge cases / errors

- 母座端未检测到合规 Source、输入电流能力不足、输入过压/欠压或输出过流时，
  公头 VBUS 必须关闭；不得仅传递一个不受控的 VBUS。
- 只连接 WCH-Link 而未给转接板供电时，目标板不得通过烧录信号出现部分上电。
- 烧录转接板插入时，目标板固件即使初始化 USB，也不构成正常 USB 链路；
  WCH-Link 信号是此状态下唯一允许的数据活动。
- 任何端口反接、热插拔或输入源移除都不得让公头向母座反向供电。

## 接口契约（Interfaces & Contracts）

### 接口清单（Inventory）

| 接口 | 类型 | 范围 | 变更 | 契约文档 | 负责人 | 使用方 | 备注 |
| --- | --- | --- | --- | --- | --- | --- | --- |
| USB-C 输入母座 | 硬件 | external | New | 本文 | 硬件 | 外部 5 V Source | 仅电源与 CC Sink |
| USB-C 目标公头 | 硬件 | external | New | 本文 | 硬件 | 逻辑分析仪主板 | 5 V Source 与 SDI 复用 |
| WCH-Link 下载器接口 | 硬件 | external | New | 本文 | 硬件 | WCH-Link | `GND`/`SWCLK`/`SWDIO`，无供电回路 |

### 信号契约

| 工装端口 | 信号 | 连接 | 禁止连接 |
| --- | --- | --- | --- |
| USB-C 输入母座 | VBUS/GND/CC | 电源输入、CC Sink 控制 | D+/D-、SBU、SuperSpeed 到任何其他端口 |
| USB-C 目标公头 | VBUS/GND/CC | 受保护 5 V Source、CC Source 控制 | 输入母座的数据触点 |
| USB-C 目标公头 | D+/D- | WCH-Link `SWCLK`/`SWDIO` | USB Host PHY、输入母座 D+/D- |
| WCH-Link | GND/CLK/DIO | 目标公头 GND/D+/D- | 目标或工装的 3.3 V/5 V 电源网络 |

## 验收标准

- Given 工装母座连接满足输入策略的 5 V Type-C Source，
  When 工装公头插入未上电的目标板，
  Then 目标板经其现有 USB-C 电源路径正常启动，且 WCH-Link 电源针脚未参与供电。

- Given 目标板已稳定上电且工装连接 WCH-Link，
  When 执行 WCH-Link 两线 SDI 下载，
  Then 可稳定擦除、写入、校验并复位 CH32H417WEU6。

- Given 工装接在目标板上，
  When 观察 D+/D-，
  Then 它们只与 WCH-Link 相连，不与 USB Host PHY 或母座 USB2 引脚导通。

- Given 输入 Source 被移除、输入能力不足或输出过流，
  When 保护状态触发，
  Then 公头 VBUS 关闭且不存在向输入端或 WCH-Link 的反向供电。

- Given 拔除工装并把普通 USB-C 数据线接到目标板，
  When 执行正常功能验证，
  Then USB2/USB3 枚举和数据传输不因该工装设计而受影响。

## 验收清单

- [x] 主板不改动、外置烧录转接板复用 USB2/SWD 的长期边界已明确。
- [x] 电源、CC、WCH-Link 和 USB2 信号契约已明确。
- [x] 反灌、输入能力不足和 USB 数据误连边界已覆盖。
- [x] 已定义可用于原理图、PCB 和样机 review 的验收条件。

## 非功能性验收 / 质量门槛

### Hardware validation

- 原理图 review：确认不存在输入母座 D+/D- 到目标公头的路径，且没有
  WCH-Link 电源针脚到目标电源网络的路径。
- PCB review：确认 USB2 同名触点并接、`SWCLK`/`SWDIO` 映射无交叉、
  VBUS 保护器件位于公头电源路径内。
- 样机测试：测量启动浪涌、烧录期间 VBUS、输出限流、反灌、两种公头插入方向
  和 WCH-Link 全擦除/回读校验。

### Visual Evidence

PR: none

## 风险 / 开放问题 / 假设

- 待实现时选择双端口 Type-C 电源角色控制和 VBUS 保护器件；不得以被动
  VBUS 短接或单个 CC 电阻替代该功能。
- 目标板完整采集模式的 1.10 A 预算适用于选择允许完整运行的工装版本；
  仅烧录版本仍须实测启动和下载峰值电流后确定最低输入宣告。
- WCH-Link 信号在未供电目标上的实际高阻行为未作为安全假设，必须由转接板
  电路或实测验证覆盖。

## 参考

- [CH32H417 USB2/SWD 物理引脚分配](../../research/hd3ss3220-i2c-gpio-allocation-evidence.md)
- [主板电源架构](../../research/power-architecture.md)
- [HD3SS3220 固定 UFP/Sink 选择](../../research/hd3ss3220-ufp-selection.md)
- [TI HD3SS3220 data sheet](https://www.ti.com/lit/ds/symlink/hd3ss3220.pdf)
- [WCH CH32H417 data sheet](https://www.wch-ic.com/downloads/CH32H417DS0_PDF.html)
