# RoyalBlue54L Feather: changes from upstream 9177387

## Modification notice (CERN-OHL-P v2)

This is Modified Source of the RoyalBlue54L Feather by Helmut Lord (Lord's Boards),
github.com/LordsBoards/RoyalBlue54L-Feather-Hardware, licensed under CERN-OHL-P v2.
It was modified on 25 September 2026, starting from upstream commit
91773871badad9a8edb488c15e7f8817707130b8 ("Fixed DRC errors", 25 Feb 2025). The work was
done by an AI hardware agent (Claude, by Anthropic) using the Eloom CLI for Matrice
Technologies (Eloom). The changes are listed below. The original `LICENSE` and notices are
kept unchanged, and the modified design stays under CERN-OHL-P v2.

**The author has not reviewed these changes.** Passing KiCad's electrical and design rule
checks does not prove that the board works, can be manufactured or is safe. No board was
built or tested, and no circuit behaviour was simulated.

## Result

| Stage | ERC | DRC |
|---|---|---|
| KiCad's bundled copy (17 Feb 2025, the board agent's runs) | 43: 5 errors, 38 warnings | 283: 237 errors, 46 warnings |
| Upstream 9177387 (25 Feb 2025) | 43: 5 errors, 38 warnings | 62: 16 errors, 46 warnings |
| This revision | **0 errors, 0 warnings** | **0 errors, 0 warnings**, 0 unconnected, schematic parity ran with 0 items |

Final runs: KiCad 10.0.6 through `eloom kicad check`. ERC run b2970791-3e76-4eb3-b13f-1b60e89f4dbc,
DRC run 6c997f6f-55f9-4a23-949f-2f0b73162ddc (board sha256
7ea0766ba156b926a9c97283d10266e7b4e5bb50088f3287ab821ea7a6f0e85c). The run folders are in `runs/`:
the two checks with their results, the BOM export with its CSV, and the manifests of the
Gerber and drill exports, which name each exported file with its size and SHA-256. The
exported Gerber and drill files themselves are not included.

Netlist, 9177387 → now: the same 95 nets with the same nodes. Only component footprint
names changed: 41 now point to the archived copies (item 5b), TP5–TP12 use D1.0mm pads
(item 4a), and J2 uses the mirrored header (item 4d).

## Starting point

- Upstream was cloned and 9177387 checked out. Git submodules (nordic-lib-kicad,
  marbastlib, hlord2000-kicad-libraries) were not fetched; the design's own embedded copies
  were archived instead (item 5).
- `fixed/`, the working copy this revision was made in, was seeded with that commit's
  project files: root and four sheet schematics, board, project file, both library tables,
  `lib/footprints`, `lib/symbols`, `LICENSE` and `README.md`. 3D models, the jobset and the
  NFC antenna sub-project were not copied.
- Checks: KiCad 10.0.6 through `eloom kicad check`, with KiCad's default global library
  tables in a scratch `KICAD_CONFIG_HOME`.
- Baseline (17:42 CEST), ERC 43 items: 5 power_pin_not_driven errors, 4 endpoint_off_grid,
  28 lib_symbol_mismatch, 4 lib_symbol_issues, 2 footprint_link_issues. DRC 62 items:
  5 hole_clearance, 11 courtyards_overlap, 40 lib_footprint_mismatch, 2 lib_footprint_issues,
  4 silk_overlap; 0 unconnected, 0 parity.

## What was not changed

No rule severity, exclusion, design rule or netclass was changed:
`RoyalBlue54L-Feather.kicad_pro` is byte-identical to 9177387. It already contained, and
still contains, the author's settings:

- ERC checks set to ignore: single_global_label, four_way_junction, pin_to_pin,
  simulation_model_issue, footprint_filter.
- DRC checks set to ignore: missing_courtyard, footprint_filters_mismatch,
  footprint_type_mismatch, npth_inside_courtyard, pth_inside_courtyard, text_thickness.
  KiCad also lists track_not_centered_on_via and tuning_profile_track_geometries as ignored
  by default.
- 30 DRC exclusions, all copper_edge_clearance: 28 for the castellated half-hole pads of
  headers J1 (16) and J2 (12), which sit on the board edge by design, and 2 for pads of the
  NFC FPC connector J6 at the bottom edge.

These are the author's design decisions and are reported here as they were. They are why
castellations and the other ignored checks do not appear in either count.

## Method

- Every file change was written with `eloom kicad write`, passing the SHA-256 that
  `eloom kicad read` had just observed, and only when that hash equalled the hash of the
  file the edit was computed from (`tools/ewrite.py`). Edits are exact text replacements in
  KiCad's own S-expressions (KiCad 9 file format, as upstream), so every line not described
  here is byte-identical to 9177387.
