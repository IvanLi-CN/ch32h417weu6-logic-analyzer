# CH32H417WEU6 `VDDIO` 电流预算与 GZ1005D121TF 复核

## 结论

前一轮用 `TPS7A2033PDBVR` 的 `300 mA` 输出能力作为 L4 电流，**是不成立的**。那是电源的供电能力，不是本项目 `VDDIO` 网络的负载电流；它不能据此否决 `GZ1005D121TF`。

按 2026-08-25 网表，`VDDIO` 上唯一的主动负载是 MCU `U5` 的 `VDDIO` 引脚。HSADC 以 `20 MS/s` 运行时，WCH 仅公布该域 `I_DDIO = 1.8 mA` 的**典型值**；没有公布 CH32H417WEU6 在此项目软件、所有 GPIO 状态及温度范围内的 `VDDIO` 最大工作电流。因此，无法从数据手册把 `200 mA` 磁珠额定转换成“已经通过保证”的结论，也不能用 LDO 的 `300 mA` 直接作硬拒绝。

`GZ1005D121TF` 对 L4 的正确状态是：**不能由现有公开规格保证放行，也不能仅由 `300 mA` 判为不可用。**若它成为唯一串联供电件，应在首板以目标的 `20 MS/s` HSADC、实际固件和 I/O 活动量测 `VDDIO` 电流及 MCU 引脚处 `VDDIO - VREFP`。如果网表中 L4 和 L5 均装配，它们是并联支路，L4 单颗电流更不能直接等同于整个 `VDDIO` 电流。

## 网表实际负载

对 `/Users/ivan/Downloads/Netlist_Schematic1_2026-08-25.enet` 的所有引脚逐项枚举，连接到 `VDDIO` 的器件只有：

| 器件 | 连接 | 对直流电流预算的意义 |
| --- | --- | --- |
| `U5` `CH32H417WEU6` | 29: `VDDIO` | **唯一主动负载**；其内部普通 I/O 域和内部 I/O LDO 的输入电流。 |
| `C64` `100 nF` | `VDDIO` 至 `GND` | 去耦；稳态不构成 DC 负载，瞬态时提供/吸收电荷。 |
| `C65` `4.7 uF` | `VDDIO` 至 `GND` | 去耦；稳态不构成 DC 负载，瞬态时提供/吸收电荷。 |
| `L4` | `AVCC -> VDDIO` | 供电支路，不是负载。 |
| `L5` | `AVCC -> VDDIO` | 第二条同端点供电支路；若两个 BOM 件均装配，则与 L4 并联。 |

`U5` 的两个 `VIO18` 引脚（28、66）都接到独立的 `1V8` 网络，而非 `VDDIO`。因此，在此网表的电源拓扑中，外部 `VIO18` 负载不通过 L4。这个判断只来自连接关系；外部驱动 `VIO18` 的上电/内部 LDO 状态另有已知的 WCH 规格边界，不能从该结论推导其启动安全性。[N1][W1]

**网表异常/待确认项：**L4 和 L5 都是 `MPZ1608S221ATA00`、`AVCC -> VDDIO`，并且两者均为 `Add into BOM: yes`。若这是有意的，两颗是并联；若 L5 是复制残留或 DNP 意图，必须在原理图/BOM 中明确。本文分别给出“单颗 GZ”与“L4/L5 均在位”的影响，但不擅自假定其中一颗不存在。

## WCH 电流规格的正确含义

### HSADC 的可用典型值

WCH 表 3-52（HSADC 电气特性）在 `20 MS/s`、`f_HSADC = 100 MHz` 的同一表中列出：

| 参数 | 含义 | 值 | 可否作为保证上界 |
| --- | --- | ---: | --- |
| `I_DDA` | HSADC 供电电流（`VDD33A` 域） | `1.1 mA typ.` | 否，只有典型值。 |
| `I_DDIO` | HSADC I/O 引脚电流（`VDDIO` 域） | `1.8 mA typ.` | 否，只有典型值。 |
| `f_S` | HSADC 采样率 | `20 MHz typ.` | 表明本项目的 20 MS/s 目标所在工作点；不等价于全部芯片电流规格。 |

因此，对 L4 最直接、且与 20 MS/s 对应的已公布量是 `1.8 mA typ.`。`1.1 mA` 从 `VDD33A` 取电，不应再加到 L4 的 `VDDIO` DC 电流中，尽管两者都会加载 L4 上游的 `AVCC`。

### `200 mA` 不是工作预算

WCH 表 3-1 将 `I_VIO = 200 mA` 写在**绝对最大值**中，定义为所有 `VDDIO` 与 `VIO18` 供电引脚的合计总电流。绝对最大值是不可超越的应力边界，不是芯片在某个工作模式下的保证或推荐供电电流。

此外，该数值合并了两个供电域；本项目的 `VIO18` 外接到 `1V8`，不经过 L4。因此它既不能当作 L4 的实际负载，也不能当作 `VDDIO` 的保证最大工作电流。它只说明：即使在极端应力解释下，芯片两个 I/O 供电域的合计也不得跨越 `200 mA`。

WCH 没有在数据手册中给出单独的 `VDDIO` 总工作电流最大值，亦没有给出与“HSADC 20 MS/s + 本项目 16 路数字输入”的组合电流上界。故不应虚构一个可计算的保证最坏值。

## GZ1005D121TF 的直流压降

顺络原厂规格给出 `GZ1005D121TF`：`DCR <= 0.40 Ohm`、额定电流 `200 mA`、`120 Ohm +/-25% @ 100 MHz`。[S1]

### 单颗作为唯一 L4

稳态电阻压降为 `Delta V = I_VDDIO x R_DCR`：

