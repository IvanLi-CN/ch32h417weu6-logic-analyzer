# USB-C 烧录转接板实现状态

> 当前有效规范以 `./SPEC.md` 为准；这里记录实现覆盖和待办事实。

## Current Status

- Implementation: 未开始
- Lifecycle: active
- Spec disposition: create

## Coverage / rollout summary

- 主板不做任何原理图、PCB 或固件改动。
- 待新建独立小板 EDA 工程，接口为 USB-C 输入母座、USB-C 目标公头和
  WCH-Link 下载器连接器。
- 待确定满足 `SPEC.md` 输入能力检测、Source 输出与反灌保护要求的
  Type-C 电源控制方案和器件。

## Remaining Gaps

- 完成小板原理图、PCB、BOM 与机械尺寸。
- 定义 WCH-Link 连接器的具体系列、针序和装配方式。
- 测量目标板的上电及烧录峰值电流，并据此冻结母座输入和公头输出电流策略。
- 完成 `SPEC.md` 中的原理图 review、PCB review 和样机验收。

## Related Changes

- None

## References

- `./SPEC.md`
- `./HISTORY.md`
