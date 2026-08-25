# CH32H417 VDDK 与 VDD12A 外供电指导

## 结论

当前将 `1V2` 经两条独立的 0-ohm 支路分别送往 `VDDK` 和 `VDD12A`
的拓扑，**不与 WCH 原厂典型外供电路冲突**。更直接地说，WCH 数据手册
图 3-1-2 的 CH32H417 外部 DC-DC 单一 5-V 典型电路，将同一个外部 1.2-V
输出直接同时接到 `VDD12A` 与 `VDDK`；两路各自仅保留引脚旁去耦。

在已查资料中，**未找到**要求两路各自使用独立稳压器、各自串 0-ohm，或
各自串磁珠的表述。因此，不能把这三项说成 WCH 的强制要求。

两条 0-ohm 支路可保留为调试、改版或电流测量边界，但贴装后并不构成有效的
电源域隔离。是否改成磁珠应由具体的噪声/EMI 测量和布局回路决定，不能仅以
WCH 文档作为必须改动的依据。

## 两个电源域

| 电源 | 原厂定义 | 工作范围 | 原厂去耦要求 |
| --- | --- | --- | --- |
| `VDDK` | 数字内核电路供电 | 1.17--1.27 V，典型 1.2 V | 每个引脚 0.1 uF；主 `VDDK` 再并 2.2 uF |
| `VDD12A` | 内置 USB 3.0 模块供电 | 1.17--1.27 V，典型 1.2 V | 图 3-1-2 给出 0.1 uF 并联 2.2 uF |

`VDD12A` 的 A 不是 ADC 模拟供电后缀；它是 USB 3.0 模块专用的 1.2-V
电源。`VDDK` 则是数字内核的 1.2-V 电源。二者是不同的片内负载域，但 WCH
的外供电典型电路将其共用为一条板级 1.2-V 电源，并以各自的本地去耦处理
高频电流回路。

数据手册表 3-2 注 1 进一步说明，两路电流较大，考虑 PCB 走线压降时，外供
建议为 `1.2 V + 10--50 mV`，即 1.210--1.250 V。

## 当前网表的核对

当前网表把 `1V2` 分成两条独立的 0-ohm 支路，分别接至 `VDDK` 与
`VDD12A`；每路都有 100 nF 和 4.7 uF 对地去耦。其容量不低于 WCH 图
3-1-2 中每路 100 nF 和 2.2 uF 的组合，但网表不能证明电容与相应供电引脚的
PCB 回路足够短，布局仍须按原厂典型电路的本地去耦意图实现。

项目定义的 `1V2` 目标为 1.218 V。计算为：

```
1.218 V - 1.200 V = 18 mV
```

所以 1.218 V 位于 WCH 建议的 `1.210--1.250 V` 内，也位于绝对工作范围
`1.17--1.27 V` 内。这个判断的前提是满载时两个 MCU 供电引脚处的实测最小值
仍在 1.17--1.27 V 内；18 mV 只是标称源端相对 1.2 V 的余量，不能替代 PCB
压降和瞬态跌落验证。

## 关于共用、0-ohm 与磁珠

WCH 数据手册要求“同名电源引脚必须短接”，这里指同一名称的多个引脚，不等同
于要求把 `VDDK` 与 `VDD12A` 直接短接。不过，图 3-1-2 已明确示范了将二者
共同接到一个外部 1.2-V 输出的可接受实现。

但在已查原厂资料中：

- 未找到要求 `VDDK` 与 `VDD12A` 必须使用两个独立的 1.2 V 稳压器；
- 未找到要求任一路必须串 0-ohm；
- 未找到要求任一路必须串磁珠；
- 未找到禁止二者共用同一外部 1.2 V 稳压器的表述。

因此，当前的共同上游 `1V2` 加两条 0-ohm 分支是可接受的实现。若目标是让
两路在高频上相互隔离，0-ohm 本身不提供这种隔离；但 WCH 的典型电路也没有
在两路间串磁珠。不能在缺少布局、转换器阻抗和实测频谱的条件下，宣称某个
磁珠必然改善该芯片的供电质量。作为样机边界，保留两条 0-ohm 分支便于测量或
改版；作为滤波元件则没有设计作用。

## 外供电的软件动作

WCH 官方 SDK 的 `VDD12_ExternPower` 例程明确规定：在外部 1.2 V 已稳定后，
执行 `PWR_VDD12ExternPower()`，以降低片内 VDD12 电压，降低功耗和发热。例程
在 V3F 的 `SystemInit()` 后、唤醒 V5F 前调用它；函数实现同时把三个相关的
电源配置字段设置为 `001b`。

当前设计的 `VDDK` 和 `VDD12A` 都是外供 1.2 V，故固件应按该官方例程在
早期启动路径执行此函数，且只有在外部 1.2 V 已稳定后执行。原厂例程把条件描述
为“芯片 VDD12 外部供电”，并未给出可仅对 `VDDK` 或仅对 `VDD12A` 单独启用
的公开说明；不要将该函数外推为单一路外供时的独立配置方法。

## 原厂来源

- WCH, [CH32H417 datasheet, CH32H417DS0](https://www.wch-ic.com/downloads/CH32H417DS0_PDF.html), section 1.4.3, figure 3-1-2, and table 3-2 note 1. Local original: `docs/datasheets/wch-ch32h417-datasheet/CH32H417DS0-V1.8.PDF`; text extraction: `docs/datasheets/wch-ch32h417-datasheet/wch-ch32h417-datasheet.md`.
- WCH, [official CH32H417 SDK: VDD12_ExternPower](https://github.com/openwch/ch32h417/tree/main/EVT/EXAM/PWR/VDD12_ExternPower), especially [V3F main.c](https://github.com/openwch/ch32h417/blob/main/EVT/EXAM/PWR/VDD12_ExternPower/V3F/User/main.c) and [PWR implementation](https://github.com/openwch/ch32h417/blob/main/EVT/EXAM/SRC/Peripheral/src/ch32h417_pwr.c).
- Project topology evidence: `/Users/ivan/Downloads/Netlist_Schematic1_2026-08-25.enet`; project 1.218-V target: `docs/SELECTION.md`.
