# Logic Analyzer Hardware Requirements

## Confirmed Product Constraints

- The product has 16 digital capture channels with a 200 MS/s target.
- The compact board uses CH32H417WEU6 and retains USB 3.0 SuperSpeed for the
  high-rate capture path.
- Supported probe logic standards are 1.8 V, 2.5 V, 3.3 V, and 5 V. There is
  no 1.2 V probe-logic mode. A possible 1.2 V *power* rail is a separate issue.
- One globally selected `VIO18` rail sets both the digital-buffer supply and
  the capture GPIO voltage domain. The documented adjustable range is 1.84 V
  through 3.16 V.
- The probe-side path must stay high impedance. It includes RC filtering, ESD,
  and electronic-fuse protection. The stated 50-ohm goal is controlled trace
  impedance, not a 50-ohm input termination requirement.
- The compact layout target retains less than 50 mil channel-length mismatch.

## Digital Buffer Requirements

The signal direction is `probe -> buffer -> CH32H417 GPIO`. A suitable buffer
must therefore provide all of the following without another active level-shift
IC:

- 16 non-inverting channels, or an equivalently implemented 16-channel bank.
- Operation from the real `VIO18` range of 1.84 V through 3.16 V.
- 0 V to 5 V-tolerant probe-facing inputs at every selected `VIO18` voltage.
- Outputs limited to 0 V through `VIO18`, so that no 5 V reaches the MCU.
- No input bus-hold and no material reduction in probe input impedance.

Propagation delay alone is not a 200 MS/s qualification criterion because it
primarily shifts the capture time. The prototype must instead validate
short-pulse observability, inter-channel skew, output edge quality at the MCU
load, and routing together.

## Primary Evidence

- [Reference project 1](https://oshwhub.com/aaahaley/project_xkzwqync)
- [Reference project 2](https://oshwhub.com/calvaria/project_slbqmtir)
- [Project 2 16-channel / 200 MHz README statement](https://github.com/Calvariaa/CH32H417_Logic_Analyzer_Mini/blob/main/README.md#L14-L15)
- [Project 2 DAC-controlled VIO implementation](https://github.com/Calvariaa/CH32H417_Logic_Analyzer_Mini/blob/main/CH32H417_Logic_Analyzer_1v0/CH32H417_Logic_Analyzer_APP/Common/hardware.c#L24-L31)
- [Project 2 analog / logic VIO switching](https://github.com/Calvariaa/CH32H417_Logic_Analyzer_Mini/blob/main/CH32H417_Logic_Analyzer_1v0/CH32H417_Logic_Analyzer_APP/Common/hardware.c#L273-L302)
- [Project 2 200 MHz capture mode](https://github.com/Calvariaa/CH32H417_Logic_Analyzer_Mini/blob/main/CH32H417_Logic_Analyzer_1v0/CH32H417_Logic_Analyzer_APP/Common/ch32h417_logic.c#L66-L75)
