# Source design input

`CH32H417 逻辑分析仪.eprj2` is a hard link to the authoritative LCEDA Pro
project at `/Users/ivan/Sync/LCEDA-Pro/projects/CH32H417 逻辑分析仪.eprj2`.
It is provided here only so that design work and documentation have a stable,
local source-input path.

## Read-only rule

Treat this entire directory as read-only input. Agents and tools may inspect
the source project only when a task explicitly requires it. They must not
modify, save, migrate, optimize, reformat, or otherwise write the `.eprj2`
file or any adjacent source-input file without the owner's explicit approval.

Because this is a hard link, writing this copy would directly modify the
authoritative LCEDA Pro project. The project file is intentionally ignored by
Git: it is not a repository artifact and must not be committed, copied, or
redistributed from this repository.
