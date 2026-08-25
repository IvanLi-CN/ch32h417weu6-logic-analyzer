# CH9441P Function-Level Alternatives

## Current Project Decision

The project uses `HD3SS3220` in I2C-managed UFP/Sink configuration. It supplies
the required automatic CC orientation handling and reversible SuperSpeed mux.
Its I2C link uses CH32H417 `I2C2` on `PB10/PB11`, directly pulled up to
`VIO18`. The concrete connection requirements are recorded in
[HD3SS3220 UFP port selection](hd3ss3220-ufp-selection.md).

This note remains a function-level comparison, not the active BOM decision.

## Conclusion

`CH9441P` is a Type-C reversible SuperSpeed signal mux, not a USB hub. It
does not create downstream USB ports. A replacement must route one USB
SuperSpeed link through one Type-C connector and swap the two differential
channel pairs when the connector is flipped.

The closest WCH continuity candidate is `CH9441C`: WCH covers both `CH9441P`
and `CH9441C` in the same data-sheet scope, with the same automatic CC
orientation recognition and USB 3.2 Gen1 target. `TI HD3SS3220` is the
selected one-chip redesign: it adds a Type-C DRP controller and supports up
to 10 Gbps. Diodes `PI5USB30213A` and
`PI5USB31213A` are direct non-WCH alternatives with integrated CC logic and
an automatic SuperSpeed mux. `CH9440C`/`CH9440D` are mux-only alternatives:
they require the product to determine orientation and drive their `SEL` input
itself.

This note intentionally makes no package, footprint, or pin-compatibility
claim. None of the candidates should be treated as a drop-in replacement.

## WCH Baseline

