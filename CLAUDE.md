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

## Design concept (shared with `slimsplaydy`)

Ultra-thin **travel** split that clamshells shut. Thesis: PG1316S are so low that when pressed they
drop below the nice!nano's USB-C receptacle, so the **USB-C is the thickness floor** — xiphos is "the
thinnest a board can be while keeping an onboard USB-C port." Enablers:

- **nice!nano mounted SMD upside-down** in a board cutout so the 3.2mm USB-C sits flush with the PCB
  bottom and the nano's back on top.
- **PCB cut away under the battery** (case holds up to a 3.0mm cell, which stays under the USB-C height).
- **Blind slot milled in the back** where the `magsafe` silk ring is — **route no traces across it** —
  holding a **steel ring** flush with the board bottom. The ring is for **mounting** to Hunter's
  existing MagSafe accessories (leg straps, desk clamps, tent stands), **not** for clasping the halves
  (they'd be ~6mm apart and facing away when closed). Use ferromagnetic steel (low-carbon / 400-series,
  not 304), sized to the MagSafe magnet array so pucks self-center. JLC can do the blind slot; it's
  pricey (more than the PCBs) but confirmed.
- **Case:** open-bottom ring, rim only as tall as a *depressed* key (adds no height). Embedded **clasp
  magnets** (matched polarity — mind the flip-over pose) + a **rabbet** edge for shear/registration
  hold the halves face-to-face. Halves **fully separate** (no hinge — Hunter likes them apart/tented).
  Closed stack ~**<9mm**. Clamping holds every key depressed, so closed it stays asleep and **wakes on
  un-clamp** (key release is the transition ZMK wakes on).

## Production (JLCPCB PCBA)

Fab + assembly at JLCPCB. BOM/LCSC: PG1316S **consigned** `C9900170245`; Molex Pico-EZmate `C505023`;
power switch `C2911519`; reset switch `C79174`. nice!nano = **DNP / hand-soldered** (not in JLC's
library). Panelize with **KiKit** (multiboard, on Hunter's machine) after routing. slimsplaydy's
test PCBA order + BOM/CPL live in `../slimsplaydy/production/` and `../slimsplaydy/Production_JLCPCBA/` —
reuse as reference.

## Hardware (inherited from slimsplaydy — confirm/adjust as xiphos diverges)

- **36-key** split ergo: a bit of **splay**, quite a bit of **stagger**. (Was 34; xiphos adds one
  outboard pinky key per hand — a new single-key `shift` zone anchored to `finger_pinky_home`,
  intended as a SHIFT key. Wired to the previously-spare MCU pin **P0**. 2026-06-03.)
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
  `power_switch_smd_side`, `reset_switch_smd_side`, `magsafe_silkscreen`, `mounting_hole_npth`.
  Change a footprint in `../ergogen-footprints` first, then sync the local copy.
- Battery connector is `battery_connector_molex_pico_ezmate_1x02` (regular Molex Pico-EZmate,
  **1.20mm** pitch). This is a **third-party** footprint vendored from `../ceoloide/ergogen-footprints`
  (CC-BY-NC-SA, infused-kim + ceoloide), NOT one of Hunter's `ergogen-footprints` - so it lives only
  in the local `footprints/` copy. (Switched 2026-06-03 from the 1.0mm Pico-EZmate **Plus**, which was
  the uncommon part.) Nets default to BAT_P / GND; no 3D model in this footprint.

## Board structure (single combined PCB — reworked 2026-06-04)

- **One ergogen pcb, `xiphos_unrouted`**, contains BOTH halves as two separate board outlines
  (`board_left` + `board_right`) in one `.kicad_pcb`. Replaces the old two-file (left/right) split
  and the *hand-merged* `slimsplaydy_both*` boards (those were assembled in KiCad, never generated).
- **Nets are namespaced `L_` / `R_`** on everything (switch nets, GND, RAW, 3V3, BAT_P, RST) so the
  two electrically-independent halves never collide on one board → clean combined netlist/BOM.
  Right-half switch/MCU nets are built from `{{zone.name}}_{{col.name}}_{{row}}` to drop the
  `mirror_` prefix (verify on regen that they come out clean, e.g. `R_finger_pinky_bottom`).
- **Deterministic refs** (left footprint groups first): `S1-S18` left keys, `S19-S36` right;
  `MCU1/CONN1/PWR1/RST1/MAG1` left, `*2` right; `MH1-30` mounting holes. The schematic must match
  these exactly (see `Schematics_and_Ergogen.md` workflow).
- **Project files:** one `xiphos.kicad_pro` (carries Hunter's design rules / via sizes from the old
  slimsplaydy_left project), one `xiphos.kicad_sch`, one `xiphos.kicad_pcb`. Workflow: regen →
  `output/pcbs/xiphos_unrouted.kicad_pcb`, copy to `xiphos.kicad_pcb`, route. (Run ergogen pointed
  at the **folder** so local `footprints/` load: `ergogen .` or `ergogen <dir> -o output`.)
- Old inherited KiCad files (routed slimsplaydy boards, schematics, `_both*` merges, backups) are
  archived under **`_old_slimsplaydy/`** for reference — not deleted.

## TODO / open items

- **Schematic:** `xiphos.kicad_sch` exists as a **machine-generated draft** — transformed from the
  archived `slimsplaydy_left.kicad_sch` into a combined sheet (reuses Hunter's `huntercooh:PG1316S` /
  `nice_nano_raw21` symbols; left nets prefixed `L_`, `power:GND` converted to `L_GND`/`R_GND` global
  labels, right half duplicated w/ fresh UUIDs + `S+18`/`MCU2`/etc + `R_` nets). Validated w/o KiCad:
  every label maps to a real PCB net, no orphans/dup-UUIDs, paren-balanced. **Finish in KiCad** (can't
  run it here): (1) add 2 shift switches `S18`->`L_shift_outer_mid`, `S36`->`R_shift_outer_mid` (copy a
  PG1316S, pin1=shift net, pins2/3=GND); (2) aux NC nets `3V3`/`P101-107` not on sheet (as in the
  original — ignore/add); (3) right half is +254mm right, rearrange to taste; (4) ERC + "Update
  Schematic from PCB" per `Schematics_and_Ergogen.md`.
- **Panelization for JLCPCB assembly:** deferred until the layout is nudged + routed (the assembly
  panel/frame + fiducials + tooling holes must wrap the *final* outline). Pull current JLC specs then.
- **Rewrite `README.md`** — still slimsplaydy's text.
- **Firmware** lives in sibling `../zmk-config-xiphos` (created 2026-06-04; cradio-derived direct-pin
  shield + 36-key keymap from sweep-pro).
- **git is live** — repo at https://github.com/jusdisgi/xiphos, HTTPS remote. Per workspace policy,
  don't run git; hand Hunter the commands. HTTPS remote form only, **never** SSH.
