# TPS25200DRVR Peripheral Selection

## Scope and Conclusion

This note selects the peripheral network for the project power path:

```text
USB-C VBUS 5 V -> TPS25200DRVR -> EA3036CQBR -> three buck outputs
```

The selection is based on the project requirements and the TI TPS25200
data sheet/EVM documents. It does not copy the former TPS259251 values.

Recommended first prototype values:

| Function | Recommended value | Part requirement | Decision |
| --- | ---: | --- | --- |
| TPS25200 IN local bypass | `0.1 uF` | X7R, `50 V`, ceramic, `0603`, placed at IN-GND | Required by TI data sheet |
| Additional IN bulk ceramic | `10 uF` | X7R/X5R, `25 V`, ceramic, `1206`, placed in the same input power area | Recommended for this USB-C-to-EA3036 path |
| TPS25200 OUT local capacitor | `22 uF` | X7R/X5R, `16 V`, low-ESR ceramic, `1206` | Recommended starting value from TI typical application |
| RILIM | `75 kOhm` candidate | E24 common value, `1%`, `0603`, short connection to ILIM-GND | Valid only if the finalized board budget is at most 1.10 A; see current-limit analysis |
| EN pull-up | `300 kOhm` | `1%`, `0603`, IN/VBUS to EN | Recommended for always-on operation; no RC capacitor |
| FAULT | `NC` | No pull-up or MCU route fitted | No fault-observation requirement exists in the current project |
| IN TVS | DNP by default | Optional port-protection footprint | Not required by TPS25200 itself |

The `10 uF` input capacitor is not a TPS25200 minimum requirement. The
minimum local requirement from TI is `0.1 uF` ceramic. The additional `10 uF`
is a practical first-prototype choice because the device is fed from a
hot-pluggable USB-C VBUS and drives the input capacitors of EA3036C.

