# UART Debug Mapping in the Reference Projects

## Scope and Evidence Standard

This note records only connections that are directly visible in an accessible
reference schematic or explicit in the corresponding firmware. It does not
infer a missing RX connection from the existence of a UART peripheral.

## Project A: `aaahaley/project_xkzwqync`

The public path resolves to project UUID
`5164038bbe434efda62fb927214a7035`. Its public project-detail API returns
`104001`, but the opened read-only editor exposes `P1.Schematic1`. The relevant
CH32H417QEU6 connections are:

| Original MCU pin | QEU6 package pin | Schematic connection | UART alternate function |
| --- | --- | --- | --- |
| `PA9/OTG_VBUS` | 88 | Net `$1N585` to R25 pin 1; R25 is 499 ohm | `USART1_TX`, AF7 |
| `PA10/OTG_ID` | 89 | Net `$1N584` to R24 pin 1; R24 is the paired series resistor | `USART1_RX`, AF7 |

The schematic uses anonymous net IDs rather than visible `TX` and `RX` labels
for this pair. The data sheet establishes the UART1 TX/RX functions; Project B
explicitly migrates only the PA9 side, as its firmware is output-only.

Sources:

- https://oshwhub.com/aaahaley/project_xkzwqync
- https://oshwhub.com/api/common/projects/getUuidByPath?path=aaahaley%2Fproject_xkzwqync
- https://oshwhub.com/api/project/5164038bbe434efda62fb927214a7035
- https://www.wch-ic.com/downloads/CH32H417DS0_PDF.html

## Project B: `calvaria/project_slbqmtir`

The accessible `Schematic-mini0v1` sheet shows one debug-UART net only:

| Signal | Reference connection | WEU6 package pin | Firmware function |
| --- | --- | --- | --- |
| `TX1` | U1 `PD1` to H2 pin 2; H2 pin 1 is GND | 53 | `USART6_TX`, AF8 |
| RX | No RX net, connector pin, or MCU connection is present | Not applicable | Not enabled |

The project marks the package migration as `PA9 -> PD1`. Its firmware selects
`DEBUG_UART6`, configures `GPIOD` pin 1 with AF8, and initializes the port with
`USART_Mode_Tx`, not receive or full-duplex mode. This is a single-direction
debug-print output, not a two-wire serial interface with a missing mapping.

The complete pin correspondence for a deliberate two-wire debug-UART port is:

| Function | Project A QEU6 | New WEU6 design |
| --- | --- | --- |
| TX | PA9, pin 88, `USART1_TX` AF7 | PD1, pin 53, `USART6_TX` AF8 |
| RX | PA10, pin 89, `USART1_RX` AF7 | PD0, pin 52, `USART6_RX` AF8 |

Only the TX row exists in Project B. Adding the RX row is a new design choice:
route the new RX net to PD0 and update firmware to use
`USART_Mode_Rx | USART_Mode_Tx`.

Sources:

- https://oshwhub.com/calvaria/project_slbqmtir
- https://oshwhub.com/api/project/4ad0e69bb4e74891adba6df8ea3e9938/proImages
- https://github.com/Calvariaa/CH32H417_Logic_Analyzer_Mini/blob/main/CH32H417_Logic_Analyzer_1v0/CH32H417_Logic_Analyzer_APP/Common/Debug/debug.h
- https://github.com/Calvariaa/CH32H417_Logic_Analyzer_Mini/blob/main/CH32H417_Logic_Analyzer_1v0/CH32H417_Logic_Analyzer_APP/Common/Debug/debug.c
- https://www.wch-ic.com/downloads/CH32H417DS0_PDF.html
