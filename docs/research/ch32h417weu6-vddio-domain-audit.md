# CH32H417WEU6 VDDIO 电源域核验

## 结论

当前网表和 WCH 原厂数据**不能给出 CH32H417WEU6 的 VDDIO 最大工作电流**。WCH 没有发布该封装、该软件配置、该 I/O 活动条件下的 VDDIO 最大工作电流；因此不能把电源器件的额定输出电流，也不能把 `VDDIO + VIO18` 的 `200 mA` 绝对最大值，当成 VDDIO 的设计负载。

能够严格确认的是：

- WEU6 的 VDDIO 域只有四个封装引脚：`13 XI`、`14 XO`、`20 PA5/ADC5/OP1_O1/DAC2`、`29 VDDIO`。
- 目标 `20 MS/s` 的 HSADC 表中，`I_DDIO = 1.8 mA typ.`；这不是最大值。HSADC 的 `I_DDA = 1.1 mA typ.` 从 `VDD33A` 域取电，不应加到 VDDIO 磁珠电流。
- 由网表电阻能计算的 VDDIO 域外部静态输出负载只在 `PA5`：DAC 输出为 `3.3 V` 时为 `31.2 uA`；按 WCH 的 `VDDIO = 3.6 V` 允许上限为 `35.1 uA`。这不是 MCU 的总 VDDIO 电流。
- 16 路数字采集、状态 LED、I2C 和 HSADC 管脚所属的高速 GPIO 均是 VIO18 域。它们的板级 1.8 V 电源来自独立降压支路，不经过 VDDIO 的两条 PCB 串联磁珠支路；MCU 内部 I/O LDO 是否正确关闭/隔离仍需由固件配置和 WCH 规则单独确认。

因此，对串联磁珠的正确陈述是：**公开资料不支持用一个“最大 VDDIO 工作电流”完成额定电流保证。** 但也不能再把 LDO 的 `300 mA` 额定输出当作 VDDIO 负载。

## 原厂引脚域清单

WCH 的 CH32H417WEU6 引脚图使用黑色标出 `@VDDIO power`。图中全部黑色引脚如下；没有其他 WEU6 管脚属于 VDDIO 域。[W1]

| QFN68 引脚 | 原厂名称 | 当前网表网络 | 当前连接和电流方向 |
| ---: | --- | --- | --- |
| 13 | `XI` | `XI` | 25 MHz 晶体的一端、20 pF 到地。为交流振荡节点，无电阻性 DC 负载。 |
| 14 | `XO` | `XO` | 同一晶体另一端、20 pF 到地。为交流驱动节点，无电阻性 DC 负载。 |
| 20 | `PA5/ADC5/OP1_O1/DAC2` | `DACFB` | 当前接 DAC2 输出，经 27 kOhm 进入 1.8 V 降压器的反馈网络。DAC 高电平时向该网络源出微安级电流。 |
| 29 | `VDDIO` | `VDDIO` | VDDIO 域的唯一电源输入。由 `AVCC` 经两条并联的 220 Ohm @100 MHz、50 mOhm 磁珠支路供电；并联 100 nF 和 4.7 uF 到地。 |

这里的“VDDIO 域引脚”指原厂引脚图的供电标识，不是把所有名称中带有 `ADC` 或 `GPIO` 的管脚都归入 VDDIO。尤其不能从功能名推断电源域。

## VDDIO 网络本身

`VDDIO` 网络上只有以下连接：

1. 两条从 `AVCC` 到 `VDDIO` 的并联磁珠供电路径。两条路径均装入 BOM，标称直流电阻均为 50 mOhm；DC 分流受公差、温度、走线电阻影响，不能把总电流机械地等分到单颗元件。
2. MCU 的 pin 29 电源输入。
3. 100 nF 与 4.7 uF 去耦电容到地。两者稳态 DC 电流为零，但在负载边沿和启动时提供/吸收瞬态电流。

所以，流经任一串联磁珠路径的电流是 MCU `VDDIO` 输入电流加上去耦电容充放电电流的分流结果；网表中没有另一个直接挂在 `VDDIO` 网络上的直流负载。

## VDDIO 引脚的外部负载计算

### XI 与 XO

网表为 XI/XO 配置一个 25 MHz、20 pF 负载规格的晶体，并在每个节点各接一个 20 pF 到地的电容。WCH 对这种晶体模式只给出 `I_HSE = 0.8 mA typ.` 的 **HSE 驱动电流**，未给该项的最大值，也未把它定义为 VDDIO 电源输入电流。因此该数值不能用于构造最大 VDDIO 预算。[W2]

晶体网络没有到电源或地的电阻性 DC 路径；外部提供方波模式下，XI 输入漏电流的绝对值最大为 `1 uA`，但当前网表不是该模式。[W2]

### PA5 / DAC2

PA5 接到 DAC2 的反馈注入网络。直流等效为：

