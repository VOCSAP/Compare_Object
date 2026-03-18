# CLAUDE.md — Compare_Object

## Program Purpose

`Z_A_BC_COMPARE_OBJECT` is a report (6 includes + main) that compares ABAP objects
(Workbench objects and Customizing table contents) between two SAP systems via RFC.
The user selects packages, programs, function modules or customizing tables,
then the report fetches and diff-displays the data from origin and target systems.

---

## Objects Analyzed

| File | Include / Role | Lines | Category |
|------|----------------|-------|----------|
| `z_a_bc_compare_object.prog.abap` | Main report (start, INITIALIZATION, AT SELECTION-SCREEN) | 75 | A |
| `z_a_bc_compare_object_top.prog.abap` | Global types, constants, variables | 67 | A |
| `z_a_bc_compare_object_sel.prog.abap` | SELECTION-SCREEN definition | 97 | A |
| `z_a_bc_compare_object_f01.prog.abap` | Module wrappers (PBO/PAI screens 2000/3000) | 81 | A |
| `z_a_bc_compare_object_c01.prog.abap` | Local class `lcl_main` — DEFINITION | 576 | B |
| `z_a_bc_compare_object_c02.prog.abap` | Local class `lcl_main` — IMPLEMENTATION + `lcl_customizing_handle_events` | 3992 | C |

Total: **4888 lines** across 6 files.

---

## Architecture

- **One local class** `lcl_main` holds all logic (definition in `_c01`, implementation in `_c02`).
- **RFC-based comparison**: reads TADIR, TDEVC, TFDIR from remote systems via `RFC_READ_TABLE`.
- **ALV display**: results shown using `CL_SALV_TABLE` on two side-by-side custom containers (screens 2000/3000).
- **Commented parallelisation include** (`z_a_bc_compare_object_cta`) — feature not activated.
- **Commented archive call** (`zcl_archive=>archive_add`) — archive not implemented.

---

## Audit Scope

- Audit date: 2026-03-18
- Target conventions: SAP standard ABAP 7.40 / ECC6
- Source format: abapgit ZIP (`ZCOMPARE_OBJECT.zip`)
- All 6 `.abap` files fully analyzed
- See `TODO.md` for all findings — see `MEMORY.md` for session log
