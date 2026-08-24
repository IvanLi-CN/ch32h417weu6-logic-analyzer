# USB-C 烧录转接板演进历史

> 当前有效规范以 `./SPEC.md` 为准；本文件只记录影响长期理解的决策理由。

## Decision Trace

- CH32H417WEU6 的 `PB8`/`PB9` 同时承载 USB2 `D+`/`D-` 和 WCH-Link
  两线 SDI 下载的 `SWCLK`/`SWDIO`，因此不能让普通 USB Host 与 WCH-Link
  同时直接接入。
- 决定使用外置 USB-C 烧录转接板，而不是在主板插入 0 Ohm 跳线、额外
  下载排针或 USB2 模拟开关。
- 烧录转接板以独立的 USB-C 母座接收 5 V，以公头向目标板重新作为 Source，
  并独占目标侧 D+/D- 给 WCH-Link。

## Key Reasons / Replacements

- 该架构使“正常 USB 数据连接”和“烧录连接”在物理上互斥，消除了两者
  同时驱动 PB8/PB9 的冲突。
- 主板保持单一 USB-C 口和既有 USB3 布局，不引入烧录时的信号切换元件。
- 供电由转接板的受控 VBUS 处理，避免使用 WCH-Link 的辅助电源输出承担
  主板电源预算。

## References

- `./SPEC.md`
- `./IMPLEMENTATION.md`
