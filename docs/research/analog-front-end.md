# Analog Front-End Amplifier Choice

## Conditional Selection

When the reference project's high-bandwidth analog capture path is retained,
use `OPA2356`. `OPA2365` is not a direct substitute for that path.

| Parameter | OPA2356 | OPA2365 |
| --- | --- | --- |
| Gain-bandwidth | 200 MHz | 50 MHz |
| Slew rate | +300 / -360 V/us | 25 V/us |
| 0.01 percent settling, 4 V step | 120 ns | 300 ns |
| Input offset, typical | 2 mV | 100 uV |
| Input range | Ground to approximately V+ - 1.5 V | Rail-to-rail |

OPA2356 has the bandwidth and settling margin relevant to the original analog
capture topology. Its input common-mode limit must be respected by the existing
attenuation and bias network. OPA2365 is preferable for a lower-bandwidth,
precision, rail-to-rail ADC buffer, but its speed makes it unsuitable as a
direct replacement in the high-bandwidth path.

Sources: [OPA2356 data sheet](https://www.ti.com/lit/ds/symlink/opa2356.pdf),
[OPA2365 data sheet](https://www.ti.com/lit/ds/symlink/opa2365.pdf)