Primary sources: [TPS25200 data sheet, Rev. F](https://www.ti.com/lit/ds/symlink/tps25200.pdf)
and [TPS25200EVM-618 User's Guide](https://www.ti.com/lit/pdf/SLVUA53).
Project requirements: [Logic Analyzer Hardware Requirements](project-requirements.md),
[EA3036C inductor selection](ea3036-inductor.md), and [hardware selection](../SELECTION.md).

## Project Power Basis

The fixed EA3036C rails are `3.32 V` and `1.218 V`; the external `VIO18` rail
uses the existing schematic implementation. No official evidence in this
project yet establishes its maximum voltage or total load, so the final TPS25200
current budget and `RILIM` value are not frozen here.

This is a design allocation, not a claim that every source supplies 1 A.
Default-current Type-C operation must remain below `450 mA`; firmware enables
the full-rate mode only after HD3SS3220 reports Medium (1.5 A) or High (3 A)
source current. See [Power Architecture](power-architecture.md) for the
complete rail and source-current rationale.

The TPS25200 is specified for up to `2.6 A` continuous output current and has a
programmable current-limit range of `85 mA` to `2.9 A` typical. The selected
current limit therefore needs to protect the USB-C source without nuisance
tripping while the EA3036 input capacitors charge. TI explicitly recommends
checking the minimum current-limit threshold against startup into a full or
heavy capacitive load.

Sources: [TPS25200 data sheet, product page](https://www.ti.com/product/TPS25200),
[TPS25200 data sheet, Rev. F](https://www.ti.com/lit/ds/symlink/tps25200.pdf).

## IN and OUT Capacitors

### IN

TI specifies a `0.1-uF or greater ceramic bypass capacitor` from IN to GND as
close to the IC as possible. The TI EVM uses both a `0.1 uF, 50 V` ceramic
capacitor and a `10 uF, 25 V` ceramic capacitor on the input side.

For this project use:

- `C_IN_HF = 0.1 uF, X7R, 50 V, ceramic`, directly beside TPS25200 pin 6 and
  the ground return;
- `C_IN_BULK = 10 uF, X7R or X5R, 25 V, ceramic`, in the short USB-C VBUS to
  TPS25200 input-current loop.

The `25 V` rating gives useful margin for the TPS25200 input withstand
capability, but it does not turn the board into a 20 V operating product.
The TPS25200 IN pin can withstand up to `20 V` DC, while its recommended
operating VIN range is only `2.5 V` to `6.5 V` and TI describes normal power
rails as below `5 V` in the power-supply recommendation. If the USB-C source is
guaranteed to be a compliant 5 V source, a lower capacitor voltage rating can
be electrically sufficient, but `25 V` is the preferred prototype rating.

The existing EA3036C input capacitors are downstream capacitors. They do not
replace the `0.1 uF` high-frequency bypass capacitor at TPS25200 IN. Keep the
EA3036C VIN1/VIN2/VIN3 and VCC local capacitors at the regulator as specified
by its own data sheet.

### OUT

TI's typical TPS25200 application uses `22 uF` at OUT, and the EVM provides
selectable `22 uF`, `47 uF`, and `150 uF` output capacitance. TI recommends a
low-ESR ceramic output capacitor and notes that more output capacitance reduces
the output peak when the input is driven above the overvoltage-lockout level.

For the first project prototype use `C_OUT = 22 uF, X7R/X5R, 16 V, low-ESR
ceramic` directly at TPS25200 OUT-GND. Keep the EA3036C input capacitors at
the EA3036 pins; do not rely on a long trace between those capacitors and
TPS25200 OUT as the only local output bypass.

The `16 V` rating is selected because TPS25200's normal output is a 5 V rail,
while its specified output clamp can reach `5.55 V` maximum. This rating is a
component margin choice, not a claim that the output is regulated at 5 V under
all fault conditions.

Sources: [TPS25200 data sheet, typical application and capacitance guidance](https://www.ti.com/lit/ds/symlink/tps25200.pdf)
and [TPS25200EVM-618 BOM](https://www.ti.com/lit/pdf/SLVUA53).

## RILIM Selection

Use a `1%` resistor in the TI-recommended range `33 kOhm <= RILIM <= 1100 kOhm`.
`75 kOhm` is an E24 common value. Applying TI's Equation 1 with resistor
tolerance gives approximately `1.175 A` minimum and `1.420 A` maximum current
limit. It stays below the 1.5-A Type-C Medium advertisement and would provide
`0.075 A` margin only for a separately established 1.10-A board budget.

This candidate is valid only with the firmware source-current policy above and
after the actual maximum board budget is established. It is not a license to
draw 1.10 A from a Default-current source. Do not freeze this resistor from an
assumed external-VIO18 voltage range.

RILIM must be placed immediately beside the ILIM pin, with a very short trace
to the device ground. TI states that this trace must be kept short to reduce
parasitic effects on current-limit accuracy.

## EN Connection

EN is an active-high TTL/CMOS-compatible logic input. It cannot float. TI gives
an input rising threshold of `1.9 V` typical and a falling threshold of `0.6 V`
typical. The EN recommended operating range is `0–6.5 V`.

For the project's always-on USB-C power path, use:

```text
TPS25200 IN/VBUS --- 300 kOhm, 1% --- EN
```

TI explicitly shows a `300 kOhm` pull-up from VIN to EN and explains that the
large resistor limits current into the integrated EN protection zener. Do not
add an EN capacitor or RC delay unless the new system specification requires
delayed enable. A direct copper connection from a clean 5 V VBUS is within the
EN operating range, but the `300 kOhm` pull-up is the preferred implementation
for this design because USB-C VBUS can experience abnormal transients and the
TI typical circuit uses the resistor.

If firmware-controlled disabling is later required, EN must still have a
defined high/low state and the control connection must be checked against the
CH32H417's selected `VIO18` voltage. In particular, do not assume a 1.8 V
MCU output is a guaranteed high for a TPS25200 input whose typical turn-on
threshold is `1.9 V`. This note therefore recommends always-on EN and does not
introduce an additional active control IC.

## FAULT Pin Disposition

FAULT is an active-low open-drain output. It indicates overcurrent,
overtemperature, and OVLO shutdown. It does **not** assert while the device is
in its `5.4 V` output-overvoltage-clamp (OVC) mode.

Leave FAULT electrically `NC` in the current project. The earlier proposal to
pull it up to `VIO18` and route it to the CH32H417 is withdrawn: both `VIO18`
and the MCU are powered from the EA3036C downstream of TPS25200 OUT, so they
cannot reliably remain alive while a TPS25200 fault removes or collapses OUT.
The overcurrent flag also has an approximately `8 ms` deglitch, and a hard
short can hold OUT near ground before that interval expires.

The pin needs a pull-up to a live low-voltage I/O rail only if a future design
defines a fault-observation function with a supply domain that remains valid
during the relevant fault. That is a new architectural requirement; no
pull-up resistor or MCU GPIO is selected now. See
[TPS25200 FAULT observability](tps25200-fault-observability.md) for the
fault-by-fault evidence.

## TVS Requirement

An input TVS is **not a required TPS25200 peripheral**. TI states that, in its
USB hot-plug application, TPS25200 can withstand the cable-induced input
transient without an input bulk bypass capacitor or other external input
overvoltage-protection component. The device also has a `7.6 V` typical input
overvoltage lockout and a `5.25–5.55 V` output clamp specification.

That does not make the internal protection a replacement for a USB-C port
protection design:

- the input lockout disconnects the load; it does not clamp VBUS to 5 V;
- the output clamp acts on OUT, not on the USB-C connector side of IN;
- a TVS decision depends on the USB-C cable, source compliance, ESD target,
  and any required IEC test level, none of which is frozen in the current
  project requirements.

For this project, keep an IN-to-GND TVS footprint as `DNP` for the first
prototype rather than making it a mandatory TPS25200 component. Populate it
only if the USB-C port-level transient test requires it. No specific TVS model
is selected in this note because the required standoff voltage, clamping
voltage, pulse power, and port-level ESD standard have not been specified.

## Layout Requirements

Follow the TI layout guidance and keep the following loops compact:

1. Place the `0.1 uF` IN capacitor directly beside pin 6 and route its ground
   to the TPS25200 ground/PowerPAD with the shortest practical path.
2. Place the `22 uF` OUT capacitor directly beside pin 1 and its ground return.
3. Keep the USB-C VBUS-to-IN and OUT-to-EA3036 input-current paths short and
   wide. Do not route sensitive signals through these switching current paths.
4. Place RILIM immediately beside pin 2. Keep its trace short and connect its
   return to the local power ground.
5. Keep EN away from the high-current IN/OUT copper and the USB-C connector
   transient path. EN needs no RC in the recommended always-on implementation.
6. Connect pin 5 GND and the exposed PowerPAD directly to the PCB ground plane
   with a wide, short connection and thermal vias as allowed by the footprint.
7. Use the actual `DRV` WSON-6 land pattern and follow TI's exposed-pad
   assembly recommendation; this is a new layout, not a TPS259251 footprint
   conversion.

## Unresolved Items Before Release

- Measure the actual 5 V steady-state current and startup/inrush current of
  EA3036C with all three channels and all project loads enabled.
- Confirm the USB-C source-current contract. If the source is limited below
  the selected current-limit threshold, TPS25200 cannot create additional
  available power.
- Verify the TPS25200 junction temperature at the maximum continuous current;
  its `60 mOhm` typical on-resistance and small WSON package make copper area
  and thermal vias material.
- Test USB-C hot plug, input overshoot at pin 6, OUT clamp behavior, and
  EA3036 startup with `C_IN_HF + C_IN_BULK + C_OUT` installed.
- Decide the port-level ESD/TVS requirement separately from the TPS25200
  minimum functional circuit.

## Sources

- [TI TPS25200 product page](https://www.ti.com/product/TPS25200)
- [TI TPS25200 data sheet, Rev. F](https://www.ti.com/lit/ds/symlink/tps25200.pdf)
- [TI TPS25200EVM-618 User's Guide](https://www.ti.com/lit/pdf/SLVUA53)
- [Project requirements](project-requirements.md)
- [EA3036C inductor selection](ea3036-inductor.md)
- [Power Architecture](power-architecture.md)
- [Hardware selection](../SELECTION.md)
