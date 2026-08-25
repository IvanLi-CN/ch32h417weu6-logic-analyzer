# Four-Channel HSADC Feasibility

## Conclusion

The original project software cannot be extended to four high-speed ADC
channels by changing only the channel count. Its current analog path supports
two selectable inputs, but only one HSADC input is active during a capture.

For the WEU6 redesign, four analog input pins can be exposed as four selectable
inputs because the device package provides one HSADC peripheral with four
external HSADC channels. This is not four independent ADC converters and does
not provide four simultaneous 20 Msps streams.

For every selected HSADC input, the supply condition is unchanged:
`VDD33A = VREFP(internal) = VDDIO = 3.32 V`; the capture-I/O rail may remain at
the selected 1.80-V, 2.50-V, or 3.00-V `VIO18` mode. See
[CH32H417WEU6 HSADC and VIO18 Operating Constraints](ch32h417weu6-hsadc-vio18-constraints.md).

## What the Original Project Actually Does

The original Q2H2 project states that the HSADC has two channels, but that DMA
cannot dynamically switch between them, so only one channel is sampled at a
time. The WEU6 port keeps this behavior and maps the two inputs to PC2 and PC3.

The public firmware confirms the implementation:

- `ADC_CHANNEL0` and `ADC_CHANNEL1` are the only two logical ADC selections.
- `HSADC_Function_Init()` configures either PC2/HSADC channel 2 or
  PC3/HSADC channel 3, based on one `adc_channel` value.
- The firmware has two buffers, `ADC_BufA` and `ADC_BufB`; these are DMA
  double buffers, not two analog acquisition channels.
- The USB path uploads those buffers as one sample stream. There is no channel
  count, channel mask, per-channel buffer, or interleaving metadata in the
  current ADC parameter structure.

Therefore the current software is not a two-channel simultaneous ADC
implementation. It is a single-channel high-speed sampler with two selectable
physical inputs.

## WEU6 Hardware Limit

The CH32H417 family provides one 10-bit HSADC peripheral with a maximum
conversion rate of 20 Msps. The package selection data lists the variants as
follows:

| Device | HSADC units | External HSADC channels |
| --- | ---: | ---: |
| CH32H417WEU6 | 1 | 4 |
| CH32H417QEU6 | 1 | 7 |

QEU6 therefore exposes three more selectable HSADC input pins than WEU6; it
does not contain a second HSADC converter. Both package variants retain one
HSADC conversion path.

The WCH SDK exposes `HSADC_ChannelConfig(channel)`. The channel field is a
single 3-bit selection (`HSADC_CHSEL`), and the SDK has no HSADC channel
sequence table or scan-mode configuration. The official HSADC example also
selects one channel before starting DMA.

This gives the following boundary:

| Requested function | Result | Reason |
| --- | --- | --- |
| Four selectable analog inputs, one sampled at a time | Feasible | Extend the channel mapping and command range to four inputs. |
| Four simultaneous high-speed analog streams | Not supported by the current HSADC path | There is one converter and one channel-select field, not four converters. |
| Four interleaved streams at a claimed 20 Msps per channel | Not feasible | The 20 Msps limit belongs to the single HSADC aggregate path. |
| Existing firmware and host software used unchanged | Not feasible | Current commands and data structures describe one selected channel; the public port contains only the Windows host executable, not host source. |

## Required Work for Four Selectable Inputs

If the intended feature is four inputs that the user selects before capture,
the change is contained and does not require an additional ADC chip:

1. Confirm the four WEU6 HSADC pin mappings and route all four on the PCB.
2. Extend the firmware channel enum and GPIO/HSADC mapping from two entries to
   four entries.
3. Keep the existing one-channel capture format, or extend the command range
   while preserving the existing two-channel values for compatibility.
4. Add four input choices to the host software and validate the selected input
   end to end.

If the intended feature is four waveforms captured in one acquisition, this is
not a simple extension. It would require a different acquisition architecture,
a documented sample layout and USB protocol, a host decoder, and a measured
throughput/latency validation. The current public HSADC driver does not expose
the hardware sequence mechanism needed for this mode, so it should not be
promised for the CH32H417WEU6 design.

## Meaning of Round-Robin Sampling

There are two different interpretations of "four channels sampled in turn":

- **Sample-by-sample rotation** (`CH0`, `CH1`, `CH2`, `CH3`, repeat): the HSADC
  has no automatic sequence engine or DMA channel-switch mechanism for this.
  A CPU interrupt would have to rewrite the channel selection at conversion
  speed, which is not a reliable 20 Msps acquisition method.
- **Block-by-block rotation**: acquire a block from one channel, stop or reach
  a controlled DMA/burst boundary, select the next channel, and acquire the
  next block. This can be explored in firmware, but the result is four
  time-separated bursts rather than four simultaneous waveforms. The average
  per-channel rate is bounded by approximately one quarter of the aggregate
  HSADC rate, with additional loss from switching and restart overhead.

Block-by-block rotation would also require the USB stream to identify the
channel or define a fixed block order. The current host software and firmware
protocol do not contain that metadata.

## Sources

- [Original Q2H2 project](https://oshwhub.com/q2h2/project_bszkxrnf)
- [WEU6 project port README](https://github.com/Calvariaa/CH32H417_Logic_Analyzer_Mini)
- [WEU6 project firmware](https://raw.githubusercontent.com/Calvariaa/CH32H417_Logic_Analyzer_Mini/main/CH32H417_Logic_Analyzer_1v0/CH32H417_Logic_Analyzer_APP/Common/ch32h417_logic.c)
- [WCH official CH32H417 SDK repository](https://github.com/openwch/ch32h417)
- [WCH official HSADC header](https://raw.githubusercontent.com/openwch/ch32h417/main/EVT/EXAM/SRC/Peripheral/inc/ch32h417_hsadc.h)
- [WCH official HSADC driver](https://raw.githubusercontent.com/openwch/ch32h417/main/EVT/EXAM/SRC/Peripheral/src/ch32h417_hsadc.c)
- [WCH CH32H417 datasheet download page](https://www.wch-ic.com/downloads/CH32H417DS0_PDF.html)
- [CH32H417 product selection table mirror](https://cdn.promelec.ru/upload/items/2026/01/12/PRODUCT_GUIDE.PDF)