- Where copper moved, all zones (GND, VDD, VSYS and +BATT pours, and the teardrops) were
  refilled by KiCad 10.0.6's own zone filler (pcbnew Python) in a scratch copy, with the
  project's rules loaded; only the resulting `filled_polygon` data was copied back.
  Refilling the untouched 9177387 board this way reproduced the same 62 DRC items. KiCad has
  no scriptable teardrop generator, so a teardrop at a via that moved had its outline
  translated with the via, then its fill recomputed by KiCad.
- Each group was first tried on a scratch copy with `kicad-cli`, then written to `fixed/`
  and re-checked with `eloom kicad check` (ERC, and DRC with parity).

## Changes

### 1. power_pin_not_driven (5 ERC errors): PWR_FLAG on five externally fed nets

Each net below is fed only through passive pins, so ERC cannot see a source. KiCad 10's
`power:PWR_FLAG` (from `eloom kicad inspect power:PWR_FLAG`) was embedded in each sheet's
`lib_symbols` in the files' KiCad 9 dialect; ERC confirms it matches the library. Flags sit
on 1.27 mm grid points of existing wires. Where a flag sits mid-wire, the wire was split
there and a junction added.

| Flag | Sheet | Net | At (mm) | Why the net is externally fed |
|---|---|---|---|---|
| #FLG01 | Debugger | /Debugger/USB.VBUS | (106.68, 24.13), on the J3 VBUS wire | USB-C receptacle J3's VBUS pins (passive) |
| #FLG02 | Debugger | /Debugger/nRF52_VDD | (203.2, 24.13), on the C10–C15 decoupling wire | U5 (nRF52833) takes VDDH from VBUS, so its REG0 regulator drives the VDD pins, which the symbol types as unspecified |
| #FLG03 | nPM1300 | VDD | (231.14, 93.98), rotated 180°, between JP1 and the VDD symbol | BUCK2 output (U2 VOUT2, L1, C8) reaches VDD through the bridged solder jumper JP1 (passive) |
| #FLG04 | Connectors | +BATT | (186.69, 99.06), rotated 180°, between J4 pin 1 and +BATT | battery connector J4 (passive) |
| #FLG05 | Connectors | GND | (181.61, 106.68), rotated 180°, at the corner of the J4 pin 2 / MP ground wire | battery return on J4 |

### 2. endpoint_off_grid (4 ERC warnings): TP7 onto the 1.27 mm grid

KiCad files these under the root sheet, but the items are on the **Debugger** sheet: test
point TP7 (/Debugger/D+) at x = 95.201 mm, its wire to the D+ line, and the junction there.
TP7, the two wire ends and the junction moved +0.049 mm in x, to 95.25 mm (the column of
TP5 and TP8). TP7's hidden fields moved by the same amount.

### 3. hole_clearance (5 DRC errors): four short reroutes

`RoyalBlue54L-Feather.kicad_pcb`. The rule is the board setup's 0.254 mm hole clearance
(unchanged). Widths, layers and nets are unchanged; every run keeps 45° geometry and ends
exactly where it did. The clearances below are computed from the geometry, and DRC confirms
all five pass.

