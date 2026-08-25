# TPS25200 Enable Resistor and HD3SS3220 Power-On Timing

## Decision

For the always-on protected USB input, connect the TPS25200 `EN` input to
raw `VBUS` through one `330 kOhm`, `±5%`, single `0402` resistor. Do not use
a direct copper short, and do not add an RC capacitor to this input.

For HD3SS3220, a dedicated voltage monitor is **not inherently required by
the function of the USB-C controller**. It is required only if the product
needs the HD3SS3220 power-on requirement to be met by a circuit with a
component-level guarantee. The present topology does not establish either
permitted timing case, so keeping `ENn_CC` tied low is not a data-sheet-backed
choice. A monitor with an open-drain output, such as TPS3898A, is one direct
way to create that guarantee; it is not the only conceivable circuit.

## Existing Resistor Value and Package

The current netlist has no single high-value resistor in the 240--360 kOhm
range. It has four `330 kOhm`, `±5%`, `0402x4` isolated resistor arrays
(Yageo `YC124-JR-07330KL`). Every resistor element in each array is already
connected as a signal-input pull-down, so no element can be repurposed for the
USB input without changing the capture-input circuitry.

Use the same already-qualified nominal value as a new **single** `330 kOhm`,
`±5%`, `0402` resistor. Its tolerance range is 313.5--346.5 kOhm, entirely
above TI's stated 300-kOhm pull-up example. A 1% tolerance provides no useful
functional benefit here: this resistor neither sets a timing constant nor an
analog threshold.

## TPS25200 `EN` Connection

TI specifies all of the following for `EN`:

- It cannot be left floating.
- A direct connection to a clean, compliant 5-V USB `VBUS` is electrically
  within the recommended `EN` range of 0--6.5 V.
- This is not equivalent to a robust connection to the TPS25200 input node:
  `IN` has a 20-V absolute-maximum rating, while `EN` has a 7-V
  absolute-maximum rating.
- TI explicitly permits `EN` to be tied to `VIN` **with a pull-up resistor**;
  the internal zener clamps EN and TI calls for a sufficiently large
  `300 kOhm` pull-up to hold EN below its absolute maximum rating.

Thus, `330 kOhm` from raw `VBUS` to `EN` follows the function of TI's 300-kOhm
example while reusing the project's existing nominal value. It changes neither
the normal enable state nor the TPS25200 startup delay. It limits current into
the EN clamp when raw VBUS is abnormal. A direct short works only under the
extra system assumption that raw VBUS will never exceed the EN rating; it
throws away the protection margin TI designed into the resistor connection.

Source: [TI TPS25200 data sheet, Rev. F, pin-functions table, absolute/recommended ratings, and Section 7.3.1](https://www.ti.com/lit/ds/symlink/tps25200.pdf).

## Why HD3SS3220 Has a Timing Requirement

HD3SS3220 has two separate supplies. `VDD5` powers the CC controller and
VCONN path, while `VCC33` powers the SuperSpeed mux. Its PORT, ADDR, I2C,
interrupt/output, VCONN fault, and direction pins are non-failsafe.

The TI data sheet provides exactly two permitted startup cases when any of
those pins is pulled up to a supply other than `VDD5`:

1. `ENn_CC` may remain low only if `VDD5` is stable at least **2 ms before**
   `VCC33`.
2. Otherwise, keep `ENn_CC` high while both supplies ramp, then hold it high
   for at least **2 ms after both are stable**, and only then drive it low.

The condition prevents a non-failsafe pin supplied from the 3.3-V domain from
back-powering or partially biasing internal circuitry before the 5-V domain is
valid. It is a specified power-on condition, not a recommendation for lower
noise or a controller startup preference.

The current netlist ties `ENn_CC` directly to ground. It feeds `VDD5` from the
protected 5-V rail and derives `VCC33` from a buck channel whose enable is
tied directly to that same 5-V rail. This shows common origin, but proves
neither the required 2-ms lead of `VDD5` nor the controlled-enable sequence.
It can work on a particular assembled board, but it is not guaranteed by the
published component specifications.

Source: [TI HD3SS3220 data sheet, Rev. E, Section 6.3.12 and power-on timing table](https://www.ti.com/lit/ds/symlink/hd3ss3220.pdf).

## Does an RC-Only Alternative Avoid a New IC?

A simple RC directly from `ENn_CC` to ground is wrong in polarity: `ENn_CC`
is active low, so a capacitor to ground initially holds it low and enables the
CC controller immediately. An inverted passive delay (capacitor coupled from
`VDD5` to `ENn_CC`, resistor to ground, with required clamp/reset treatment)
can delay the falling edge and may be made to work in a measured prototype.

It is not a data-sheet guarantee in this design. The current three-channel
buck data sheet gives only `V_EN-L <= 0.4 V` and `V_EN-H >= 1.4 V`; it does
not specify an enable-threshold maximum, an output-start-time maximum, or a
power-good output. Therefore neither an RC delay added to the 3.3-V buck
enable nor an RC delay added to `ENn_CC` can prove, across tolerance,
temperature, plug-in ramp, brownout, and discharge conditions, that `VCC33`
is stable and `ENn_CC` remains high for the specified 2 ms.

The no-new-IC option is valid only under a measurement-based product rule:
scope `VDD5`, `VCC33`, and `ENn_CC` at the IC pins during cold start, hot plug,
brownout, and cable removal, and accept it only if every required condition is
demonstrated for the released component and layout. It remains an empirical
qualification, not a component-specification guarantee.

## Why TPS3898A Is a Guarantee Path

TPS3898A is an active-low-enable, active-low, open-drain voltage monitor.
With its supply on protected `VDD5`, its sense divider on `VCC33`, and its
open-drain output connected to `ENn_CC` with a pull-up to `VDD5`, it holds
`ENn_CC` at the required `VDD5`-relative high level while 3.3 V is invalid.
Once 3.3 V crosses the chosen threshold, the programmed capacitor delay holds
that state longer than 2 ms before the monitor pulls `ENn_CC` low.

The existing documented divider of 422 kOhm / 82.5 kOhm gives a nominal
3.06-V threshold from the TPS3898 0.5-V threshold. With 47 nF on CT, the TI
equation `t = C(uF) x 4 s + 40 us` gives approximately 188 ms, so it exceeds
the HD3SS3220 2-ms requirement by a large margin. It is deliberately longer
than necessary but valid. The open-drain output, rather than a 3.3-V push-pull
high output, is important because HD3SS3220 requires `ENn_CC` high to reach
`VDD5 - 0.3 V`.

TPS3898A is therefore necessary only if the requirement is "guaranteed by the
published circuitry" rather than "shown by board-level measurements." It is
the lowest-risk selection for the present topology because it directly checks
the rail that must be stable.

Sources: [TI TPS389x data sheet, device comparison, truth table, CT equation, and open-drain output specification](https://www.ti.com/lit/ds/symlink/tps3896.pdf); [Everanalog EA3036C data sheet, enable electrical characteristics](https://datasheet.lcsc.com/szlcsc/2005300208_Everanalog-IC-design-EA3036C_C570857.pdf).

## Required First-Board Verification

Regardless of the selected path, capture `VDD5`, `VCC33`, and `ENn_CC` at the
HD3SS3220 pins for cold start, hot plug, brownout, and cable removal. Check
the actual high and low levels against the HD3SS3220 limits: `ENn_CC` high is
at least `VDD5 - 0.3 V`, low is at most 0.4 V, and the relevant 2-ms timing
case above is met.
