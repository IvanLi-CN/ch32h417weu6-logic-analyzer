# GZ1005D121TF 在 L4 与 FB1 的适用性

## 结论

`GZ1005D121TF` 可以作为 **FB1 单独的低电流替代**，但**不能作为 L4 与 FB1 的共用料号**，也不能用来证明本项目 HSADC 20 MS/s 的供电条件成立。

阻断点不在其 `120 Ohm @ 100 MHz`，而在 L4 的供电额定和直流压降：该器件仅
`200 mA`、`DCR <= 0.40 Ohm`。U8 `TPS7A2033PDBVR` 的输出额定上限是
`300 mA`。在该保守边界，GZ 的电流额定被超过，且直流压降为 `120 mV`。

更根本地，L4 仍串在 `AVCC/VREFP -> VDDIO`。HSADC 的约束是
`VDDIO >= VREFP`；任一正向负载电流与非零串联阻抗都会令其不能仅由原理图获得硬保证。
更换为 GZ 不改变这一事实，反而因 DCR 更高而增加 DC 差分电压。

## 原厂规格

Sunlord 的 GZ 系列规格书把 `GZ1005` 明确标为 `[0402]`，尺寸为
`1.0 +/-0.15 mm x 0.5 +/-0.15 mm`，厚度 `0.5 +/-0.15 mm`。[S1]

同一份原厂表中，`GZ1005D121TF` 的规格为：

| 参数 | 原厂值 |
| --- | --- |
| 标称阻抗 | `120 Ohm +/-25% @ 100 MHz` |
| 最大 DCR | `0.40 Ohm` |
| 最大额定电流 | `200 mA` |
| 电流温度条件 | 原厂表**未标注**测试或环境温度条件；不能将其解释为 `125 C` 额定。 |

Sunlord 的产品选型指南把 GZ 系列归为「low-speed signal line」，而将 PZ/UPZ/EPZ
系列另列为「power line」。这不单独构成失效条件，但说明该 GZ 型号并非优先面向电源支路的系列。[S2]

## L4：`AVCC/VREFP -> VDDIO`

当前网表中，L4 的两端是 `AVCC` 和 `VDDIO`；U8 的输出是 `AVCC`，标称 `3.3 V`、
最大 `300 mA`。[N1] 因此以 `300 mA` 对 L4 作保守边界是合理的元件选型检查，
即使实际 L4 电流需要首板测量确定。

### 直流计算

`Delta V = I x DCR`：

| L4 方案 | 最大 DCR | 额定电流 | `I = 300 mA` 的 DC 压降 | 结论 |
| --- | ---: | ---: | ---: | --- |
| 现有 TDK `MPZ1608S221ATA00` | `0.050 Ohm` | `2.2 A` | `15 mV` | 电流额定有充足余量。 |
| Murata `BLM15AG121SN1D` | `0.19 Ohm` | `550 mA @ 125 C` | `57 mV` | 电流额定覆盖 300 mA。 |
| **Sunlord `GZ1005D121TF`** | **`0.40 Ohm`** | **`200 mA`** | **`120 mV`** | **300 mA 已超额定，不能作为允许工作点。** |

若仅以 GZ 的额定上限 `200 mA` 计算，压降也为 `80 mV`。TPS7A20 的
3.3 V 版本输出精度为 `+/-1.5%`，故最低输出为
`3.3 x (1 - 0.015) = 3.2505 V`。[T1] 忽略走线和瞬态时：

- `200 mA`：`VDDIO = 3.2505 - 0.080 = 3.1705 V`；
- 非允许的 `300 mA`：`VDDIO = 3.2505 - 0.120 = 3.1305 V`。

后者已经低于项目中采用的 `3.15 V` VDDIO 下限。前者虽未低于该绝对电压，
却依然会使 `VDDIO` 比 L4 上游的 `VREFP` 低约 `80 mV`，不能满足或证明 HSADC 的
`VDDIO >= VREFP` 关系。WCH 要求 VDDIO 旁有 `0.1 uF + 至少 10 uF` 的本地去耦，
并要求 HSADC 工作时直接验证 `VREFP - VDDIO` 不为正。[W1]

