# Candidate SPI Expansion (Fallback)

## Decision

`SPI1` is a documented fallback candidate for a future full-duplex
hardware-SPI expansion. The baseline board does not reserve an SPI expansion
interface, connector, footprint, test pads, nets, or routes. It also does not
reserve the candidate GPIOs: they remain available for other baseline-board
uses.

If a future board adopts this candidate, the CH32H417 data sheet assigns all
four SPI signals to the following WEU6 pins at `AF5`. The four SPI pads are
consecutive around one QFN68 package edge; the adjacent spare GPIO can provide
an interrupt, reset, or second chip select.

| Future expansion net | WEU6 QFN68 pad / GPIO | MCU function | Baseline-board status |
| --- | --- | --- | --- |
| `EXP_SPI_CS0` | `61 / PF4` | `SPI1_NSS`, AF5 | Future candidate only; not an expansion net or reserved GPIO. Firmware may use SPI NSS or GPIO mode if a future board adopts it. |
| `EXP_SPI_SCK` | `62 / PF5` | `SPI1_SCK`, AF5 | Future candidate only; not reserved. |
| `EXP_SPI_MOSI` | `59 / PD7` | `SPI1_MOSI`, AF5 | Future candidate only; not reserved. |
| `EXP_SPI_MISO` | `60 / PF3` | `SPI1_MISO`, AF5 | Future candidate only; not reserved. |
| `EXP_GPIO0` | `58 / PD6` | GPIO | Future candidate only; not reserved. |

The physical pad order is `PD6`, `PD7`, `PF3`, `PF4`, `PF5` (pads `58` through
`62`). This minimizes routing spread at the MCU. The CH32H417 SPI timing table
specifies a 75 MHz maximum SPI clock for both master and slave operation. That
is the MCU interface limit, not a promise that a future off-board module will
work at 75 MHz; its trace length, connector, loading, and target device set the
validated clock rate.

## Candidate Conflict Check

This is a candidate allocation, not a current interface reservation. WCH's
alternate-function table supplies the SPI1 mapping; the reference evidence is
used only to exclude conflicts with the present product functions. Because the
baseline board does not reserve these pins, re-run this check before assigning
them to an expansion interface in a future board.

- `UHSIF`: `PD10` through `PD15`, `PF0` through `PF2`, `PC6` through `PC9`,
  `PA13` through `PA15`, and `PC0` capture clock. No SPI1 pin overlaps.
- HSADC: `PC2` and `PC3`. No SPI1 pin overlaps.
- Type-C management I2C: `PB10` and `PB11`. No SPI1 pin overlaps.
- USB2, USB3, debug UART TX, DAC-controlled `VIO18`, and LEDs likewise do not
  use the five candidate pins.

`SPI2` is a separate, unreserved alternative candidate with `PB12` (`NSS`),
`PB13` (`SCK`), `PC1` (`MOSI`), and `PB14` (`MISO`), all at AF5. It is not the
preferred future choice because `PC1` is also `HSADC_IN1`, so retaining it
preserves future analog-input flexibility.

## Future Electrical Contract

If a future board adopts the SPI1 candidate, the selected SPI1 pins and `PD6`
are in the WEU6 `VIO18` I/O domain. Their logic high level therefore follows
the product's capture-domain setting (`1.80 V`, `2.50 V`, or `3.00 V`) rather
than fixed 3.3 V.

- A directly attached SPI device must accept the active `VIO18` level on every
  input and must not drive `MISO` above that level.
- A peripheral that requires a fixed 3.3 V I/O supply cannot be used directly
  while the analyzer is operating at 1.80 V or 2.50 V. It would require a
  separate level-translation decision, which is outside this candidate.
- Do not change `VIO18` while an attached peripheral is powered and driving
  the SPI bus. Put the bus in a defined idle state first.

The planned 8-pin FPC allocation, if implemented on a future board, is `GND`,
fixed `3V3` (the `VCC33` rail, nominal `3.32 V`), `VIO18`, the four SPI nets,
and `EXP_GPIO0`. `3V3` is an auxiliary expansion supply only; it does not
change the `VIO18` logic level of the SPI signals.

The baseline board shall not fit or expose any of these FPC signals. Before a
future board uses `VIO18` or `3V3` to power an expansion board, include its
load in the applicable regulator budget. `VIO18` is a variable I/O rail, not a
generic fixed-voltage accessory supply; a module powered from `3V3` must still
meet the SPI-level requirements above.

## Baseline Firmware Boundary

The baseline firmware includes no SPI-expansion support and does not reserve
these pins for that purpose. A future feature must first revalidate pin
ownership and the electrical contract, then enable SPI1, select AF5 on the four
SPI pins, retain `PD6` as GPIO, and set mode, clock rate, and chip-select
behavior for the specific target device.

## Sources

- [WCH CH32H417 data sheet](https://www.wch-ic.com/downloads/CH32H417DS0_PDF.html): WEU6 pinout (VIO18 domain), Table 2-2-8 (SPI pin functions), and Table 3-26 (SPI timing).
- [Current pin-occupancy evidence](hd3ss3220-i2c-gpio-allocation-evidence.md): current product functions excluded from the candidate allocation.
