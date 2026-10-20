# CH32H417WEU6 Logic Analyzer

This repository is the working hardware baseline for a compact logic analyzer
built around CH32H417WEU6. The project retains the useful peripheral and
capture architecture from the researched open-source references while being
redesigned for a smaller board.

Start with [the current hardware selection](docs/SELECTION.md). The supporting
evidence is indexed in [the documentation index](docs/README.md).

The pinned firmware source baseline is documented in [firmware/README.md](firmware/README.md). It is a porting baseline only; board-specific power, pin, protocol, and build gates remain open.

Vendor PDFs and their generated full-text copies are kept locally under
`docs/datasheets/`. They are ignored by Git until their redistribution status
is reviewed; each document folder contains a tracked source note and canonical
download URL.
