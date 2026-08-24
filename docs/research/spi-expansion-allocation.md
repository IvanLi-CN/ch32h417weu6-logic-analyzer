# Hardware SPI Expansion Reservation

## Decision

Reserve `SPI1` as the board's default future full-duplex hardware-SPI
expansion interface. This allocation does not replace or share any active
capture, analog, USB, UART, or Type-C management signal.

The CH32H417 data sheet assigns all four signals to the following WEU6 pins at
`AF5`. The four SPI pads are consecutive around one QFN68 package edge; the
adjacent spare GPIO is included for a future interrupt, reset, or second chip
select.

| Expansion net | WEU6 QFN68 pad / GPIO | MCU function | Use now |
| --- | --- | --- | --- |
| `EXP_SPI_CS0` | `61 / PF4` | `SPI1_NSS`, AF5 | Reserved as the first device chip-select. Firmware can use the SPI NSS function or ordinary GPIO mode as required by the future device. |
| `EXP_SPI_SCK` | `62 / PF5` | `SPI1_SCK`, AF5 | Reserved. |
| `EXP_SPI_MOSI` | `59 / PD7` | `SPI1_MOSI`, AF5 | Reserved. |
| `EXP_SPI_MISO` | `60 / PF3` | `SPI1_MISO`, AF5 | Reserved. |
| `EXP_GPIO0` | `58 / PD6` | GPIO | Reserved for an optional interrupt, reset, or additional software chip-select. |

The physical pad order is `PD6`, `PD7`, `PF3`, `PF4`, `PF5` (pads `58` through
`62`). This minimizes routing spread at the MCU. The CH32H417 SPI timing table
specifies a 75 MHz maximum SPI clock for both master and slave operation. That
is the MCU interface limit, not a promise that a future off-board module will
work at 75 MHz; its trace length, connector, loading, and target device set the
validated clock rate.

## Conflict Check

This is an explicit new allocation, not a conclusion drawn solely from empty
reference-board pads. WCH's alternate-function table supplies the SPI1 mapping;
the reference evidence is used only to exclude conflicts with the current
product functions.

- `UHSIF`: `PD10` through `PD15`, `PF0` through `PF2`, `PC6` through `PC9`,
  `PA13` through `PA15`, and `PC0` capture clock. No SPI1 pin overlaps.
- HSADC: `PC2` and `PC3`. No SPI1 pin overlaps.
- Type-C management I2C: `PB10` and `PB11`. No SPI1 pin overlaps.
- USB2, USB3, debug UART TX, DAC-controlled `VIO18`, and LEDs likewise do not
  use the five reserved pins.

`SPI2` remains a separate fallback with `PB12` (`NSS`), `PB13` (`SCK`), `PC1`
(`MOSI`), and `PB14` (`MISO`), all at AF5. It is deliberately not the primary
choice: `PC1` is also `HSADC_IN1`, so retaining it preserves future analog
input flexibility.

## Electrical Contract

The selected SPI1 pins and `PD6` are in the WEU6 `VIO18` I/O domain. Their
logic high level therefore follows the product's capture-domain setting
(`1.84 V`, `2.5 V`, or `3.16 V`) rather than fixed 3.3 V.

- A directly attached SPI device must accept the active `VIO18` level on every
  input and must not drive `MISO` above that level.
- A peripheral that requires a fixed 3.3 V I/O supply cannot be used directly
  while the analyzer is operating at 1.84 V or 2.5 V. It would require a
  separate level-translation decision, which is outside this reservation.
- Do not change `VIO18` while an attached peripheral is powered and driving
  the SPI bus. Put the bus in a defined idle state first.

Reserve schematic and PCB access for `GND`, `VIO18`, the four SPI nets, and
`EXP_GPIO0` as an unpopulated footprint or test pads. Do not fit a connector,
series resistor, pull-up, level shifter, or any other component solely for this
future option. Before using `VIO18` to power an expansion board, include its
load in the VIO18 regulator budget; it is a variable I/O rail, not a generic
fixed-voltage accessory supply.

## Firmware Boundary

No firmware change is required while the expansion pads are unused. Keep the
reserved pins in a benign input or high-impedance state at boot. A future
feature enables the SPI1 peripheral, selects AF5 on the four SPI pins, retains
`PD6` as GPIO, and sets mode, clock rate, and chip-select behavior for the
specific target device.

## Sources

- [WCH CH32H417 data sheet](https://www.wch-ic.com/downloads/CH32H417DS0_PDF.html): WEU6 pinout (VIO18 domain), Table 2-2-8 (SPI pin functions), and Table 3-26 (SPI timing).
- [Current pin-occupancy evidence](hd3ss3220-i2c-gpio-allocation-evidence.md): current product functions excluded from the reservation.
