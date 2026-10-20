# Firmware Baseline

This directory contains the pinned source baseline for the CH32H417WEU6 logic analyzer. It is an imported and auditable starting point, not a board-qualified firmware image.

## Provenance

- Upstream: [Calvariaa/CH32H417_Logic_Analyzer_Mini](https://github.com/Calvariaa/CH32H417_Logic_Analyzer_Mini)
- Pinned commit: `70a2e6844b0649b4ac07f692926f342da991395c`
- Upstream license: GPL-3.0; the included WCH SDK files retain their original WCH headers and usage notices.
- Imported tree: [ch32h417-logic-analyzer-baseline](ch32h417-logic-analyzer-baseline/)

The import keeps the MounRiver Studio project descriptions, application and IAP sources, WCH peripheral library, linker scripts, and the upstream `ch32h417_uhsif_it.o` object required by the APP V5F project. Generated `obj/` directories, prebuilt `.bin/.hex/.elf` files, vendor PDFs, EDA projects, and the Windows host executable are intentionally excluded. The local baseline normalizes the MounRiver device metadata to `CH32H417WEU` and adds the project-required external-`VDD12` startup guard; the original upstream README is retained as `README.upstream.md`.

The retained precompiled object has SHA-256 `5f58ac252b1c1e57cd2736e53721d4855097fbba780f82892d8342c516ee049e`. It has no corresponding source in the pinned upstream tree and remains a build and redistribution review item.

## Project Layout

- `CH32H417_Logic_Analyzer_1v0/CH32H417_Logic_Analyzer_APP/V3F`: APP V3F project and dual-core merge input.
- `CH32H417_Logic_Analyzer_1v0/CH32H417_Logic_Analyzer_APP/V5F`: APP V5F project, USBSS/UHSIF/HSADC logic, and merged APP output configuration.
- `CH32H417_Logic_Analyzer_1v0/CH32H417_Logic_Analyzer_IAP/V3F`: IAP project intended for initial WCH-Link programming.
- `CH32H417_Logic_Analyzer_1v0/SRC`: WCH startup, core, peripheral headers, and peripheral sources.

## Build Entry Point

Open the `.wvproj` files with MounRiver Studio 2.x and let MRS regenerate the excluded `obj/` build directories. Build the IAP V3F project separately. For the APP, build the V3F project first and then the V5F project so that MRS can produce the merged dual-core APP artifact.

The checked-in upstream generated makefiles contained machine-specific Windows paths, so they are not part of this baseline. A reproducible command-line build is a follow-up task after the toolchain version and the WEU6 project metadata are normalized.

## Compatibility Status

The imported source is not yet approved for the current board. The following gates are mandatory before generating or flashing a board image:

1. Audit all linker regions, startup files, and package assumptions for `CH32H417WEU6` after the metadata normalization.
2. Replace the upstream VIO18 policy before hardware bring-up. The reference code selects the internal VIO18 software mode and initializes `PA5/DAC2`; this project uses an independent external VIO18 rail and has no approved internal-LDO handover sequence.
3. Verify the local external-VDD12 startup policy: the V3F startup now waits 20 ms and calls `PWR_VDD12ExternPower()` before waking V5F or initializing USB3. Keep this first-board guard until scope evidence justifies changing it.
4. Reconcile the UHSIF, USB2, USBSS, HSADC, I2C2, UART, LED, and debug pins against the current [hardware selection](../docs/SELECTION.md). Do not inherit the reference board pin map from its README alone.
5. Keep SWD available during bring-up. The upstream APP disables SWJ at runtime, which would make recovery harder on a board whose USB protocol and power policy are still under audit.
6. Decide whether the project will retain the vendor-specific `U3LogicAnalyzer` protocol or define a new host contract. Upstream PulseView/libsigrok discovery cannot be assumed.
7. Resolve the source gap in `ch32h417_uhsif_it.o`, or document and accept the WCH binary dependency and its redistribution terms.

Until these gates pass, the WCH-Link may be used only for non-destructive connection and identification checks. Do not flash the imported upstream images or generate a release image from this directory.

## Local References

- [Firmware source research](../docs/research/ch32h417-firmware-sources.md)
- [Project requirements](../docs/research/project-requirements.md)
- [Power sequencing](../docs/research/ch32h417-power-sequencing.md)
- [External VIO18 analysis](../docs/research/ch32h417-external-vio18-dac-control-official-analysis.md)
