---
name: patent-3d-figure-generator
description: Generate CNIPA-compliant and GB/T-compliant patent drawings (native Python/OCP 3D-to-2D HLR projection, 300 DPI JPG, vector SVG, layered 2D AutoCAD R12 DXF, 5-section DOCX auto-backfill, isometric & modular-exploded views, and interactive Three.js Web CAD viewers) and parametric CadQuery multi-part STEP assemblies strictly aligned with patent claims and specifications. Enforces Gate 0 full-cycle kinematic/force-chain verification, Gate 1-6 zero-interference 3D modeling with Chinese STEP feature trees, on-demand minimal figure-set planning (never hardcoding a fixed figure count), real 3D spring HLR projection with geometry caching, foreground-occluder hatch clipping, automated severed-edge wavy-breakline closure, minimalist centerline rules, and zero-crossing smooth micro-arc callouts. Use when the user asks to create patent drawings (说明书附图), export patent figures from CadQuery/STEP via OCP, generate 2D DXF or 3D STEP models from a patent disclosure or specification, or audit mechanical patent figures.
---
# Patent 3D & 2D Figure Generator

Generate CNIPA-compliant and GB/T-compliant patent drawings (`说明书附图`) and parametric 3D CAD assemblies with 100% structural, dimensional, kinematic, force-chain, and reference-numeral consistency with the patent claims and specification.

## Route Selection (Clean Primary vs. Fallback Separation)

1.  **Primary Route — 3D CadQuery/STEP -> Native Python/OCP 3D-to-2D Patent Figure Pipeline**:
    - Use whenever a 3D model is requested or available (preferred for all mechanical/electromechanical patents).
    - **Stage A (3D Model — New or Modified)**: Pass **Gate 0** (full-cycle kinematics & force-chain) + **Gates 1–6** (parametric `cq.Assembly` -> zero-interference STEP with ISO 10303-21 Chinese feature tree). *Fast-Path for Frozen Models*: When the user supplies an already-approved `.py` or `.step` solely to export figures, skip rebuilding — run a read-only sanity check (flagging any fatal flaw if detected) and proceed directly to Stage B.
    - **Stage B (On-Demand Figure Planning & OCP Export)**: First plan the **minimal, clearest view combination tailored to the specific patent's core inventive points and key operating states** (never hardcoding a fixed 6-figure template — e.g., 3–4 figures for simple structures, 5–7+ for multi-state/complex assemblies, always 1 view per figure). Then reuse the verified `cq.Assembly` (prefer importing the `.py` `build_*_assembly()` directly so numbered Chinese part names stay intact; if only a `.step` file is provided, read it via `OCP.STEPCAFControl.STEPCAFControl_Reader` XCAF tree — never `cq.importers.importStep()`, which flattens part names) via `OCP.HLRBRep.HLRBRep_Algo` + `BRepAlgoAPI_Cut` to output CNIPA CPC-ready **300 DPI monochrome JPGs** (`≤ 16.0 × 24.5 cm`), **vector SVGs**, pure-Python **5-layer AutoCAD R12 ASCII `.dxf`** (`01_OUTLINE`, `02_HATCH`, `03_CENTER`, `04_LEADER`, `05_NUMERAL`), smooth **arrowless, dotless micro-arc leaders**, and auto-backfilled **5-section patent DOCX** (syncing seamlessly with `cnipa-patent-drafting-pro` for PY-to-text bidirectional cross-check and 16-item red-line verification).
2.  **Fallback Route — Standalone 2D CAD Drafting (No 3D Model)**:
    - Use ONLY when drafting 2D figures directly without a 3D model.
    - Output standalone AutoCAD R12 ASCII `.dxf` using the 9-layer structure (`01_OUTLINE`, `02_DETAIL`, `03_CENTER`, `04_REF_PILE` `PHANTOM`, `05_HIDDEN` `DASHED`, `06_HATCH_45`, `07_HATCH_135`, `08_LEADER`, `09_NUMERAL`); in this 2D-only mode, place `SOLID` black dots only inside surface areas (never on edges), and render coaxial helical springs around unhatched rods as wire cross-section circles on both sides.
3.  **On-Demand Interactive Preview**: Single-file Three.js (`r128`) Web CAD viewer with `图 1`–`图 N` camera switching, CNIPA monochrome wireframe mode, SVG callouts, `.png` snapshot, and `.stl` export.

## Mandatory 3D Modeling Quality Gates (Gate 0 + Gates 1–6)

