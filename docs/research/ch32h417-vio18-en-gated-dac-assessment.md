# CH32H417WEU6 外部 VIO18 复位行为与 EA3036C 使能策略

## 结论

不采用 MCU 控制任何 EA3036C 使能脚，也不分配 `VIO18_EN` GPIO。当前板的
电源与使能拓扑为：

```text
EA3036C CH1 / 3V3  -> VDD33, VDD33A, VDDIO, HD3SS3220.VCC33
EA3036C CH2 / VIO18 -> CH32 VIO18, SN74LVC16244A.VCC, I2C pull-ups
EA3036C CH3 / 1V2  -> VDDK, VDD12A

VDD5 -------------------------------- EN1
3V3 --- R5 10 kOhm ---+-------------- EN2
                       |
                     C15 100 nF
                       |
                      GND
3V3 --- R6 10 kOhm ---+-------------- EN3
                       |
                     C16 100 nF
                       |
                      GND
```

这实现了明确的设计目标：CH1 先从 `VDD5` 启动；CH2 和 CH3 只在 `3V3`
开始建立后才经各自 RC 获得使能。两路 RC 相同，故刻意**不**规定 CH2 和
CH3 的相对先后。该策略不需要 MCU、PGOOD、故障互锁或 CH2 到 EN3 的级联。

WCH 没有规定这三路必须存在固定上电顺序。因此，以上是当前需求下正确的
使能架构。它不等同于“具有电源良好语义”或“官方保证 CH1 已完全稳定”的
绝对时序承诺；后者不是本项目的已定义需求，且 EA3036C 数据表没有给出可用
来证明它的最坏值参数。

## 为什么使用 `3V3` 作为 RC 源

`EN2` 和 `EN3` 若改由 `VDD5` 上拉，其 RC 会在 CH1 的输出尚未建立时就开始
充电，不能表达“先建立 3V3，再释放后两路”的目标。当前 `3V3` 上拉则天然
将两路延迟的起点置于 CH1 输出开始上升之后。

`EN3` 不应改由 CH2 或 `VIO18` 上拉。当前没有“CH2 必须已稳压才允许 CH3
启动”的系统需求；这种连接只会额外引入 CH2 故障、负载和启动波形对 CH3 的
依赖。若未来出现这项独立需求，应按该需求设计真实的状态互锁，不能把一个
`VIO18` RC 节点误称为 PGOOD。

## EA3036C EN 证据与时间边界

| 资料 | 已给出的事实 | 对当前方案的含义 |
| --- | --- | --- |
| EA3036C Ver. 1.4，第 2 页 Pin Description | `EN1`、`EN2`、`EN3` 分别控制三路 Buck；EN1 和 EN3 明确不得悬空。 | 三路使能是独立输入，当前每一路都有确定连接。 |
| 第 5 页 Electrical Characteristics | 仅给出 `V_EN-L <= 0.4 V` 与 `V_EN-H >= 1.4 V`。 | 没有 EN 判定阈值的完整范围、迟滞、输入漏电或输入电容。 |
| 第 11 页 Enable Control | 每路可由独立 EN 控制；所示编程 RC 为 `VINx -> 100 kOhm -> ENx`，`ENx -> 10 nF` 至 `100 nF -> GND`。 | 支持使用独立被动 RC 产生启动延迟；没有 RC 到输出稳定时间的公式或保证。 |

当前 `R5/C15` 与 `R6/C16` 的名义时间常数都是 `10 kOhm x 100 nF = 1 ms`。
这记录的是电路常数，不是“必定等待 1 ms”或“必定等待至 3V3 稳定”的官方
保证：EA3036C 没有给 EN 阈值上界，也没有给 CH1 的最大启动/稳定时间。
`10 kOhm` 也不同于第 11 页图示的 `100 kOhm`，资料没有给出其允许范围或
用于换算延迟的公式。