| 电流取值 | 依据 | 最大 DC 压降（`0.40 Ohm`） | 结论 |
| --- | --- | ---: | --- |
| `1.8 mA typ.` | WCH 表 3-52 的 HSADC `I_DDIO`，20 MS/s | `0.72 mV` | HSADC 已公布典型电流本身远低于磁珠额定。 |
| `200 mA` | 磁珠额定电流；也是 WCH 合并 I/O 域绝对最大值的数值，**不是**本项目工作电流 | `80 mV` | 磁珠处在额定上限；不能将此视为设计余量。 |
| `300 mA` | TPS7A20 的输出能力，**不是负载边界** | `120 mV` | 数学上成立，但不可用于判定 MCU 的实际或保证工作点。 |

若暂只检查绝对电压，TPS7A2033 的最低标称精度输出 `3.3 x (1 - 1.5%) = 3.2505 V`，则上述三点忽略走线和动态跌落后分别是 `3.24978 V`、`3.1705 V`、`3.1305 V`。其中第三行不是允许的设计电流；第二行也不是 WCH 的正常工作预算。该计算不能替代引脚处电压测量。

### L4、L5 均装配

当前网表明确有两个完全相同端点的磁珠。若 L4 改为 GZ、L5 仍为现有 TDK `50 mOhm`，两支路的 DC 分流近似按电阻倒数决定：

`I_L4 / I_total approx. 0.05 / (0.40 + 0.05) = 11.1%`

这只适用于 DC、两条走线电阻可忽略且两颗都装配；高频分流由阻抗曲线、DC bias、去耦和布局决定，不能使用这个式子。若两颗都换为相同 GZ，则各自近似分担一半 DC 电流，等效 DCR 约 `0.20 Ohm`。这些并联事实进一步说明：以 U8 的 `300 mA` 直接判定单颗 L4 过流，既不代表 MCU 负载，也不匹配当前网表。

## 与 HSADC 关系及首板判据

目标是 HSADC `20 MS/s`，不是以 `200 MHz` 数字翻转频率推导 L4 电流。WCH 对 HSADC 的供电关系要求 `VDD33A >= VDDIO`，且 `VREFP <= VDD33A`；表 3-52 同时给出 HSADC 工作时 `VDDIO` 下限为 `VREFP`。[W1]

在本项目 WEU6 封装中，未引出的 `VREFP` 内部等于 `VDD33A`。而 L4/L5 把 `AVCC` 串联后送至 `VDDIO`，任何正向 `VDDIO` 电流都会产生非零的 `VDD33A/VREFP` 到 `VDDIO` 差。因此，即使 GZ 在 `1.8 mA typ.` 时仅约 `0.72 mV` DC 压降，串联拓扑也**不能仅凭原理图**保证 `VDDIO >= VREFP`；它需要实测。

可接受 GZ 作为 L4 的最低验证条件是：

1. 在 MCU `VDDIO` 引脚/最近去耦焊盘测量串联支路电流，运行目标固件、HSADC 连续 `20 MS/s`、数字输入实际最大活动量、以及 `VIO18` 切换状态。
2. 用差分探头在同一状态直接测 `VDDIO - VREFP`。HSADC 启用期间最小值不得小于 `0 V`。
3. 确认 WCH 要求的主 `VDDIO` 本地去耦为 `0.1 uF + 至少 10 uF` 的**有效**电容。当前网表的 C64 `100 nF` + C65 `4.7 uF` 标称值尚未达到该建议的 10-uF 标称量级。

在这些验证完成前，结论应表述为“GZ 与 20 MS/s 的已公布典型 HSADC 电流相容，但未获 WCH 最大 `VDDIO` 电流规格保证”，而不是“可保证”或“由 U8 的 300 mA 证伪”。

## 来源

- **[N1]** [Netlist_Schematic1_2026-08-25.enet](</Users/ivan/Downloads/Netlist_Schematic1_2026-08-25.enet>)：`U5` pin 29 = `VDDIO`；pins 28/66 = `1V8`；C64/C65、L4/L5 的 `VDDIO` 连接。网表 JSON 的组件键：`gge111`、`gge234_9`、`gge233_8`、`gge226`、`gge226_1`。
- **[W1]** WCH [CH32H417DS0 V1.8](https://www.wch-ic.com/downloads/CH32H417DS0_PDF.html)，第 1.4.3 节、表 3-1、表 3-2、表 3-52；本地镜像的对应摘录见 [wch-ch32h417-datasheet.md](/Users/ivan/Projects/Ivan/ch32h417weu6-logic-analyzer/docs/datasheets/wch-ch32h417-datasheet/wch-ch32h417-datasheet.md:280)、[绝对最大值](/Users/ivan/Projects/Ivan/ch32h417weu6-logic-analyzer/docs/datasheets/wch-ch32h417-datasheet/wch-ch32h417-datasheet.md:1176)、[工作条件](/Users/ivan/Projects/Ivan/ch32h417weu6-logic-analyzer/docs/datasheets/wch-ch32h417-datasheet/wch-ch32h417-datasheet.md:1183)、[HSADC 表](/Users/ivan/Projects/Ivan/ch32h417weu6-logic-analyzer/docs/datasheets/wch-ch32h417-datasheet/wch-ch32h417-datasheet.md:1686)。
- **[S1]** Sunlord [GZ Series Multilayer Chip Ferrite Bead](https://www.sunlordinc.com/uploads/files/20221115/GZ%20series%20of%20Multilayer%20Chip%20Ferrite%20Bead.pdf)，GZ1005 规格表。
- **[T1]** TI [TPS7A20 data sheet](https://www.ti.com/lit/ds/symlink/tps7a20.pdf)，`TPS7A2033` 输出精度 `+/-1.5% max`。
