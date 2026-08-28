# HD3SS3220 VBUS_DET Resistor Audit

## Scope

This audit checks every project-authored Markdown file that contains
`VBUS_DET`, `RVBUS`, `900 kOhm`, or `910 kOhm`. It does not alter the netlist
or supersede the board-level USB-C review.

## Official Evidence

The governing source is the current [TI HD3SS3220 data sheet, Rev. E](https://www.ti.com/lit/ds/symlink/hd3ss3220.pdf).

- Pin Functions, `VBUS_DET`: one `900 kOhm` external resistor is required
  between system VBUS and `VBUS_DET`.
- Section 5.3, Recommended Operating Conditions: `R(VBUS)` is listed as
  `880 kOhm` minimum, `900 kOhm` nominal, and `910 kOhm` maximum.
- Section 5.5, Electrical Characteristics: `RVBUS`, the external resistor
  between VBUS and `VBUS_DET`, is listed as `855 kOhm` minimum,
  `887 kOhm` typical, and `920 kOhm` maximum.
- Figure 6-1 explicitly shows `900 kOhm +/-1%`; therefore TI does not treat
  `900 kOhm` as a zero-tolerance purchased-resistor requirement.
- TI's public [HD3SS3220 schematic-review helper](https://e2e.ti.com/cfs-file/__key/communityserver-discussions-components-files/196/3220-schematic-review.pdf)
  explicitly lists a `910K` pull-up from VBUS for this pin.

`910 kOhm` is an E24 nominal value. With a `1%` part its resistance interval
is `900.9 kOhm` to `919.1 kOhm`, which is inside the published `855 kOhm` to
`920 kOhm` electrical-characteristics interval.

The data sheet does not state how the `880 kOhm` to `910 kOhm` recommended
row and the `855 kOhm` to `920 kOhm` electrical-characteristics row apply to
the tolerance endpoints of a purchased resistor. This project therefore does
not assert a precedence between the two rows. The documented basis for the
E24 choice is the calculated fit within the published `RVBUS`
electrical-characteristics interval together with TI's public `910K`
review-helper recommendation.

## Verified Project Documentation

- `docs/research/netlist-schematic1-2026-08-25-review.md` excludes the
  VBUS-detection resistor from the fabrication blockers, records the
  calculated `900.9 kOhm` to `919.1 kOhm` interval, and retains the single
  `910 kOhm, 1%` E24 selection.

- `docs/SELECTION.md:67` records the project connection as raw VBUS through
  `910 kOhm` to `VBUS_DET`.
- `docs/research/hd3ss3220-ufp-selection.md:25` selects one
  `910 kOhm, 1%` resistor.
- `docs/research/hd3ss3220-ufp-peripheral-review.md:40` correctly notes that
  the positive tolerance endpoint remains below `920 kOhm`.
- `docs/research/ch9441p-alternatives.md:61` uses `900 kOhm` as TI's nominal
  description, not as a project BOM instruction.

## Decision

Retain the existing single `910 kOhm, 1%, 0402` VBUS-to-`VBUS_DET` resistor.
No resistor-value change is required by the cited TI material under the
project's single-E24 constraint.