| Net, layer | Change | Clearance to the hole |
|---|---|---|
| /Debugger/USB.VBUS, B.Cu, 0.3 mm (TP5 to the J3 VBUS via at 125.405, 102.53) | was (122.3, 101.38) → (123.67, 102.75) → (125.185, 102.75) → via; now (122.3, 101.38) → (122.3, 101.50) (a stub inside TP5's pad) → (123.65, 102.85) → (125.085, 102.85) → via. The run moves 0.085–0.1 mm away from J3's NPTH peg hole (123.96, 102.09; Ø0.65) | 0.197 and 0.185 → 0.282 and 0.285 mm |
| /Debugger/USB.CC2, In4.Cu, 0.15 mm | horizontal run y 101.44 → 101.41 (x 141.44–144.96); both 45° ends slide along their own lines. The allowed window is y 101.39–101.42 (0.15 mm to USB.CC1 at y 101.09 above, 0.254 mm to the via below) | to the VSYS via (142.98, 101.9): 0.235 → 0.265 mm; to USB.CC1: 0.20 → 0.17 mm (rule 0.15) |
| Net-(J8-SWCLK), B.Cu, 0.15 mm | the 45° run into via (133.84, 105.4) is offset 0.05 mm: corner (133.84, 104.49) → (133.84, 104.54); horizontal end 134.61 → 134.66 | to the SWDIO via (133.39, 104.33): 0.2526 → 0.272 mm |
| Net-(J8-~{RESET}), In4.Cu, 0.15 mm | y 104.95 → 104.90 between x 132.90 and 136.15, with 0.05 mm 45° jogs at both ends (3 segments added). This is midway between the SWCLK via below and the nRF52_VDD and GND vias at y 104.4 above. The 0.48 mm leaving via (136.68, 104.95) stays horizontal, so its teardrop still fits | to the SWCLK via (133.84, 105.4): 0.225 → 0.275 mm; to the vias above: 0.325 → 0.275 mm |

### 4. courtyards_overlap (11 DRC errors)

Courtyard gaps as KiCad computes them (pcbnew courtyard polygons):

| Pair | 9177387 | Now | How |
|---|---|---|---|
| TP5/TP8, TP8/TP7, TP7/TP6, TP9/TP10, TP9/TP11, TP10/TP12, TP11/TP12 (7 pairs) | overlap | 0.41 mm | smaller pads, same pitch (a) |
| H1/J5 | overlap | 0.03 mm | J5 moved (b) |
| J3/J5 | overlap | 0.22 mm | courtyards drawn to the outlines (c) |
| J4/J2, H2/J2 | overlap | on different sides | J2 back on B.Cu (d) |

**(a) TP5–TP12** (the Debugger's USB and SWD test pads, on B.Cu, 2.4 mm pitch).
`TestPoint:TestPoint_Pad_D1.5mm` → `TestPoint:TestPoint_Pad_D1.0mm`, KiCad's own library
footprint, unmodified (DRC finds no library mismatch). The swap was made in the Debugger
sheet's Footprint fields and in the board. Positions and pitch are unchanged, and tracks
still end at pad centres. Courtyards go from r 1.25 to r 1.0 mm. A 1.0 mm pad at 2.4 mm
spacing is still a normal spring-probe target, but a fixture made for the old 1.5 mm pads
needs its drawing checked.

**(b) J5** (Qwiic, JST SH SM04B-SRSS-TB, locked, top edge) moved **+0.32 mm in x**
(127.99 → 128.31 mm) to clear H1's courtyard (the M2.5 hole's courtyard has r 2.75 mm). It
moved along the edge, so its distance to the edge is unchanged. Its four in-pad vias (GND,
VDD, SDA, SCL at y 98.633) moved with it. The In2.Cu SDA and SCL runs that leave those vias
at 45° were translated by the same 0.32 mm, their horizontal runs are 0.32 mm shorter, and
the two teardrop outlines at those vias were translated with them. The move is bounded by
H1 on the left and J4 on the right (J5/J4 is now 0.08 mm).

**(c) J3/J5.** Both courtyards were bounding boxes that overlapped at empty corners (J3's
rear top corner, J5's rear left corner). The nearest real features, J3's rear shell tab S1
and J5's pad 4, are 1.23 mm apart after the move. Both courtyards were drawn to the parts'
outlines:
- J5 now has **KiCad 10's own courtyard for this exact part**. The current Connector_JST
  library draws JST_SH_SM04B-SRSS-TB's courtyard around the body, pads and mounting pads
  (16 segments) instead of a box. J5's pads, silkscreen and fab graphics were compared with
  KiCad 10's footprint and are identical, and DRC now finds J5 matching that library entry.
- J3 (HRO TYPE-C-31-M-12) keeps its box except the two empty rear corners. Each is cut at
  45°, tangent to a 0.5 mm margin around the rear shell tab's rounded end, and stays
  0.5 mm clear of pads A1/B12 and of the body. In J3's local frame, the corner
  (∓5.32, −5.27) becomes (∓5.32, −4.10) → (∓4.74, −4.68) → (∓4.05, −4.68) → (∓4.05, −5.27).
  Both corners are cut, for symmetry. Every J3 pad, the body and the shell tabs keep at
  least 0.5 mm margin (the KLC value for connectors). The part is archived under a derived
  name (item 5b).

**(d) H2/J2** (M2.5 hole against the 12-pin castellated header), decided with the project
lead. In the Feather/Thing Plus geometry the hole is 3.81 mm from the last pin. The header
body clears a 5 mm screw head by 0.04 mm, and H2's courtyard (r 2.75 mm) reaches 0.21 mm
into the header body. No courtyard could honestly be trimmed, and moving H2 or J2 breaks
the form factor.
- History: up to commit a77a3d6 both headers, J1 and J2, were on B.Cu. In 0385544
  ("Updated to include castellations"), J2 moved to F.Cu while J1 stayed on B.Cu. The
  castellated footprint has its edge pads at local +x, so the top row cannot sit on the
  back with its castellations on the top edge and pin 1 at the left. The move to F.Cu reads
  as forced by the footprint rather than chosen.
- Change: a derived footprint
  `RoyalBlue54L-Feather-Connector_PinHeader_2.54mm:PinHeader_1x12_P2.54mm_Vertical_Mirrored`,
  a new file in the author's library. It is the author's footprint mirrored in x: edge pads
  at local −x, drill offsets mirrored, and the name, description and Value say so. J2 uses
  it on **B.Cu at −90°**, exactly as J1 is placed, and J2's Footprint field on the Connectors
  sheet was changed to match. Reference, value, uuid, pad uuids and the pin-to-net map are
  unchanged.
- Proof that nothing physical moved, from KiCad's own geometry. For each of the 8 copper
  layers, both masks and both pastes, J2's pad shapes before and after have zero XOR area.
  The 24 holes are identical in position, size and net. The drill file is identical apart
  from its date line. In the Gerbers, every draw and flash is identical once each aperture
  is resolved to its definition. The files differ as text only because KiCad writes J2's
  octagonal pads as apertures rotated 270° instead of 90° (the octagon is symmetric under
  180°) and renumbers apertures. The courtyard and fab Gerbers change as intended: J2's
  outline moves from the F layers to the B layers. These comparisons were made in the
  working copy, and their outputs are not included here; they can be repeated by exporting
  both boards with KiCad and comparing them. After the change DRC shows no parity item and
  no library mismatch for J2, and the author's 28 castellation exclusions still match.
- Caveat: KiCad's mounting-hole footprint has only a top-side courtyard. Bottom-side
  hardware at H2 (a nut, or a 5 mm standoff) now has the same 0.04 mm margin to a fitted
  header body as the author's design already had at J1/H4. A hex standoff wider than 5 mm
  across the corners will touch a fitted header at H2, as it already would at H4.

**J4** (battery JST PH) was moved 0.06 mm in an intermediate step to clear J2's front
courtyard. After (d) that was no longer needed, so J4 and its in-pad +BATT via are back
exactly at their 9177387 positions.

### 5a. Symbol library warnings (34 ERC warnings): archive the embedded symbols

New project library `lib/symbols/RoyalBlue54L-Feather.kicad_sym`, registered in
`sym-lib-table` as `RoyalBlue54L-Feather` (a `${KIPRJMOD}` path). It holds the design's own
embedded definitions, copied verbatim from the sheets' `lib_symbols` (renamed from
`Library:Name` to `Name` and re-indented; each copy was identical across sheets). The
matching `lib_symbols` entries and each symbol's `lib_id` now read `RoyalBlue54L-Feather:<Name>`.
No symbol graphics, pins or fields changed.

| Symbol (was) | Instances | Why it warned |
|---|---|---|
| Device:C_Small | 25 (all capacitors) | KiCad 10's copy differs: plate stroke 0.3048 vs 0.3302 mm, Datasheet `""` vs `"~"`, Description field position |
| Device:LED_Small | D1 | KiCad 10 renamed the `Sim.Pin` field to `Sim.Pins` |
| Jumper:SolderJumper_2_Bridged | JP1 | KiCad 10 redrew the graphics and sets `exclude_from_sim no` |
| Connector:USB_C_Receptacle_USB2.0_16P | J3 | KiCad 10 renumbered the shield pin from `S1` to `SH`. Updating from the library would have disconnected J3's shield pads (numbered S1) from GND, which is why the embedded copy is kept |
| hlord2000-Device:Crystal_GND24_Small_EZRoute | Y2 | library is an unfetched git submodule |
| nordic-lib-kicad-nrf52:nRF52833-QDXX | U5 | unfetched submodule |
| nordic-lib-kicad-npm:nPM1300-QEXX | U2 | unfetched submodule |
| nordic-lib-kicad-nrf54-modules:BM15x | U1 | unfetched submodule |

The four submodule entries in `sym-lib-table` were left unchanged. Nothing references them
now, and they become valid again if the submodules are fetched.

### 5b. Footprint library warnings (40 + 2 DRC, 2 ERC): archive the embedded footprints

New project library `lib/footprints/RoyalBlue54L-Feather.pretty`, registered in
`fp-lib-table` as `RoyalBlue54L-Feather`. Each entry is one placed instance from the board
turned back into library form, preferring an unrotated instance. Placement angles were
removed from pads and texts, footprint rule areas were mapped back to the footprint frame,
and nets, schematic links and locks were dropped. Geometry and item uuids are kept. The
files are KiCad 9 format, as upstream. The board footprints and the placed symbols'
Footprint fields now read `RoyalBlue54L-Feather:<Name>`. The Footprint defaults inside the
archived symbol definitions stay as the author had them. No placed footprint changed
geometry: DRC's own library comparison checks every instance against its archived entry and
finds no mismatch, and parity passes.

| Footprint (was) | Instances | Source instance |
|---|---|---|
| Capacitor_SMD:C_0402_1005Metric | 18 | C13 |
| Capacitor_SMD:C_0603_1608Metric | 7 | C11 |
| Resistor_SMD:R_0402_1005Metric | 6 | R1 |
| Inductor_SMD:L_0603_1608Metric | L1 | L1 |
| LED_SMD:LED_0603_1608Metric | D1 | D1 |
| Crystal:Crystal_SMD_2016-4Pin_2.0x1.6mm | Y2 | Y2 |
| Connector_JST:JST_PH_S2B-PH-SM4-TB_1x02-1MP_P2.00mm_Horizontal | J4 | J4 |
| Package_DFN_QFN:QFN-32-1EP_5x5mm_P0.5mm_EP3.6x3.6mm_ThermalVias | U2 | U2 |
| Package_DFN_QFN:QFN-40-1EP_5x5mm_P0.4mm_EP3.6x3.6mm_ThermalVias | U5 | U5 |
| Package_SON:Winbond_USON-8-1EP_3x2mm_P0.5mm_EP0.2x1.6mm | U4 | U4 |
| Package_TO_SOT_SMD:SOT-553 | U3 | U3 |
| nordic-lib-kicad-nrf54-modules:BM15x-LGA-45_15.8x10mm (unfetched submodule) | U1 | U1 |
| marbastlib-various:USB_C_Receptacle_HRO_TYPE-C-31-M-12 (unfetched submodule) → `USB_C_Receptacle_HRO_TYPE-C-31-M-12_ChamferedCourtyard` (derived name, item 4c) | J3 | J3 |

KiCad's own libraries were not used to update any part. The footprint entries for the
unfetched submodules (`marbastlib-various`, `nordic-lib-kicad-nrf54-modules`) were left in
`fp-lib-table` unchanged.

### 6. silk_overlap (4 DRC warnings, plus 2 created by the J5 move)

| Item | Change | Why |
|---|---|---|
| Board text "+" (battery polarity mark beside J4) | 0.1 mm down, (139.71, 102.855) → (139.71, 102.955), with its render cache | it was 0.046 mm from J4's outline. The board's texts use the Ubuntu Sans font, which is not installed on this machine; KiCad substitutes Verdana Bold and draws a larger "+". The new position clears J4 with either font (0.14 mm with the substitute) |
| LOGO1 (Lord's Boards logo, silkscreen only) | 0.95 mm down | after J5 moved, J5's pin-1 mark touched the logo |
| R5 (0402, 10 kΩ, I2C pull-up) | 0.03 mm down, with its two in-pad vias. The NPM1300.SCL runs leaving pad 2 on F.Cu and In4.Cu stay horizontal, their 45° corners slide along the unchanged diagonals, and the three teardrops at pad 2 moved with it | R5's and R7's outlines were 0.06 mm apart (rule 0.0762); now 0.09 mm |
| C9 (0402, 4.7 µF) | 0.03 mm left, with its two in-pad vias (zone connections only, no tracks) | it was 0.065 mm from U1's outline; now 0.095 mm |

### 7. Anything else

Nothing new appeared. After every group, DRC reported 0 unconnected items and no parity
items. One environmental fact matters for anyone repeating the checks: the silkscreen font
(Ubuntu Sans) is not installed here, so KiCad substitutes Verdana Bold and says so. Silk
clearances were made to hold with both fonts where it mattered (item 6).

## Files

Changed: `RoyalBlue54L-Feather.kicad_pcb`, `sch/Connectors.kicad_sch`, `sch/Debugger.kicad_sch`,
`sch/nPM1300.kicad_sch`, `sch/nRF54L15.kicad_sch`, `sym-lib-table`, `fp-lib-table`.
Unchanged: `RoyalBlue54L-Feather.kicad_sch` (root), `RoyalBlue54L-Feather.kicad_pro`,
`LICENSE`, `README.md`, and the author's existing library files.
Added: `lib/symbols/RoyalBlue54L-Feather.kicad_sym`, `lib/footprints/RoyalBlue54L-Feather.pretty/`
(13 footprints),
`lib/footprints/RoyalBlue54L-Feather-Connector_PinHeader_2.54mm.pretty/PinHeader_1x12_P2.54mm_Vertical_Mirrored.kicad_mod`,
this notice (`CHANGES.md`) and the final runs (`runs/`).