WCH describes CH9441 as a Type-C orientation-adaptive USB 3.0 analog switch.
It contains four bidirectional 2:1 QPDT switches that form two differential
channels, recognizes the active CC pin, and automatically switches the
SuperSpeed paths for either Type-C orientation. The published target is USB
3.2 Gen1 SuperSpeed; WCH specifies 6 Gbps differential signaling support.
[WCH CH9441 product page](https://www.wch.cn/products/CH9441.html)

| Hard constraint | CH9441P function-level requirement |
| --- | --- |
| USB data function | One USB 3.2 Gen1 SuperSpeed link. This is two differential channels for the USB TX and RX pairs, not a USB hub or a multi-port controller. |
| Type-C role | WCH states that it supports Type-C source-side and sink-side applications and automatically handles connector orientation from CC. It is not documented as a USB PD protocol controller. |
| Downstream ports | None. USB 2.0 D+/D-, VBUS power switching, and USB PD policy are outside this SuperSpeed mux function. |
| Power | `VBUS` is the high-voltage LDO input (3-29.5 V); an external `VDD` input is 2.7-5.5 V when used. The current analyzer port is a 5 V USB sink, so PD high-voltage operation is not required. |
| Control levels | The WCH family documentation states 1.8 V, 2.5 V, 3.3 V, and 5 V logic control compatibility independent of the supply voltage. |
| Clock and firmware | The WCH product documentation lists neither an external reference-clock input nor a host configuration bus, and describes orientation switching as automatic hardware behavior. Do not infer USB PD negotiation from that behavior. |

The CH9441 data-sheet landing page explicitly includes `CH9441P` and
`CH9441C` in its scope. [WCH CH9441DS1 data sheet](https://www.wch.cn/downloads/CH9441DS1_PDF.html)

## Candidate Assessment

| Candidate | Classification | Function-level fit | Hard constraints and required redesign work |
| --- | --- | --- | --- |
| `CH9441C` (WCH) | Equal functional candidate | The same WCH data-sheet scope covers it and `CH9441P`. It preserves automatic CC orientation behavior, the USB 3.2 Gen1 target, and the high-voltage-LDO function. | This is the closest WCH continuity candidate when the port must remain autonomous. It is not footprint or pin compatible with `CH9441P`; use its own symbol, footprint, and routing. |
| `PI5USB30213A` (Diodes) | Equal functional candidate | Integrated Type-C dual-role controller, CC detection, and 2:1 USB 3.2 Gen1 5 Gbps mux. It explicitly supports Device/UFP mode and automatic orientation; in pin-control mode, the `PORT` pin can select the role without I2C firmware. | The appropriate non-WCH 5 Gbps alternative. It operates from 2.7-5.5 V and needs its own Type-C peripheral design, including VBUS detection and configured UFP/Sink mode. Confirm distributor availability before BOM lock. [Diodes product page](https://www.diodes.com/part/view/PI5USB30213A) |
| `PI5USB31213A` (Diodes) | Equal functional candidate with higher link margin | Same integrated CC/orientation architecture as `PI5USB30213A`, with a 10 Gbps USB 3.2 Gen2 mux. It supports Device/UFP mode and pin or I2C control. | It adds no usable throughput with CH32H417 USB 3.2 Gen1, but is valid if its supply chain is better. It also operates from 2.7-5.5 V and needs a new Type-C peripheral design. [Diodes product page](https://www.diodes.com/part/view/PI5USB31213A) |
| `HD3SS3220` (TI) | Selected, equal or better core function with a new power architecture | Integrated CC controller and 2:1 SuperSpeed mux; automatic attach, orientation, and role detection; DFP, UFP, and DRP modes; USB 3.1 Gen1/Gen2 up to 10 Gbps. It exceeds CH9441P in link rate and Type-C role handling. | It is not powered directly from a 20 V or 28 V VBUS rail: it needs `VDD5` at 4.5-5.5 V and `VCC33` at 3.0-3.6 V. It can sense 4-28 V VBUS through a 900 kohm resistor, but this is not a high-voltage supply input. The selected project uses direct `PB10/PB11` I2C pulled to `VIO18`; it still does not support USB PD communications, so a PD controller is required when PD negotiation is needed. [TI product page](https://www.ti.com/product/HD3SS3220), [TI data sheet](https://www.ti.com/lit/ds/symlink/hd3ss3220.pdf) |
| `CH9440C` / `CH9440D` (WCH) | Mux-only alternative | WCH describes `CH9440` as a 2:1 QPDT differential switch for USB 3.0 SuperSpeed. The mux itself has a 3.3 V or 5 V `VDD` input, `EN#`, and `SEL`; it does not contain the CH9441 CC detector or high-voltage LDO. | A viable lower-cost redesign only when the device itself detects CC orientation and drives `SEL`. It is not an autonomous replacement. [WCH CH9441/CH9440 product page](https://www.wch.cn/products/CH9441.html) |

## Selection Rules

- Current project selection: use `HD3SS3220` in I2C-managed UFP/Sink mode.
- Need a WCH continuity candidate: evaluate `CH9441C`.
- Need an autonomous, non-WCH 5 Gbps part: evaluate `PI5USB30213A`.
- Need an autonomous, non-WCH 10 Gbps-capable part: evaluate
  `PI5USB31213A`.
- Redesign the port around regulated 5 V and 3.3 V rails and want stronger
  built-in Type-C role handling or 10 Gbps capability: use `HD3SS3220`.
- Use `CH9440C` or `CH9440D` only when the board and firmware explicitly
  implement CC orientation detection and drive `SEL`.

`ANX7408` is another technically valid 5 Gbps integrated CC/mux part, but it
includes a USB-PD controller and requires a separate digital-I/O supply and
configuration integration. It is retained only as a supply-chain fallback;
the details are in [the additional candidate note](ch9441p-additional-candidates.md).

## CH32H417WEU6 Constraint

The selected `CH32H417WEU6` does not contain the `USBPD Type-C` peripheral:
the official CH32H417 resource table marks this resource as absent for the
WEU6 package, while QEU6 and MEU6 include it. Its generic GPIO or ADC could
be used to build a basic UFP/Sink-only CC detector, but that is new firmware
and a new Type-C state-machine design, not reuse of a WCH USBPD peripheral.
For that alternative design, retain one 5.1 kohm `Rd` resistor from each
connector CC pin to ground, sample the active CC line with high-impedance MCU
inputs, and set the mux orientation before enabling USB SuperSpeed. It must be
validated against the USB Type-C attach/detach and orientation cases. This
fallback does not apply to the selected `HD3SS3220`, whose CC `Rd` is internal.

The reference application firmware includes the USBPD header but does not
initialize or access the USBPD peripheral. This does not change the WEU6
hardware limitation. [CH32H417 data sheet](../datasheets/wch-ch32h417-datasheet/CH32H417DS0.PDF)

USB 2.0-only Type-C switches and USB hub controllers are excluded: they do
not provide the required reversible SuperSpeed differential-channel function.