- DAC 输出与反馈节点之间：`27 kOhm`；
- 反馈节点到 1.8 V：`100 kOhm`；
- 反馈节点到地：`100 kOhm`；
- 220 pF 电容在 DC 下开路。

对 PA5 看入该网络：

```text
VTH = 1.8 V x 100 kOhm / (100 kOhm + 100 kOhm) = 0.9 V
RTH = 100 kOhm || 100 kOhm = 50 kOhm
I_PA5 = (V_DAC2 - 0.9 V) / (27 kOhm + 50 kOhm)
```

| DAC2 输出条件 | PA5 对外静态电流 | 方向 |
| --- | ---: | --- |
| `3.3 V`（本设计标称） | `31.2 uA` | 从 PA5 源出至反馈网络 |
| `3.6 V`（WCH VDDIO 允许上限） | `35.1 uA` | 从 PA5 源出至反馈网络 |
| `0 V` | `11.7 uA` | PA5 从反馈网络灌入 |

原厂引脚图把 PA5 标为 VDDIO 域，故这给出了该管脚的外部负载上限。WCH 同时说明 `VDD33A` 为 DAC 模拟部分供电，但未在数据表中把 DAC2 输出级的电流明确分解为 VDDIO 或 VDD33A。因此，`35.1 uA` 应记录为 PA5 的已知外部管脚负载；它非常小，但不能在缺少该分解规格时声称为 VDDIO 总电流的完整一项。[W1][W3]

## 与其他电源域的分界

下表列出当前网表中容易被误归到 VDDIO 的负载。它们不应计入 VDDIO 串联磁珠的 PCB 电流预算。

| 当前功能和 MCU 管脚 | 原厂供电域 | 板上电流路径 | 是否流经 VDDIO 串联磁珠 |
| --- | --- | --- | --- |
| 16 路数字采集：`PD10..PD15`、`PF0..PF2`、`PC6..PC9`、`PA13..PA15` | VIO18 | 输入来自 16 位缓冲器；缓冲器 VCC、两个 VIO18 引脚和 I2C 上拉均在独立 `1V8` 网。 | 否，PCB 上由独立 1.8 V 降压支路供电。 |
| 两个状态 LED：`PB0`、`PB1` | VIO18 | LED 阳极经 4.7 kOhm 接 3.3 V；MCU 置低时，电流从 3.3 V 经 LED 流向 MCU 灌电流和地。 | 否。它是 VIO18 域管脚的灌电流和 3.3 V 外部支路负载。 |
| I2C：`PB10`、`PB11` | VIO18 | 各有 4.7 kOhm 上拉到 `1V8`。若两个管脚同时为低，外部上拉电流为 `2 x 1.8 / 4.7 k = 0.766 mA`，由 1.8 V 流向地。 | 否。 |
| HSADC 管脚：`PC0..PC3`，其中 `PC2/PC3` 接两路模拟前端 | VIO18（GPIO 管脚域） | HSADC 的数字/IO 项由数据表另列为 `I_DDIO`；模拟核心项为 `I_DDA`。 | `I_DDIO` 应按数据表归入 VDDIO，`I_DDA` 归入 VDD33A；不可由“PC0..PC3 是 VIO18 GPIO”忽略这两个转换器规格项。 |
| USB 2.0：`PB8/USB2DP/SWCLK`、`PB9/USB2DM/SWDIO` | VDD33 | 连到 USB D+/D- 与 ESD 网络。 | 否。 |
| USB 3.0 差分管脚 | VDD12A | USB 3.0 PHY 电源域。 | 否。 |
| HSADC 模拟核心、DAC 模拟部分 | VDD33A | `AVCC` 直接接 MCU 的 VDD33A；它和 VDDIO 磁珠支路在 `AVCC` 上游汇合。 | `I_DDA` 不流经 VDDIO 磁珠，但会加载 AVCC 上游稳压器。 |

原厂供电方案说明 VIO18 在默认模式下由 VDDIO 通过芯片内部 I/O LDO 产生；但当前网表把 VIO18 接在独立的 1.8 V 降压输出。网表只能证明这条外部 PCB 供电路径不经过 VDDIO 磁珠，不能证明 MCU 内部 LDO 已经按要求关闭或不存在反灌路径。该配置问题必须以实际固件的电源控制寄存器设置和 WCH 的外供 VIO18 规则另行确认。[W3][N1]

## 可用电流数据与不可计算项

