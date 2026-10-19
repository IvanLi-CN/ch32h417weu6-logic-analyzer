# CH32H417 Logic Analyzer Firmware Sources

## Scope

This note evaluates public firmware sources for the current `CH32H417WEU6` logic analyzer: 16 digital channels, a 200 MS/s target, USB 3.0 SuperSpeed transport, optional HSADC capture, externally supplied selectable `VIO18`, and the existing USB-C/HD3SS3220 hardware boundary.

The sources below are references for a controlled port. None of the prebuilt images in an external repository is approved for the current board without a pin, power, USB descriptor, and host-protocol audit.

## Current Board Constraints

The project selects `CH32H417WEU6`, retains USB 3.0 SuperSpeed for the high-rate path, and requires 16 digital capture channels at a 200 MS/s target ([project requirements](project-requirements.md)).

The project requires an independent external `VIO18` rail shared with the input buffer, while `VDD33A` and `VDDIO` remain fixed at 3.32 V when HSADC is used ([project requirements](project-requirements.md), [power architecture](power-architecture.md)).

The current WCH evidence does not establish a safe firmware handover between the internal I/O LDO and this independent external `VIO18` rail ([external VIO18 analysis](ch32h417-external-vio18-dac-control-official-analysis.md)). Firmware that writes the WCH `VIO18` software-mode registers or drives a DAC feedback node therefore requires a separate electrical review.

## Candidate Comparison

