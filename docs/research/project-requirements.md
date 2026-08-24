# Logic Analyzer Hardware Requirements

## Confirmed Product Constraints

- The product has 16 digital capture channels with a 200 MS/s target.
- The compact board uses CH32H417WEU6 and retains USB 3.0 SuperSpeed for the
  high-rate capture path.
- Supported probe logic standards are 1.8 V, 2.5 V, 3.3 V, and 5 V. There is
  no 1.2 V probe-logic mode. A possible 1.2 V *power* rail is a separate issue.
- CH32H417 `VIO18` must be externally supplied and shared with the buffer.
  WCH-RM v1.7 now proves PA5 reset is a floating input and DAC2 is disabled,
  but WCH does not establish an independent external-LDO handover or new CH2
  feedback values. This requirement therefore does not authorize changing the
  existing VIO18 schematic. See [the official analysis](ch32h417-external-vio18-dac-control-official-analysis.md).
- When HSADC is retained, `VDD33A` and `VDDIO` remain at the fixed 3.32-V
  rail. WEU6's unbonded `VREFP` is internally equal to `VDD33A`; `VIO18`
  remains the selected 1.80-V, 2.50-V, or 3.00-V capture domain. See
  [HSADC and VIO18 operating constraints](ch32h417weu6-hsadc-vio18-constraints.md).
- The probe-side path must stay high impedance. It includes RC filtering, ESD,
  and electronic-fuse protection. It has no 50-ohm input-termination
  requirement; any controlled-impedance target must be verified against the
  finished board stack-up.
- The compact layout target retains less than 50 mil channel-length mismatch.

## Digital Buffer Requirements

The signal direction is `probe -> buffer -> CH32H417 GPIO`. A suitable buffer
must therefore provide all of the following without another active level-shift
IC:

- 16 non-inverting channels, or an equivalently implemented 16-channel bank.
- Operation from the external `VIO18` range specified by the existing circuit.
- 0 V to 5 V-tolerant probe-facing inputs at every required buffer supply.
- Outputs limited to 0 V through `VIO18`, so that no 5 V reaches the MCU.
- No input bus-hold and no material reduction in probe input impedance.

Propagation delay alone is not a 200 MS/s qualification criterion because it
primarily shifts the capture time. The prototype must instead validate
short-pulse observability, inter-channel skew, output edge quality at the MCU
load, and routing together.

## Evidence Boundary

The constraints above are owner-confirmed product requirements. Current
design decisions use manufacturer primary documentation and the active LCEDA
project when it contains a saved schematic. Reference projects are not sources
for this design.