| 项目 | WCH 数值 | 适用性 |
| --- | ---: | --- |
| HSADC `I_DDIO`，100 MHz HSADC 时钟、20 MS/s | `1.8 mA typ.` | 与项目采样速率直接相关；是典型值，不是最大值。 |
| HSADC `I_DDA`，同一条件 | `1.1 mA typ.` | 属于 VDD33A，不计入 VDDIO 磁珠。 |
| HSE 晶体模式 `I_HSE` | `0.8 mA typ.` | 驱动电流特性；不是发布的 VDDIO 输入电流最大值。 |
| 全部 VDDIO + VIO18 供电引脚合计 | `200 mA` absolute maximum | 绝对最大应力限制，且合并两个域；不是 VDDIO 的工作电流或推荐设计值。 |
| 任意 I/O 或控制引脚灌/源电流 | `+25/-25 mA` absolute maximum | 单引脚绝对最大，不可作为长期负载目标，亦不可相加成 VDDIO 电流。 |

WCH 还明确说明，供电电流取决于电压、温度、I/O 负载、软件配置、工作频率、翻转速率和执行代码。这正是不能从“20 MS/s”单一条件推导全芯片 VDDIO 最大值的原因。[W4]

**可证明的数值边界：**

- PA5 的外部 DC 负载：最大 `35.1 uA`（以 `VDDIO = 3.6 V` 和现有阻值计）。
- HSADC 20 MS/s 的已公布 VDDIO 项：`1.8 mA typ.`。
- 当前网表、WCH 数据表和元件额定组合不能推出 `I_VDDIO,max,operating`；因此也不能据此正式保证某颗磁珠的额定电流覆盖所有软件、温度和 I/O 组合。

将 `1.8 mA`、`0.8 mA` 和 PA5 的微安级负载直接相加并命名为“最大电流”是不成立的：前两项均为典型值，测试/启用条件与互相包含关系未发布，且芯片其余 VDDIO 消耗未给出上限。

## 对首板量测的影响

若要让元件额定覆盖变成可验证结论，必须在 MCU VDDIO 输入及两条磁珠支路上实测，而不是以稳压器额定替代：

1. 目标固件下同时启用连续 20 MS/s HSADC、16 路数字采集最大翻转活动、HSE、USB 2.0/3.0 的实际使用组合。
2. 测量 VDDIO 引脚附近的总电流及每条并联供电路径的电流；两条路径不应假设均分。
3. 同步测量 MCU VDDIO 与 VDD33A/VREFP 的瞬态电压。HSADC 条件要求 `VDD33A >= VDDIO`，且 `VREFP <= VDD33A`；不能只以平均 DC 压降判定。[W5]

## 来源

- **[W1]** WCH, [CH32H417DS0 V1.8 官方数据表](https://www.wch-ic.com/downloads/CH32H417DS0_PDF.html), section 2.1.1, CH32H417WEU6 pin arrangement and legend. Project mirror: [pin arrangement](/Users/ivan/Projects/Ivan/ch32h417weu6-logic-analyzer/docs/datasheets/wch-ch32h417-datasheet/wch-ch32h417-datasheet.md:920), [GPIO rail definitions](/Users/ivan/Projects/Ivan/ch32h417weu6-logic-analyzer/docs/datasheets/wch-ch32h417-datasheet/wch-ch32h417-datasheet.md:910).
- **[W2]** Same WCH data sheet, [external clock specifications](/Users/ivan/Projects/Ivan/ch32h417weu6-logic-analyzer/docs/datasheets/wch-ch32h417-datasheet/wch-ch32h417-datasheet.md:1287), including table 3-11.
- **[W3]** Same WCH data sheet, [CH32H417 power scheme](/Users/ivan/Projects/Ivan/ch32h417weu6-logic-analyzer/docs/datasheets/wch-ch32h417-datasheet/wch-ch32h417-datasheet.md:276), which assigns analog blocks to VDD33A, conventional I/O to VDDIO, and high-speed I/O to VIO18.
- **[W4]** Same WCH data sheet, [absolute maximum and supply-current qualification](/Users/ivan/Projects/Ivan/ch32h417weu6-logic-analyzer/docs/datasheets/wch-ch32h417-datasheet/wch-ch32h417-datasheet.md:1176), [supply-current factors](/Users/ivan/Projects/Ivan/ch32h417weu6-logic-analyzer/docs/datasheets/wch-ch32h417-datasheet/wch-ch32h417-datasheet.md:1207).
- **[W5]** Same WCH data sheet, [HSADC characteristics](/Users/ivan/Projects/Ivan/ch32h417weu6-logic-analyzer/docs/datasheets/wch-ch32h417-datasheet/wch-ch32h417-datasheet.md:1683), table 3-52.
- **[N1]** [Current netlist, VDDIO and MCU power-pin connections](</Users/ivan/Downloads/Netlist_Schematic1_2026-08-25.enet:5141>), [crystal network](</Users/ivan/Downloads/Netlist_Schematic1_2026-08-25.enet:2368>), [DAC feedback resistor network](</Users/ivan/Downloads/Netlist_Schematic1_2026-08-25.enet:920>), and [parallel VDDIO feed](</Users/ivan/Downloads/Netlist_Schematic1_2026-08-25.enet:1515>).
