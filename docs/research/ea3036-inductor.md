# EA3036C Output Inductor Selection

## Decision

Use `SHC252012-1R5M` for an EA3036CQBR output when the layout allows a
`252012` package. It is a 1.5 uH part and is the appropriate value in the
supplied SHC table for this regulator.

`252012` denotes the approximately 2.5 mm x 2.0 mm x 1.2 mm package class
(also commonly called 1008). It is not the smaller 0806 / 2016 package class,
so the PCB footprint must be designed for the actual vendor land pattern.

## EA3036CQBR Requirements

`EA3036CQBR` is the EA3036C three-channel synchronous buck regulator. The
vendor data sheet specifies a 2.7 V to 5.5 V input, 1.5 MHz nominal switching
frequency, 1 A continuous load per channel, 1.8 A minimum / 2.2 A typical
PMOS current limit, and recommends 1.0 uH to 2.2 uH output inductors. Its
typical circuit uses 1.5 uH on every output.

At 5 V input, 1.5 uH, and 1.5 MHz, the calculated inductor ripple is about
0.50 A at 3.3 V output and at most 0.56 A across a 1.84 V to 3.16 V adjustable
output range. A 1 A load therefore gives an approximately 1.28 A inductor
peak; even a 1.5 A transient load gives about 1.78 A.

Sources:

- [EA3036C data sheet, electrical characteristics and application information](https://datasheet.lcsc.com/datasheet/pdf/d83e9c52d5fbdb916908193b63b8ae60.pdf?productCode=C570857)
- [EA3036C data sheet mirror with searchable application-information text](https://w2.electrodragon.com/Chip-dat/ever-analog-dat/EA3036C-datasheet.pdf)

## SHC252012-1R5M Check

The supplied SHC table specifies the `SHC252012-1R5M` as:

| Parameter | Conservative value used for this check |
| --- | --- |
| Inductance | 1.5 uH |
| DCR | 53 mOhm maximum |
| Heat-rating current | 3.2 A |
| Saturation current | 3.6 A |

Those margins exceed the EA3036C's intended per-channel load and the calculated
normal inductor peak current by a wide margin. At 1 A DC output current, the
maximum DCR contributes at most 53 mV voltage drop and 53 mW winding loss.
Thus, neither DCR nor current capability is a reason to move up to a larger
inductor for this project.

The supplied image labels the second current subcolumn as "Max", although its
values are lower than the typical values. That is internally inconsistent for a
current rating, so this note deliberately uses the smaller values but does not
call them guaranteed limits. Before production, obtain the supplier's data
sheet and confirm both the current-rating convention and how `Isat` is defined
(for example, the allowed inductance drop). This is a procurement traceability
check, not a reason to reject the part for normal EA3036C use.

Source: [user-supplied SHC252012 electrical-characteristics image](../evidence/shc252012-electrical-characteristics.png).
