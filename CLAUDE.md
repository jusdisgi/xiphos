# CLAUDE.md — xiphos

Notes for future sessions. Workspace-wide conventions live in `../CLAUDE.md`; this file holds only
what's specific to **xiphos**. Keep it thin. (Name is lowercase `xiphos` everywhere — repo,
folder, and filenames.)

## What this is

- **xiphos** — a new split ergonomic keyboard, **forked from `../slimsplaydy`** (copied
  2026-06-03). The driving change is a **key-count / layout evolution** of the slimsplaydy base.
- Lineage note: slimsplaydy itself *began* as the PG1316S version of splave-ferris, but has
  evolved well past it and was recently restructured (Choc variant removed, collapsed to a single
  version with `config.yaml` at the repo root — **no variant subfolders**). xiphos starts from that
  current slimsplaydy, not from splave-ferris.

## Hardware (inherited from slimsplaydy — confirm/adjust as xiphos diverges)

- 34-key split ergo: a bit of **splay**, quite a bit of **stagger**.
- Switches: **Kailh PG1316S** ultra-low-profile.
- Controller: **nice!nano** (not XIAO), mounted **SMD upside-down** with a board cutout so it sits
  ~1.6 mm above the PCB. Everything on the board is **SMD, front side only** — hot-plate buildable,
  no soldering iron intended.
- Slimness features: PCB cut away under the battery; case includes a battery holder (up to 3.0 mm
  cell) and magnet pockets so the halves clasp shut (~8 mm closed). Case STEP files in `Case_Models/`.

## Layout source of truth

- Edit the ergogen **`config.yaml`** (at the repo root) to change the layout, then regenerate and
  route in KiCad. The key-count/layout change for xiphos happens **here first**.
- Local `footprints/` mirror `../ergogen-footprints`: `switch_pg1316s`, `mcu_nice_nano_smd`,
  `conn_molex_pico_ezmate_plus_1x02`, `power_switch_smd_side`, `reset_switch_smd_side`,
  `magsafe_silkscreen`, `mounting_hole_npth`. Change a footprint in `../ergogen-footprints` first,
  then sync the local copy.

## What was NOT copied

The fork is lean: `production/`, `output/`, `Production_JLCPCBA/`, `*-backups/`, `*.kicad_prl`,
autosave/lock files were dropped (all regenerable). The routed KiCad boards, schematics, config,
footprints, README, `.gitignore`, and `Case_Models/` came across.

## TODO / open items

- **Rename inherited files to `xiphos_*` (lowercase).** The KiCad project/board/schematic files
  still carry the `slimsplaydy_*` names. Rename to `xiphos_*` once the new layout is settled, and
  update the internal KiCad cross-references (`.kicad_pro` → `.kicad_pcb`/`.kicad_sch`) when you do.
- **Rewrite `README.md`** — it's still slimsplaydy's text.
- **git is Hunter's job.** No git history here yet. Don't run git — hand Hunter the commands. Setup:
  `git init -b main`, then add the `jusdisgi/xiphos` remote.
- Decide final key count / thumb cluster, then spin up a sibling `zmk-config-xiphos` for firmware
  (model the keymap on `../zmk-config-sweep-pro`).
