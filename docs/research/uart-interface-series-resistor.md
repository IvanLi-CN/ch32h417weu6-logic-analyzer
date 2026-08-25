# Debug UART Interface Series Resistors

## Scope

This note concerns only the debug-UART connection being migrated to the
CH32H417WEU6. It does not infer a UART requirement from unrelated MCU alternate
functions or from a resistor whose complete net topology has not been verified.

## Reference Evidence

### Project B: `calvaria/project_slbqmtir`

The accessible `Schematic-mini0v1` sheet connects `U1.PD1` directly to the
two-pin debug connector `H2` as `TX1`; `H2` pin 1 is ground. There is no series
resistor on this net. Its firmware configures `PD1` as `USART6_TX` and enables
transmit mode only.

This is the relevant reference for the compact WEU6 debug-output connection.

### Project A: `aaahaley/project_xkzwqync`

The read-only `P1.Schematic1` view shows `R25 = 499 ohm` between `U5` pin 88
(`PA9/OTG_VBUS`) and an otherwise anonymous net. It also shows a paired resistor
at `U5` pin 89 (`PA10/OTG_ID`). The complete far-side topology of these two
resistors is not exposed by the available read-only view, so their interface and
purpose are not established.

`PA9` and `PA10` have USART1 alternate functions, but their pin names also
identify OTG functions. Pin multiplexing alone cannot prove that either
resistor belongs to a UART connection.

## Electrical Interpretation

A resistor in series with an external GPIO can serve different purposes. Those
purposes must not be conflated.

| Connection direction | Possible function | Placement and consequence |
| --- | --- | --- |
| MCU TX to connector | Edge damping / source termination | Place next to the MCU output. A value in the tens of ohms may reduce ringing and EMI when the trace or cable requires it. |
| MCU TX to connector | Fault-current limiting | A larger value limits current if an external device drives or shorts the line. At 3.3 V, 499 ohm bounds the ideal resistive current to about 6.6 mA. |
| Connector to MCU RX | Input-fault limiting | A resistor next to the MCU limits current into input clamp structures. It is not source termination, because the signal driver is external. |

The CH32H417 data sheet specifies 5 pF typical I/O capacitance and GPIO output
drive capability. It does not make a 499 ohm UART series resistor a general
requirement. A 499 ohm part is substantially larger than a normal edge-damping
value and creates an RC edge-rate penalty with cable and receiver capacitance.

## Design Decision for the New WEU6 Board

For the Project B-style local debug header, retain the proven direct topology:

| Signal | WEU6 pin | Connection | Default fitted series resistor |
| --- | --- | --- | --- |
| TX | `PD1`, `USART6_TX` AF8 | MCU to debug-header TX | None |
| RX, only if bidirectional debug is required | `PD0`, `USART6_RX` AF8 | debug-header RX to MCU | None |

Do not copy `R24/R25 = 499 ohm` into this UART path. Do not reserve an RX
series-resistor footprint: the external USB-UART adapter is the signal source,
so an MCU-side RX resistor cannot provide source termination. It also does not
make an incompatible 5 V adapter safe; use an adapter compatible with the
target I/O voltage.

If the final TX connection uses a long external cable and measurements show a
need for damping, that is a separate TX-only requirement. A TX test option may
then be placed next to the MCU output and initially fitted as 0 ohm.

## Sources

- https://oshwhub.com/calvaria/project_slbqmtir
- https://github.com/Calvariaa/CH32H417_Logic_Analyzer_Mini/blob/main/CH32H417_Logic_Analyzer_1v0/CH32H417_Logic_Analyzer_APP/Common/Debug/debug.c
- https://oshwhub.com/aaahaley/project_xkzwqync
- https://www.wch-ic.com/downloads/CH32H417DS0_PDF.html