因此，不应把当前 RC 写成经过器件数据表认证的 PGOOD 延时；也无需据此增加
MCU 门控。若未来规格新增“CH1 必须在最坏情况下完全稳定后才能启动其余两路”，
则该规格需要额外的、具有最坏值时序指标的电源时序/状态方案。它不是 WCH
对当前三路供电的要求。

## PA5/DAC2 复位事实

PA5 的复位配置**就是高阻输入状态**：WCH `CH32H417RM` v1.7 的
`GPIOA_CFGLR = 0x44444444`，结合 GPIO 配置编码，PA5 为
`CNF5=01b`、`MODE5=00b` 的浮空输入。DAC2 的 `DAC_CTLR` 复位为零，
`EN2=0`，输出缓冲不使能。

这证明复位期间 MCU 不会主动把 PA5 拉高或拉低，也不会输出固件选择的 DAC
电压。高阻输入并不宣称零漏电、零电容或替代外部反馈网络的偏置；它的结论
仅是：MCU 不会在复位时主动选择一个 VIO18 电压。DAC2 禁用时的默认 VIO18
状态仍由 CH2 的外部反馈网络决定，不能靠增加 MCU 到 EN 的门控来改善。

| 问题 | 结论 |
| --- | --- |
| 是否需要 PB3 或其他 GPIO 控制 EN2/EN3 | 否。这样不会增加 PA5 高阻复位态的确定性，反而把电源可用性绑定到 MCU 复位和软件状态。 |
| 是否需要 EN 下拉或 MCU 上电顺序 | 否。现有 RC 已给 EN2/EN3 确定的被动状态。 |
| PA5/DAC2 是否在复位时主动选择错误电压 | 否。PA5 是浮空输入，DAC2 禁用。 |
| 是否可由这组 WCH 复位寄存器推导独立外部 VIO18 的完整交接规范 | 否。那是独立的外部 VIO18/内部 I/O LDO 资料边界。 |

## 软件边界

固件不配置任何 EA3036C 使能 GPIO。系统上电后，固件只在需要选择 VIO18
目标值时将 PA5 配置为模拟输入、写入 DAC2 数据并按 DAC 章节配置后使能 DAC2。
WCH 表 3-54 的 `t_SETTLING <= 4 us` 仅在表中所列负载条件下适用于 DAC 输出
到 `+/-1 LSB`；它不是外部 Buck 反馈环路稳定时间的保证。

WCH 对独立 1.80/2.50/3.00-V 外部 `VIO18` 与内部 I/O LDO 的 POR、NRST、
brownout 和电压切换交接没有公开应用电路或时序。这个边界不构成把 EN2/EN3
改为 MCU 控制的理由；它单独记录在
[外部 VIO18 DAC 控制官方分析](ch32h417-external-vio18-dac-control-official-analysis.md)。

## 来源与归档

| 标识 | 文档与位置 | 稳定来源 |
| --- | --- | --- |
| WCH-RM | `CH32H417RM` v1.7：§2.4.1 p7，§9.2.2 与 §9.3.1.1 pp136--137，第 18 章（尤其 §18.2.2、§18.4.1）。 | [WCH 下载页](https://www.wch-ic.com/downloads/CH32H417RM_PDF.html) |
| WCH-DS | `CH32H417DS0-EN` v1.8：§1.4.3 p8，§1.4.45 p24，§2.1.1 p27，§3.3.32 Table 3-54 pp134--135。 | [WCH 下载页](https://www.wch-ic.com/downloads/CH32H417DS0_PDF.html) |
| EA3036C | `EA3036C 3CH Power Management IC` Ver. 1.4：p2 Pin Description，p5 Electrical Characteristics，p11 Enable Control。 | [LCSC 托管的厂商数据表](https://datasheet.lcsc.com/datasheet/pdf/d83e9c52d5fbdb916908193b63b8ae60.pdf?productCode=C570857) |

WCH 文档的 MinerU 归档、来源、版本和哈希记录在
[数据手册归档说明](../datasheets/wch-ch32h417-datasheet/README.md) 与
[参考手册归档说明](../manuals/wch-ch32h417-reference-manual/README.md)。