1.  **Gate 0 — Full-Cycle Kinematics, Clearance & Force-Chain Verification (前置运动与力流闭环门禁)**:
    - BEFORE writing 3D coordinates, walk through the **entire multi-step operating cycle** (loading -> lifting/transport -> unloading -> empty return/reset) and verify quantitatively:
      - **Working-Surface Clearance**: Auxiliary mechanisms (claws, guards, strippers) must clear the primary working face and external boundaries (e.g., truck bed/floor, material pile) in all active states.
      - **Empty-Return / Reset Timing**: Verify no unintended re-triggering or timing conflict occurs during idle/return strokes (e.g., claws re-closing on an unloaded pile before lift-off).
      - **Force-Chain & Self-Locking Balance**: Trace every load path under peak load (confirm rigid fastening where upward/separating loads act, self-locking cam/ratchet dead-center alignment, and reset spring force > total moving mass + friction).
      - **Field Foolproofing**: Include visual phase/state indicators, manual reset access for bistable mechanisms, and overload compliance where jamming can occur.
2.  **Gate 1 — Single-Source Datum, Multi-Plane Decoupling & XZ Normal Safety**:
    - Define ALL key coordinates, clearances, strokes, and wall thicknesses in one top-level parameter dictionary `P`; derive mating positions algebraically from `P`.
    - Separate multi-plane transmission sub-assemblies along explicit depth planes so full-section views remain continuous.
    - **Ban Single-Sided `Workplane("XZ")` Extrusion**: `Workplane("XZ").extrude(+L)` extrudes along **`-Y`** (`X × Z = -Y`). For any feature spanning `[y0, y1]` on `XZ`, ALWAYS use `.extrude((y1 - y0)/2.0, both=True).translate((0, (y0 + y1)/2.0, 0))`.
3.  **Gate 2 — True Mechanical Joint Geometry & Anti-Pitfall Rules**:
    - Union closed circular hinge bosses (`R_boss >= R_bore + 3.0 mm`) before cutting real bores (`R_bore = R_pin + 0.5 mm`).
    - Put coaxial torsion springs/rollers in disjoint `Y` pockets; auto-inset every `make_helical_spring()` at both ends by `wire_r + 0.35 mm`.
    - Saddle-trim rectangular seats on cylindrical shells with `circle(R_shell + 0.1)`; build real hollow sockets/clevises/stepped ledges with `0.00 mm³` overlap; solve tangent curved contacts analytically (`+0.15 mm` micro-gap).
4.  **Gate 3 — Environment-Aware Pre-Delivery Verification**:
    - Auto-detect whether `cadquery`/OCP is installed in the current environment (local or cloud). If available, execute real OCCT Gate 5 directly; ONLY if `cadquery`/OCP is unavailable, run `Shapely` 2D exact-polygon collision assertions (`intersection.area == 0.000 mm²`) and Matplotlib visual slice checks across all states.
5.  **Gate 4 — Visual Slice Self-Audit ("先照镜子再交稿")**:
    - Visually inspect dense mechanism sections (tooth engagement, bore wall thickness, spring pockets, extreme-state clearances) before finalizing.
6.  **Gate 5 — Embedded 3D OCCT Audit (`run_3d_audit`)**:
    - Embed zero-dependency `run_3d_audit(assy)` checking: `isValid()` + `Volume() > 0`, single-part continuity (`len(Solids()) == 1` outside explicit whitelist), pairwise AABB + OCCT `intersect().Volume() > 1.0 mm³` interference scan, `2.5 mm` anti-floating connectivity, and `*_audit.json` export across **every operating state**.
