# EA3036C and HD3SS3220 Power-On Timing

## Question

Can the current board topology prove the HD3SS3220 condition for permanently
asserting its active-low CC enable:

```text
VDD5 stable at least 2 ms before VCC33 is valid.
```

The answer is **no**. The topology has a natural causal order, but the EA3036C
data sheet does not publish the worst-case timing needed to turn that order
into a 2-ms guarantee.

## Current Netlist Facts

The supplied `Netlist_Schematic1_2026-08-25.enet` connects the electronic
fuse protected `+5V` rail to all of the following:

- EA3036C `VIN1`, `VCC`, and `EN1`;
- HD3SS3220 `VDD5`.

EA3036C channel 1 produces the board `3V3` rail, and that rail supplies
HD3SS3220 `VCC33`. The HD3SS3220 active-low `ENn_CC` input is tied to GND.

Thus, when protected `+5V` appears, the HD3SS3220 CC supply and the EA3036C
input/enable begin together. The 3.3-V rail can only rise after the EA3036C
starts switching. This does establish the intended direction of the sequence:
`VDD5` precedes `VCC33`. It does not quantify the interval between them.

## What the EA3036C Data Sheet Guarantees

The original Everanalog EA3036C Ver. 1.4 data sheet has these relevant entries:

| Item | Published value | What it proves |
| --- | --- | --- |
| `VINx`, `VCC` operating range | 2.7 V to 5.5 V | The 5-V rail is a valid supply input. |
| Input UVLO rising threshold | 1.7 V min, 1.9 V typ, 2.1 V max | Internal operation does not begin solely at 5 V. |
| Enable low level | 0.4 V max | A low input disables a channel. |
| Enable high level | 1.4 V min | A high input enables a channel. There is no specified upper switching threshold. |
| Enable-delay example | 100 kOhm from `VINx` to `ENx`, and 10 nF to 100 nF from `ENx` to GND | The user may program a nominal enable delay. |

The data sheet does **not** specify any of these as a minimum, typical, or
maximum timing parameter:

- `VIN` or `VCC` ramp time;
- `EN1` threshold maximum or input-current model;
- delay from `EN1` valid to switching start;
- soft-start duration;
- time for the 3.3-V output to reach its valid level;
- output-start behavior over load, output capacitance, input-ramp slope, and
  temperature.

Page 9 contains oscilloscope plots headed "Typical Operating Characteristics"
at `VIN = VCC = 5 V` and `TA = 25 C`, including VIN and EN power-on examples.
Those plots confirm that the converter normally starts after its input/enable
condition; they are not a timing specification and give no lower-bound
guarantee for this board.

Source: [Everanalog EA3036C Ver. 1.4 data sheet](https://datasheet.lcsc.com/datasheet/pdf/d83e9c52d5fbdb916908193b63b8ae60.pdf?productCode=C570857), pages 5, 9, and 11. The project-local original is
`docs/datasheets/everanalog-ea3036c-datasheet/EA3036C.pdf`.

## HD3SS3220 Requirement

HD3SS3220 has two supplies: `VDD5` for the CC controller and `VCC33` for the
SuperSpeed mux. Its `DIR` pin is a non-failsafe pin and is pulled up to the
board 3.3-V rail in this design. TI therefore permits a permanent low on
`ENn_CC` only if `VDD5` has been stable for at least 2 ms before `VCC33`.

If that condition cannot be guaranteed, TI requires `ENn_CC` to remain high
while both supplies ramp and for at least 2 ms after both are stable, then be
asserted low. The 2-ms values are explicit minimum timing requirements, not
typical guidance.

Source: [TI HD3SS3220 data sheet, Sections 5.6 and 6.3.12](https://www.ti.com/lit/ds/symlink/hd3ss3220.pdf).

## Why the Present Connection Cannot Prove 2 ms

For the direct-enable alternative, the required quantity is:

```text
t_margin = time(VCC33 becomes valid) - time(VDD5 becomes stable)
t_margin >= 2 ms in every permitted condition
```

The EA3036C documentation does not provide an upper bound for the 3.3-V-valid
event, or a bounded delay relative to VDD5 stabilization. In particular, the
`EN1 = +5V` connection adds no intentional external delay. The data sheet's
typical startup plot cannot supply the missing worst-case limits. Consequently,
the present schematic supports the engineering intuition that the 5-V rail will
normally arrive first, but it cannot support a data-sheet-level claim that the
interval is always at least 2 ms.

This remains true across the cases that matter for the TI requirement:
cold start, hot plug, slow upstream ramp, input brownout/recovery, different
EA3036C samples, output-capacitor tolerance, load, and temperature.

## Minimal Change Without an Additional IC

There is **no component-level guaranteed no-new-IC fix that can be derived
from the EA3036C data sheet alone**.

An RC from `+5V` to EA3036C `EN1` follows the vendor's suggested topology and
can create an intentional *nominal* startup delay. It is reasonable as a
prototype experiment, but it cannot be assigned a guaranteed 2-ms-or-greater
margin because the data sheet omits the enable threshold upper bound and the
maximum enable-to-3.3-V-valid time.

Likewise, a passive RC attached to HD3SS3220 `ENn_CC` cannot verify that 3.3 V
is valid and cannot, from the available timing data alone, guarantee the
required high level or delay through all power-cycle conditions.

Therefore the options are explicit:

1. Keep `ENn_CC` permanently low and treat the 2-ms condition as a first-board
   measurement/acceptance requirement, not a guaranteed design property.
2. Add a voltage-monitor/open-drain timing function that watches the 3.3-V
   rail, holds `ENn_CC` high from `VDD5`, and pulls it low only after the
   3.3-V-valid threshold plus a specified delay. This is the minimum route to
   a component-specification-level guarantee; it requires an additional active
   device.

The second option is not required merely because the EA3036C power direction is
wrong; it is required only when the project requires the TI 2-ms condition to
be guaranteed before hardware measurement.