因此，**GZ1005D121TF 不可用于 L4，也不可作为两处共用料号。**20 MS/s 并不把
`120 Ohm @ 100 MHz` 转换成充分条件；供电是否合格还取决于电流瞬态、去耦、PCB 回流和
MCU 引脚处的 `VREFP - VDDIO` 实测。

## FB1：`+5V -> AV5`

当前 FB1 只给 U6 `OPA2356AIDGKR` 的 `AV5` 供电，AV5 侧有 `100 nF`（C42）和
`22 uF`（C43）。U6 的两个输出经各自 `33 Ohm` 电阻驱动 HSADC 输入，且各输入侧有
`47 pF` 对地电容。[N2]

对 FB1，GZ 的 `200 mA` 额定可覆盖该原理图的已声明持续上界：OPA2356 的最大静态电流为
`14 mA/通道`，连续输出电流为 `60 mA/通道`；即使以两个通道都持续源出 60 mA 的极端上界估算，
`2 x (14 + 60) = 148 mA < 200 mA`。[O1] 实际的 20 MHz、`47 pF` 负载电流也远小于
`60 mA`：以 `3.3 Vpp` 正弦为例，每通道电容电流峰值约为
`2 pi x 20 MHz x 47 pF x 1.65 V = 9.7 mA`。这只是输出负载电流的量级检查，
不是对供电纹波的保证。

所以，若只替换 FB1，`GZ1005D121TF` 的**额定电流可接受**。但它将现有 FB1 的
`1 kOhm @ 100 MHz / 0.12 Ohm / 1.5 A` 改为 `120 Ohm / 0.40 Ohm / 0.2 A`，高频隔离
和供电余量都显著下降；不建议因为「两个位置同料号」而把它用于 L4。

## 来源

1. **[S1]** Sunlord，[GZ Series Multilayer Chip Ferrite Bead 官方规格书](https://www.sunlordinc.com/uploads/files/20221115/GZ%20series%20of%20Multilayer%20Chip%20Ferrite%20Bead.pdf)，封装图（`GZ1005 [0402]`）和 GZ1005 规格表（`GZ1005D121TF`：120 Ohm、0.40 Ohm、200 mA）。原厂表未给这 200 mA 的温度条件。
2. **[S2]** Sunlord，[Product Selection Guide 2023A](https://www.sunlordinc.com/uploads/files/20230823/Product%20Selection%20Guide%202023A.pdf)，GZ 为 low-speed signal line，PZ/UPZ/EPZ 为 power line。
3. **[N1]** [Netlist_Schematic1_2026-08-25.enet](</Users/ivan/Downloads/Netlist_Schematic1_2026-08-25.enet>)：U8 `TPS7A2033PDBVR` 的 3.3 V/300 mA 与 `AVCC` 输出（1314-1377 行）；L4 的 `AVCC -> VDDIO` 和现有 TDK 参数（1510-1555 行）。
4. **[N2]** 同网表：FB1 的 `+5V -> AV5` 和现有参数（8443-8487 行）；U6、R17/R18、C35/C36、C42/C43 的网络（1927-2037、2204-2324、8490-8574 行）。
5. **[T1]** TI，[TPS7A20 数据手册](https://www.ti.com/lit/ds/symlink/tps7a20.pdf)，3.3 V 版本输出精度 `+/-1.5% max.` 与 `0-300 mA` 额定输出。
6. **[O1]** TI，[OPA356/OPA2356 数据手册](https://www.ti.com/lit/ds/symlink/opa356.pdf)，电气特性表：`14 mA/通道` 的指定温度范围静态电流上限、`+/-60 mA` 连续输出电流。
7. **[W1]** WCH，[CH32H417DS0 官方数据手册](https://www.wch-ic.com/downloads/CH32H417DS0_PDF.html)，电源关系及表 3-52 HSADC 条件；项目内的 [参考供电拓扑核验](ch32h417-qeu6-reference-power-topology.md) 已摘录对应要求及引脚处验证方法。
8. **[M1]** Murata，[BLM15AG121SN1 官方规格书](https://www.murata.com/products/productdata/8796740059166/ENFA0018.pdf?1730777411000=)：`120 Ohm +/-25% @100 MHz`、`0.19 Ohm`、`550 mA @125 C`。
