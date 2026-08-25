# L4 与 FB1 共用 0402 磁珠选型

## 结论

若保留现有的两颗串联磁珠位置，并希望 `L4` 与 `FB1` 使用同一料号，推荐贴装
`Murata BLM15PX221SN1D`。

它是 `1005 metric / 0402 inch` 的单路电源线磁珠，规格为：

- `220 Ohm +/-25% @ 100 MHz`
- `DCR <= 0.10 Ohm`
- `1.4 A @ 85 C`，`0.8 A @ 125 C`

这个选择保留了当前 L4 的 `220 Ohm @ 100 MHz` 目标，同时把 0402 封装下的直流
电阻和额定电流留在适合电源支路的范围。原厂将 BLM15PX 系列列为 DC power line
用途。

`TPS7A2033` 的连续额定输出上限为 `300 mA`；即使该输出的全部连续电流都流经 L4，
该磁珠在 `125 C` 的 `800 mA` 额定仍有余量。其最大直流压降为
`Delta V = I x R`：`100 mA` 时 `10 mV`，`300 mA` 时 `30 mV`。
这不替代对 `VDDIO` 与 `VREFP` 差分电压的首板测量，也不对 L4 是否应保留作拓扑结论；
本文只给出“保留该位置”前提下的器件选型。

`FB1` 的 OPA2356 电源支路负载显著小于上述 L4 电流边界，因此同一料号用于
`+5V -> AV5` 没有额定电流或直流压降问题。相较当前 FB1 的 `1 kOhm @ 100 MHz`，
它并非同阻抗替换；`220 Ohm @ 100 MHz` 仍是有效的高频隔离量级，但实际抑制效果仍由
两侧去耦、布局和负载频谱决定。

## 比较

| 选择 | 封装 | 阻抗 @ 100 MHz | DCR 最大值 | 额定电流 | 对两个位置的判断 |
| --- | --- | ---: | ---: | ---: | --- |
| **Murata BLM15PX221SN1D** | 1005 / 0402 | 220 Ohm | 0.10 Ohm | 1.4 A @ 85 C; 0.8 A @ 125 C | **首选共用料号**。与现有 L4 的 220 Ohm 相同，且直流压降最保守。 |
| Murata BLM15PX601SN1D | 1005 / 0402 | 600 Ohm | 0.23 Ohm | 0.9 A @ 85 C; 0.5 A @ 125 C | 可作为更高阻抗备选；用于 L4 时，300 mA 的最大 DC 压降为 69 mV，故不如 221 型保守。 |
| TDK MPZ1005A331ET000 | 1005 / 0402 | 330 Ohm | 0.30 Ohm | 0.6 A | 第二供应商备选；可用，但 DCR 高于 Murata 221 型。 |
| Fenghua CBG100505U601T | 1005 / 0402 | 600 Ohm | 1.00 Ohm | 0.1 A | **不可用于 L4，也不应作为两处共用料号。** |

风华的当前原厂表将 `CBG100505U601T` 定义为 `600 Ohm @ 100 MHz`、`DCR <= 1.00 Ohm`、
`100 mA`。此前商品页或旧资料出现的 `1.3 Ohm` 不影响结论：它的 `100 mA` 额定远低于
本设计为 L4 设定的至少 `500 mA` 选型门槛，且 DCR 也过高。它最多只能单独评估为低电流
OPA 支路的候选，不能承担 VDDIO 支路。

## 原厂来源

1. Murata，[BLM15PX221SN1 系列原厂规格书](https://www.murata.com/products/productdata/8796740059166/ENFA0018.pdf?1730777411000=)：`BLM15PX221SN1D` 的 0402/1005 尺寸、`220 Ohm`、`0.10 Ohm`、`1.4 A @ 85 C`、`0.8 A @ 125 C` 及 DC power line 用途。
2. Murata，[BLM15PX601SN1 原厂产品页](https://www.murata.com/en-us/products/productdetail?partno=BLM15PX601SN1%23)：`600 Ohm @ 100 MHz`、`0.23 Ohm`、`0.9 A @ 85 C`、`0.5 A @ 125 C`、0402/1005。
3. TDK，[MPZ1005A331ET000 原厂产品页](https://product.tdk.com/en/search/emc/emc/beads/info?part_no=MPZ1005A331ET000)：0402/1005、Power line、`330 Ohm @ 100 MHz`、`0.30 Ohm`、`0.6 A`。
4. Fenghua，[CBG 原厂规格书](https://www.fenghua.com/pdf/inductor/CBG.pdf)：1005/0402 表中 `CBG100505U601T` 为 `600 Ohm @ 100 MHz`、`1.00 Ohm`、`100 mA`。
5. TI，[TPS7A20 原厂数据手册](https://www.ti.com/lit/ds/symlink/tps7a20.pdf)：TPS7A2033 的 `0-300 mA` 输出能力。