7.  **Gate 6 — Standardized Chinese STEP Feature Tree (ISO 10303-21)**:
    - Use numbered canonical Simplified Chinese part names and embed `encode_step_chinese_iso10303(filepath)` WITH `_repair_cq_double_utf8()` pre-repair before `\X2\XXXX\X0\` escaping so SolidWorks 2025 displays zero mojibake.

## Mandatory OCP 3D-to-2D Patent Figure Rules (原生 OCP 投影出图铁律)

1.  **Real 3D Helical Springs Through HLR + Per-View Geometry Cache (`*_hlr_cache.pkl`)**:
    - Pass all real 3D helical springs directly through `BRepAlgoAPI_Cut` (when sectioned) and `HLRBRep_Algo` (`VCompound` + `OutLineVCompound`, calling `BRepLib.BuildCurve3d_s(edge)` before `BRepAdaptor_Curve`, deflection `0.15–0.25 mm`). Never overlay un-occluded hand-drawn 2D zig-zags.
    - Cache projected 2D outlines and extracted section polygons per view keyed by `(model_hash, state, view_dir, cut_config, whitelist)` (plus a `--no-cache` CLI flag) so callout, breakline, or line-weight tweaks re-render all figures in ~2 seconds while any view/cut/whitelist change cleanly invalidates only the affected view.
2.  **Foreground Occluder Masking for Local-Section Hatches**:
    - In local-section views where opaque foreground parts lie in front of the cut plane (`Y < -0.5`), project their 2D `(X, Z)` silhouettes (`BRepMesh_IncrementalMesh` triangulation union) and boolean-subtract (`difference`) them from all `Y=0` hatch segments so hatches never bleed through foreground sleeves or links.
3.  **Automated Severed-Edge Detection & Complete Wavy-Breakline Closure**:
    - Before clipping any zoomed or broken view (`zoom_win`), programmatically scan every whitelisted part's 3D `BoundingBox` against all four `zoom_win` edges (`xmin < wx0`, `xmax > wx1`, `zmin < wy0`, `zmax > wy1`) and list every severed edge.
    - Every severed rod, shaft, pillar, sleeve, or sectioned plate MUST be closed by a `make_wavy_break_line()` spanning its **exact projected width/height** (never open-ended parallel lines, never straight-line hatched cuts). Embed all wavy breakline sequences directly into the Shapely `clip_poly` boundary AND draw them in `details`. Never place an artificial breakline across a ring's true inner-bore edge or clip adjacent bellows folds.
4.  **Isometric & Modular-Exploded View Standards**:
    - **Isometric View**: Use a clean front-elevated orthographic basis (e.g., `(0.48, -0.82, 0.31)`) and place callouts **strictly on front-facing visible parts/surfaces** (never pointing to camera-occluded rear sides).
    - **Exploded View**: NEVER scatter 50–100+ small parts (lines blur into noise at 300 DPI / 2/3 scale). Group the assembly into **5–7 major functional modules** exploded cleanly along a single axis (typically `Z`), hiding tiny internal springs so modular assembly relationships read at a glance.
5.  **CNIPA CPC Spec & Crisp 300 DPI Line-Weight Hierarchy**:
    - Lock **DPI = 300** (never `> 300 DPI`), physical size **≤ 16.0 × 24.5 cm** (target width `~15.4 cm`), pure white-background monochrome **JPG** (`quality=96, subsampling=0`, `dpi=(300, 300)`), plus companion `.svg` and 5-layer R12 `.dxf`.
    - Crisp 300 DPI line hierarchy: main outlines `0.65–0.70 pt` (`butt`/`miter`), wavy breaklines/details `0.36–0.38 pt`, hatches `0.24–0.25 pt` (wider pitch on large housings, tighter on thin walls; 45°/135° alternating metals, 45°+135° cross-hatch non-metals, unhatched solid shafts/pins/bolts/rails), smooth arrowless micro-arc leaders `0.40–0.42 pt`, sans-serif numerals `9.5 pt`.
6.  **Minimalist Centerlines & Zero-Crossing Callout Verification**:
    - **One Figure = One Single View**; **default omit centerlines** on full sections, local sections, bottom views, enlargements, isometric, and exploded views (reserve a single dot-dash symmetry axis strictly for true half-section views).
    - Route smooth quadratic-Bézier micro-arc leaders into monotonically Y-sorted left/right columns with automated swap optimization and `0 crossings` assertion; verify every pointer lands squarely on its visible target part (never floating, grazing neighbors, or long-traversing dense assemblies).
7.  **5-Section DOCX In-Place Backfill**:
    - Replace Section 2 (`摘要附图`) and Section 5 (`说明书附图`) images in place (appending new figures with centered `图 N` captions and page breaks, and syncing `附图说明` text/numerals), preserving native aspect ratio (`≤ 15.5 × 23.5 cm`), OpenXML MIME compliance, and all 5-section headers/footers.

## Execution Workflow

1.  **Step 1 — On-Demand View Planning, Alignment Matrix & Scope Check**: Design the **minimal, clearest view combination** needed to clearly show every claim feature, operating state, and internal mechanism (never force a fixed figure count like 6 figures), and map 100% of `附图说明` numerals to single-view figures (`图 1`–`图 N`). Run Gate 0 if creating/modifying a 3D model; for user-supplied frozen models, run a read-only sanity check and go straight to Step 2B.
2.  **Step 2 — 3D Modeling (Gates 1–6) & OCP Export**: Pass multi-state Gate 5 (`0` interferences, `0` broken solids, `0` floating parts), run OCP HLR with per-view geometry caching + foreground-occluder hatch clipping + automated severed-edge wavy-breakline closure, and backfill the 5-section DOCX.
3.  **Step 3 — Visual Critique, PY-to-Text Cross-Check & Delivery**: Inspect every rendered 300 DPI JPG and backfilled DOCX page via `view_file` (checking all severed edges for wavy-breakline closure, visible callout landings, zero hatch bleed-through, and CPC DPI/size compliance), and verify 100% geometric/force-chain consistency between the 3D PY script and the patent text before delivery.
