# Additional Single-Chip CH9441P Candidates

## Scope

This note only considers a single IC connected to one USB Type-C receptacle
that provides both of the following functions:

1. CC attach/orientation handling suitable for a fixed USB device-side
   UFP/Sink port.
2. A USB 3.x SuperSpeed reversible signal path for one TX and one RX
   differential pair.

The target is the CH32H417WEU6 logic analyzer's USB 3.2 Gen1 (5 Gbps) device
port. USB hubs, mux-only devices, and devices requiring a separate Type-C CC
controller are outside the candidate set. DisplayPort is not a project
requirement. The project now uses `HD3SS3220` in I2C-managed UFP/Sink mode;
this note is retained as supply-chain research, not the active BOM decision.

`CH9441C/P` and `HD3SS3220` were evaluated separately and are not repeated
as additional candidates here. No package, footprint, or pin-compatibility is
implied by any entry.

## Viable Additional Candidate

### Analogix ANX7408

`ANX7408` is the only additional candidate found in official public material
that directly meets the required single-chip function without bringing a
DisplayPort routing matrix into the signal path. Analogix describes it as a
USB Type-C crosspoint switch with an on-chip microcontroller and USB PD PHY.
It supports USB 3.1 Gen1 (5 Gbps), has integrated CC detection and SuperSpeed
signal switching, and can be configured as UFP, DFP, or DRP. Thus its UFP
configuration is electrically applicable to this analyzer port.

Relevant constraints:

- It uses a regulated 3.3 V analog supply and a separate 1.8 V to 3.3 V
  digital-I/O supply.
- Its internal controller implements signal switching and Type-C/PD functions;
  the public brief exposes an I2C slave interface for an external MCU or PMIC.
- The public brief does not provide a fixed-UFP, no-I2C reference circuit or
  a public lifecycle/ordering status. Confirm the required strap/I2C setup and
  procurement before selecting it.

`ANX7408` is therefore a real functional alternative, but not a better
default than `CH9441C/P`: it has substantially broader Type-C/PD functionality
than the analyzer needs and a more complicated power/control integration.

Sources: [Analogix ANX7408 product page](https://www.analogix.com/en/products/switchesmultiplexers/anx7408), [ANX7408 product brief](https://www.analogix.com/en/system/files/AA-002342-PB-3-ANX7408_Product_Brief.pdf).

## Functionally Capable but Not Suitable Selections

### Analogix ANX7418 and ANX7428

Both devices include CC/PD control, an on-chip microcontroller, USB 3.1 Gen1
SuperSpeed switching, and explicit UFP support. They meet the narrow Type-C
orientation requirement. However, they are DisplayPort Alt-Mode crosspoint
devices: `ANX7418` includes two DisplayPort lanes and `ANX7428` includes four.
Those unused high-speed paths, PD functions, and their associated control
surface add cost and layout complexity without improving a USB-only analyzer.
They are technically capable but are not recommended.

Sources: [ANX7418 product brief](https://www.analogix.com/en/system/files/AA-002343-PB-3-ANX7418_Product_Brief_0.pdf), [ANX7428 product page](https://www.analogix.com/en/products/anx7428).

### Analogix ANX7327

`ANX7327` is a USB-PD v3.0 controller with an integrated 4x6 high-speed
de-mux. Its manufacturer explicitly specifies UFP operation and USB 3.1 Gen1
and Gen2 SuperSpeed links. It can implement the required path, but it is
designed for display/accessory and monitor applications, includes DisplayPort
Alt-Mode routing and USB PD policy capability, and is therefore materially
larger in function than the required 2:1 USB-only mux. It is not recommended
for this board.

Source: [Analogix ANX7327 product page](https://www.analogix.com/en/products/usb-type-c-4x6-demux/anx7327).

## Excluded Devices

| Device or family | Exclusion reason |
| --- | --- |
| `ANX7421` | USB 3.2 Gen1 active 1:2 mux, but its orientation control is through external I2C or `FLIP`/`USB_EN`; the official page does not specify integrated CC detection or an autonomous UFP/Sink controller. |
| `ANX7423` and `ANX7433` | Include CC detection and a 1:2 active mux, but the official product pages specify DFP USB-C applications, not the required UFP/Sink application. |
| `ANX7437` and `ANX7447` | Integrate an on-chip controller and 10 Gbps crosspoint, but only DFP or DRP modes are specified. They are not fixed-UFP candidates. |
| TI `TUSB564` and `TUSB1146` | They are Type-C high-speed crosspoint/redriver devices, not CC controllers. TI's reference topology places a separate PD controller on CC1/CC2, violating the single-chip constraint. |
| WCH `CH9440C/D` | They are SuperSpeed mux-only devices. The board must separately determine CC orientation and drive `SEL`. |

Sources: [ANX7421](https://www.analogix.com/en/products/anx7421), [ANX7423](https://www.analogix.com/en/products/anx7423), [ANX7433](https://www.analogix.com/en/products/re-timersrepeaters/anx7433), [ANX7437 and ANX7447](https://www.analogix.com/en/products/switchesmultiplexers), [TI TUSB564](https://www.ti.com/product/TUSB564), [TI TUSB1146](https://www.ti.com/product/TUSB1146), [WCH CH9441/CH9440](https://www.wch.cn/products/CH9441.html).

## Recommendation

For this USB-only, UFP/Sink, USB 3.2 Gen1 port, `HD3SS3220` remains the active
selection. `ANX7408` is the only additional single-chip candidate worth
retaining as a supply-chain fallback, contingent on obtaining its detailed
configuration documentation and verifying purchase availability. The other
technically capable Analogix devices should not be selected because their
DisplayPort and PD features are unused.
