# Logic Capture and HSADC Layout Rules

## Scope and Evidence Boundary

This document defines layout rules for the current compact
`CH32H417WEU6` logic-analyzer design:

- 16 single-ended logic inputs, `probe -> SN74LVC16244A -> CH32H417`.
- A 200 MS/s capture target.
- One 20 MS/s HSADC conversion path. The optional analog front end uses
  `OPA2356` when retained.
- One product-required external `VIO18` capture domain shared by the MCU
  high-speed pins and the digital buffer. Its control implementation is the
  existing schematic and is not redefined by this document.

The rules below are derived from the project requirements and the WCH/TI data
sheets. They do **not** infer requirements from a reference project's
schematic or PCB routing. All numerical trace, spacing, and length limits
below are project design targets unless explicitly identified as a WCH or TI
requirement.

## Hard Electrical Facts

| Fact | Layout consequence |
| --- | --- |
| CH32H417 has one HSADC, an analog input range of `VSS` to `VDDIO`, and a 20 MS/s maximum rate. | There is no simultaneous two-ADC timing pair to length-match. Keep each analog path quiet and short instead. |
| `VDD33A` powers the HSADC analog circuitry and should be at least 3.0 V when HSADC is used. WCH specifies a local 0.1 uF capacitor. | Keep the HSADC analog-supply decoupling at the MCU, with a direct ground return. |
| `VREFP` is the ADC/HSADC/DAC reference. WCH specifies a local 0.1 uF capacitor and requires `VREFP <= VDD33A`. | Treat `VREFP` as an analog-reference node, not as an ordinary power trace. The capacitor-to-ground loop is more important than trace width. |
| `VIO18` supplies high-speed I/O; WCH specifies 0.1 uF at each pin and 2.2 uF at the main `VIO18` adjacent to `VDDIO`. | Route the external regulator output to the MCU and buffer separately after its local output capacitor. Keep the buffer bypass local and do not share its return path with switch-node current. |
| SN74LVC16244A is a fast CMOS buffer. TI specifies up to 1 ns output skew over -40 C to 85 C and up to 1.5 ns over -40 C to 125 C. | PCB length matching can make board skew negligible, but it cannot eliminate device channel skew. |
| The OPA2356 data sheet calls for high-frequency layout, a ground plane, short direct traces, and bypassing at `V+`; it specifies 10 uF ceramic as the minimum bypass capacitance. | The OPA supply loop and its output-to-HSADC path must remain local to the analog area. |

## 200 MS/s Timing Boundary

At one sampling edge per sample, 200 MS/s gives a 5 ns sample interval. WCH
documents UHSIF as a 125 MHz maximum interface and does not publish an
external-input setup/hold budget, required impedance, or an allowed PCB skew
for a 200 MHz logic-capture use case.

Therefore:

- 200 MS/s is a project performance target, not an official WCH timing
  guarantee.
- No defensible IC-data-sheet "maximum length mismatch" exists for this use
  case.
- The PCB must minimize its own skew, then a prototype must validate all
  channels with fast edges and short pulses at the intended `VIO18` values.

For scale only, a 50 mil (1.27 mm) copper-length mismatch on a normal FR-4
outer-layer trace is roughly 8 to 10 ps. A 250 mil (6.35 mm) mismatch is
roughly 40 to 50 ps, still below 1% of a 5 ns sample interval and far below
the buffer's specified channel-to-channel skew. Therefore a 50 mil result is
an optional ultra-tight placement result, not an electrical requirement.

## Routing Classes

These values assume a four-layer board with a continuous L2 ground plane. If
the production stack-up differs, retain the topology rules and obtain actual
impedance from the board fabricator before assigning an impedance value.

| Net class | Width and layer | Length rule | Other rules |
| --- | --- | --- | --- |
| `LOGIC_RAW[15:0]`: final protection element to buffer input | 6 mil nominal, preferably L1 over solid L2 ground | Keep each unbuffered section no longer than 10 mm where placement permits, and include it in the 16-channel total-length report. | No vias or test-pad stubs. Keep the buffer close to the last protection element. |
| `CAPTURE_D[15:0]`: buffer output to CH32H417 UHSIF input | 6 mil nominal, all 16 on the same routing layer where possible | Keep the group mismatch at or below 100 mil (2.54 mm), measured from buffer output pad to MCU pad. | Zero signal vias is the target; one is the maximum if unavoidable. Use the same via count and reference-plane topology for every channel. |
| `AN_HIGH_Z[n]`: the analog amplifier input node after its attenuation/protection network | 6 mil nominal, L1 over L2 ground | No longer than 5 mm; direct route only. | No via, test pad, branch, parallel digital run, or copper from a switch node below it. |
| `AN_DRV[n]`: OPA2356 output to HSADC input | 6 mil nominal, L1 over L2 ground | Target no longer than 10 mm; 15 mm is the placement limit. | No via, test pad, branch, or length-tuning meander. |
| `VREFP`, `VDD33A`, OPA supply bypass loops | Use the widest practical copper, not the 6 mil signal class. | Minimize capacitor-to-pin-to-ground-loop area. | Place the ground via at the capacitor ground pad; do not route this return through digital ground current. |