| Candidate | What it contains | Fit for this project | Main blockers |
| --- | --- | --- | --- |
| [Calvariaa/CH32H417_Logic_Analyzer_Mini](https://github.com/Calvariaa/CH32H417_Logic_Analyzer_Mini) | GPL-3.0 WEU6 port, MounRiver projects, V3F/V5F source, USBHS/USBSS, 16-bit UHSIF, HSADC code, and V3F/V5F build artifacts. | Best direct firmware baseline: it explicitly targets `CH32H417WEU6`, 16 channels, 200 MHz, USB 3.0, and maps the WEU6 analog/UART/LED pins. | Its power policy drives `PA5/DAC2` for VIO18 control; the current board's independent external VIO18 strategy is not proven compatible. It also disables SWJ/SWD during startup and depends on a precompiled `ch32h417_uhsif_it.o`. |
| [Doukeyi-X/OpenSourceLogic-CH32H417](https://github.com/Doukeyi-X/OpenSourceLogic-CH32H417) | GPL-3.0 IAP + APP source, `prebuilt/` images, WCH library, and an explicit USB CDC-to-APP upgrade flow for [ALL LOGIC](https://github.com/Doukeyi-X/ALL-LOGIC). | Best packaging and recovery reference; useful if we want an explicit IAP/update workflow and a maintained host-side integration. | The README identifies the Q2H2 original hardware and does not claim this tree is a WEU6 port. Its prebuilt images must not be treated as compatible with the current board until package/pin/power differences are checked. Its UHSIF interrupt object is precompiled without corresponding source. |
| [marxing/CH32H417_LogicAnalyzer](https://github.com/marxing/CH32H417_LogicAnalyzer) | Dual-core firmware and PulseView/libsigrok host, USB 2.0/3.0 paths, HSADC, V50 test/recovery images, Type-C mux handling, TFT/UI code, and validation logs. | Strong implementation and validation reference for USB3 recovery, endpoint behavior, Type-C testing, and host integration. | It is a different board with FSW3820, TFT, controls, and external-memory options. The repository explicitly says the firmware/app has no unified license and the UHSIF object has no source; use as a technical reference, not as the default base. |
| [openwch/ch32h417](https://github.com/openwch/ch32h417) | WCH official peripheral library, startup code, and examples for USB 2.0/3.0, UHSIF, HSADC, PWR, DAC, and the dual-core device. | Required low-level reference for validating clocks, register use, `VDD12` external-power startup, USBSS, UHSIF, and HSADC behavior. | It is an SDK/example collection, not a complete logic analyzer application or host protocol implementation. |

## Best Direct Match: Calvaria WEU6 Port

The repository README explicitly says that it changes the original `CH32H417QEU6` design to `CH32H417WEU6`, keeps 16-channel 200 MHz digital capture and USB 3.0, and remaps HSADC to `PC2/PC3`, the VIO18 DAC to `PA5/DAC2`, debug TX to `PD1`, and LEDs to `PB0/PB1` ([README](https://github.com/Calvariaa/CH32H417_Logic_Analyzer_Mini/blob/70a2e6844b0649b4ac07f692926f342da991395c/README.md)).

The fixed reference commit contains both application cores and the relevant source files: `CH32H417_Logic_Analyzer_1v0/CH32H417_Logic_Analyzer_APP/V3F`, `.../V5F`, `Common/ch32h417_logic.c`, `Common/ch32h417_uhsif.c`, `Common/ch32h417_usbhs_device.c`, `Common/ch32h417_usbss_device.c`, and the IAP tree. It also publishes V3F/V5F `.bin`, `.hex`, and `.elf` build outputs.

The capture path is close to the current product requirement: the firmware configures a 16-bit UHSIF input with a 200 MHz clock division, feeds USBSS endpoint buffers, and uses HSADC DMA double buffering for the optional analog path ([UHSIF source](https://github.com/Calvariaa/CH32H417_Logic_Analyzer_Mini/blob/70a2e6844b0649b4ac07f692926f342da991395c/CH32H417_Logic_Analyzer_1v0/CH32H417_Logic_Analyzer_APP/Common/ch32h417_uhsif.c), [logic source](https://github.com/Calvariaa/CH32H417_Logic_Analyzer_Mini/blob/70a2e6844b0649b4ac07f692926f342da991395c/CH32H417_Logic_Analyzer_1v0/CH32H417_Logic_Analyzer_APP/Common/ch32h417_logic.c)).

The source must not be copied unchanged into this project. Its startup code selects WCH VIO18 software mode and initializes DAC2, while this project deliberately keeps an independent external VIO18 rail whose safe internal-LDO handover is still an evidence gap. Its hardware also routes USB2 `PB8/PB9` as the same physical pair used for WCH-Link `SWCLK/SWDIO`, matching the reason this project specifies an external programming adapter ([USB-C programming adapter specification](../specs/usb-c-programming-adapter/SPEC.md)).

The repository also has a build-identity inconsistency that must be resolved before using it as a WEU6 baseline: the V5F project is labeled `WEU`, while the V3F APP and IAP project templates still contain `QEU` identifiers. The pin map, linker settings, startup configuration, and generated images therefore need to be audited together rather than relying on the README or a prebuilt artifact alone. The Calvaria startup path also does not establish the local external-`VDD12` initialization contract; the official WCH example calls `PWR_VDD12ExternPower()` only after the external rail is stable ([WCH VDD12 example](https://github.com/openwch/ch32h417/blob/a0a56fa830b860297a64a72315a731b71c841329/EVT/EXAM/PWR/VDD12_ExternPower/V3F/User/main.c#L31-L37)).

## IAP and Host Compatibility

[OpenSourceLogic-CH32H417](https://github.com/Doukeyi-X/OpenSourceLogic-CH32H417) documents a two-stage flow: flash a V3F IAP with WCH-Link, enumerate a USB CDC bootloader, then upload a merged dual-core APP image through ALL LOGIC. It documents USB IDs `1A86:5539` for IAP and `1A86:5537`/`1A86:5538` for the USB2/USB3 analyzer application ([firmware README](https://github.com/Doukeyi-X/OpenSourceLogic-CH32H417#%E5%BF%85%E9%A1%BB%E7%83%A7%E6%9C%AC%E4%BB%93%E5%BA%93%E5%9B%BA%E4%BB%B6%E6%88%96%E4%B8%BB%E6%9C%BA%E4%BC%9A%E4%B8%8D%E5%B7%A5%E4%BD%9C)).

[ALL LOGIC](https://github.com/Doukeyi-X/ALL-LOGIC) is host-side only. It is based on DSView/PulseView, adds a CH32H417 community driver, and explicitly points to the separate firmware repository. It is useful for protocol and host testing, but it is not a replacement for target firmware.

The firmware USB interface is vendor-specific rather than a standard sigrok device protocol: the reference descriptors use VID/PID `1A86:5537`, EP1 for commands, EP2 for digital data, and EP3 for HSADC data ([descriptor source](https://github.com/Calvariaa/CH32H417_Logic_Analyzer_Mini/blob/70a2e6844b0649b4ac07f692926f342da991395c/CH32H417_Logic_Analyzer_1v0/CH32H417_Logic_Analyzer_APP/Common/usb_desc.c)). Upstream libsigrok does not list this device or `U3LogicAnalyzer`, so “PulseView compatible” requires retaining the dedicated host client/driver or implementing an equivalent protocol layer; standard PulseView discovery cannot be assumed.

## Recommended Adoption Order

1. Use the fixed Calvaria WEU6 commit as the primary code-reading baseline for UHSIF capture, USBSS/USBHS endpoint handling, dual-core startup, and the WEU6 pin remap.
2. Use the WCH SDK examples to replace or validate the low-level power and peripheral initialization, especially external `VDD12` startup and any VIO18-related register access. Treat the QEU/WEU build-template inconsistency as an explicit audit item.
3. Keep the current board's USB3/USB2 routing, HD3SS3220 I2C mapping, capture GPIO mapping, external VIO18 rail, and 20 ms external-`VDD12` guard as local board policy; do not inherit the reference board's pin or power assumptions.
4. Bring up digital capture first with a project-specific USB descriptor and host contract. Add HSADC only after the selected `PCx` mapping, external VIO18 behavior, and analog front end are electrically validated.
5. Decide separately whether to keep the original U3LogicAnalyzer/PulseView host contract or port the firmware to ALL LOGIC. The host choice changes USB descriptors, VID/PID expectations, update flow, and licensing obligations.

## License and Provenance

The Calvaria and Doukeyi firmware repositories identify the application/IAP work as GPLv3-based, but both include WCH-originated code and a precompiled UHSIF object that must be handled according to its file headers and WCH terms. The Marxing repository is more explicit that its integrated firmware package has no single unified license and that `ch32h417_uhsif_it.o` has no corresponding source ([license notes](https://github.com/marxing/CH32H417_LogicAnalyzer/blob/main/LICENSES.md)).

Before redistributing a modified firmware image or adding it to this repository, preserve the upstream notices, verify the WCH library terms, and decide how the precompiled UHSIF object is allowed to be distributed.

## Conclusion

The practical starting point is the Calvaria WEU6 firmware source, with the Doukeyi IAP/host packaging as a secondary reference and Marxing's V50 repository as a USB3 validation reference. The project now carries a source-only baseline under [firmware/README.md](../../firmware/README.md), with WEU6 metadata normalization and the external-VDD12 startup guard applied. This remains a porting baseline, not a ready-to-flash image for the current board; the independent VIO18 power boundary and the final board pin map remain the two highest-risk compatibility gates.
