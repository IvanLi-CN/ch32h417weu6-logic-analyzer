# CH32H417 Official WCH Source Register

## Scope and Handling

This register uses only WCH-published primary material. It does not use a
reference design, open-source logic-analyzer project, LCEDA project, board
measurement, or a third-party support statement.

The repository has no configured Git remote and no tracked statement of
repository visibility. Its public/private status could therefore not be
determined from local facts. Newly acquired WCH PDFs are retained locally,
read-only, and ignored by Git under the vendor-document policy. They were not
staged or committed.

## Official Sources

| ID | Official source | Revision | Evidence used |
| --- | --- | --- | --- |
| WCH-DS | [CH32H417DS0 data-sheet download](https://www.wch-ic.com/downloads/CH32H417DS0_PDF.html) | `CH32H417DS0-EN` v1.8 | Section 1.4.3 (printed page 8), Table 3-1 (page 88), Table 3-2 (page 89), and the DAC2/PA5 pin assignment. The local immutable source and MinerU mirror are documented in [the data-sheet source note](../datasheets/wch-ch32h417-datasheet/README.md). |
| WCH-RM | [CH32H417 reference-manual download](https://www.wch-ic.com/downloads/CH32H417RM_PDF.html) | Published `CH32H417RM` v1.7 | `PWR_CTLR` at Section 2.4.1 (printed page 7), GPIO Sections 9.2.2 and 9.3.1.1 (pages 136-137), and DAC Chapter 18 (pages 268-279). The local source and exact MinerU extracts are documented in [the reference-manual source note](../manuals/wch-ch32h417-reference-manual/README.md). |
| WCH-EVT | [WCH EVT download](https://www.wch.cn/downloads/CH32H417EVT_ZIP.html), [fixed WCH SDK snapshot](https://github.com/openwch/ch32h417/tree/a0a56fa830b860297a64a72315a731b71c841329) | EVT v1.5; commit `a0a56fa830b860297a64a72315a731b71c841329` | `EVT/EXAM/PWR/SEL_VIO18`, plus PWR and DAC peripheral-library sources. It corroborates register writes and warns that software VIO18 adjustment should use hardware configuration for external-device safety. |

No WCH application note or WCH-attributable support answer was needed for the
findings below.

## Applicable Limits

The explicit WCH external-connection case is narrow: WCH-DS Section 1.4.3
allows `VIO18` to be fixed shorted to `VDDIO` when low-voltage support is not
needed. WCH-RM Section 2.4.1 adds that, for a package with exposed `VIO18`,
the 3.3-V case uses an external `VDDIO`-to-`VIO18` short and turns the I/O LDO
off after power-up with `VSEL_VIO18 = 111b`.

That is not an authorization for an independently regulated `VIO18` rail at
1.8 V, 2.5 V, or 3.0 V, nor a specified transition between that rail and the
internal I/O LDO. The complete fact matrix and the project conclusion are in
[the external-VIO18 DAC-control analysis](ch32h417-external-vio18-dac-control-official-analysis.md).