Six mil is electrically adequate for all four signal classes above. It is a
routing/manufacturing width, not a declaration of 50 ohm impedance. The logic
capture bundle and the HSADC path do not have a WCH or TI requirement to be
50 ohm terminated transmission lines. Do not force a line to a claimed 50 ohm
width without the actual finished stack-up.

## Digital Capture Placement and Routing

1. Place the 16-channel buffer between the input protection bank and the MCU.
   The priority is a short, direct buffer-output-to-MCU path; the final
   unbuffered input segment is the next priority.
2. Route `CAPTURE_D[15:0]` as a homogeneous single-ended bus over uninterrupted
   ground. Do not cross an L2 split, void, anti-pad field, or plane-neck.
3. Use at least 12 mil edge-to-edge spacing (2W for 6 mil traces) between
   parallel `CAPTURE_D` runs. Use 18 mil (3W) where two routes run parallel for
   more than 10 mm. Brief pin-escape sections may be closer when no practical
   alternative exists.
4. Do not route a capture trace under or adjacent to the EA3036 switch-node,
   inductor, USB SuperSpeed lanes, or a high-current power loop. If an analog
   trace and a capture trace must cross, cross at 90 degrees with the L2 ground
   plane intact.
5. Do not compensate a long raw-input route by adding a large serpentined
   output route. Match by component placement and connector pin order first;
   use only broad, well-spaced tuning segments as a last step.
6. Tie all buffer enable/control inputs to defined states. TI explicitly warns
   that multi-bit logic inputs must not float. Place at least the local 0.1 uF
   `VIO18` bypass capacitor at the buffer supply pin with a short ground via.

Keep each channel's total board copper path within 250 mil (6.35 mm) of the
others, measured from the equivalent node after its per-channel protection
network to the MCU pad. The external probe cable and package-internal delay
are excluded. Run a separate 100 mil check over the `buffer Y pin -> MCU pad`
16-net group, because it contains the fastest board-side edges. Report the
raw-input and buffer-output segments separately; do not hide a large
input-side mismatch by electrical-length compensation after the buffer. A
50 mil result may be kept if it falls out of clean placement, but do not add
large serpentine sections just to reach it.

## HSADC and OPA2356 Placement and Routing

The CH32H417 contains one HSADC, so separately routed analog inputs do not
form a simultaneous ADC timing pair. Analog front ends should have matching
topology and passive values for consistent amplitude/frequency response, but
they must **not** be length-tuned to each other. A longer meander only adds
capacitance and pickup.

1. Keep every analog channel in one local zone: connector/protection and
   attenuation network, OPA2356, then the MCU HSADC pin. Do not send the
   amplifier input or output through the digital-capture bus.
2. Put the attenuation/protection components beside the OPA input. The
   high-impedance node between them is the most sensitive analog trace and
   uses the 5 mm/no-via rule above.
3. Put the OPA2356 supply bypass directly at its `V+` pin. The data sheet's
   10 uF minimum ceramic bypass is an electrical requirement to carry into the
   schematic and layout; do not replace it with a remote shared bulk capacitor.
4. Put the WCH-required 0.1 uF capacitors directly at `VDD33A` and `VREFP`.
   Use a separate, direct return to the uninterrupted L2 ground plane. Keep
   these capacitor loops out of the buffer and DC/DC return-current paths.
5. Do not add a capacitor, series resistor, or RC value at the HSADC pin solely
   from a generic layout rule. WCH does not publish the HSADC sampling-capacitor
   or source-impedance requirement needed to justify a numeric value. Freeze
   the front-end network from measured settling, bandwidth, and noise results.

## Layout Review Checklist

- All 16 buffer-output capture traces are 6 mil, reference the same continuous
  ground plane, have no test stubs, and satisfy the 100 mil group target;
  full board-channel copper paths satisfy the 250 mil target.
- No capture net changes layer; if one must, all affected routes retain the
  same via count and a nearby ground-return via.
- The buffer has local external `VIO18` bypassing and every OE/control input is defined.
- `VDD33A` is at least 3.0 V for HSADC operation, `VREFP <= VDD33A`, and their
  0.1 uF capacitors are at the MCU pins.
- Each OPA2356 has its local 10 uF ceramic bypass, a sub-5 mm high-impedance
  input node, and a no-via output route no longer than 15 mm to its HSADC pin.
- The 200 MS/s validation plan measures ringing and capture consistency at all
  selected `VIO18` voltages; PCB discipline alone cannot close the undocumented
  200 MHz UHSIF timing margin.

## Sources

- [Project hardware requirements](project-requirements.md)
- [WCH CH32H417 data sheet](https://www.wch-ic.com/downloads/CH32H417DS0_PDF.html)
- [WCH UHSIF development reference manual](https://github.com/openwch/ch32h417/blob/main/EVT/EXAM/UHSIF/CH32H417%E9%80%9A%E7%94%A8%E9%AB%98%E9%80%9F%E6%8E%A5%E5%8F%A3%EF%BC%88UHSIF%EF%BC%89%E5%BC%80%E5%8F%91%E5%8F%82%E8%80%83%E6%89%8B%E5%86%8C.pdf)
- [TI SN74LVC16244A data sheet](https://www.ti.com/lit/ds/symlink/sn74lvc16244a.pdf)
- [TI OPA2356 data sheet](https://www.ti.com/lit/ds/symlink/opa2356.pdf)
