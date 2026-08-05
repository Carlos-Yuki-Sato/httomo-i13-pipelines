# HTTomo Pipeline Collection — I13-2

HTTomo YAML pipelines for the Diamond Light Source **I13-2** (coherence branch) beamline.
Covers all standard experimental configurations used at I13-2.

---

## Directory Structure

```
├── radiography/              16 files  — save projections without reconstruction
│   ├── radiography_x*.yaml             (4)  180° / any scan: normalised projections
│   ├── radiography_360_auto_overlap_x*.yaml  (4)  360°: auto overlap detection
│   ├── radiography_360_fixed_overlap_x*.yaml (4)  360°: hardcoded overlap
│   └── radiography_360_overlap_sweep_x*.yaml (4)  360°: sweep overlap to find best stitching
│
├── 180_degree/               28 files  — standard 180° tomography
│   ├── x1_25/   7 files
│   ├── x2/      7 files
│   ├── x4/      7 files
│   └── x10/     7 files
│
├── 360_degree/               28 files  — extended-FOV 360° tomography
│   ├── x1_25/   7 files
│   ├── x2/      7 files
│   ├── x4/      7 files
│   └── x10/     7 files
│
├── TXM/                      10 files  — Transmission X-ray Microscopy (zone-plate optics)
│   ├── radiography_TXM.yaml                        180° projections
│   ├── 180_TXM_auto_COR.yaml                       180° tomo, auto COR
│   ├── 180_TXM_COR_sweep.yaml                      180° tomo, COR sweep
│   ├── 180_TXM_fixed_COR.yaml                      180° tomo, fixed COR
│   ├── radiography_360_auto_overlap_TXM.yaml       360° projections, auto overlap
│   ├── radiography_360_fixed_overlap_TXM.yaml      360° projections, fixed overlap
│   ├── radiography_360_overlap_sweep_TXM.yaml      360° projections, overlap sweep
│   ├── 360_TXM_auto_COR.yaml                       360° tomo, auto overlap + COR
│   ├── 360_TXM_COR_sweep.yaml                      360° tomo, COR sweep
│   └── 360_TXM_fixed_COR.yaml                      360° tomo, fixed overlap + COR
│
├── experimental/             — algorithm comparisons and parameter sweeps (not for routine use)
│   ├── eval/
│   └── parameter_sweeps/ + recon_comparison/
│
└── Proposals/                — per-visit copies with visit-specific distortion paths
    ├── mg42769-1/
    └── mg43632-1/
```

---

## Naming Convention

### Standard objectives (1.25×, 2×, 4×, 10×)

```
<scan>_<objective>_<phase>_<COR>.yaml
```

| Part | Values |
|------|--------|
| `scan` | `180`, `360` |
| `objective` | `x1_25`, `x2`, `x4`, `x10` |
| `phase` | `absorption`, `paganin_sweep`, `paganin_fixed` |
| `COR` | `auto_COR`, `COR_sweep`, `fixed_COR` (omitted for `paganin_fixed` full recon with auto COR) |

### Radiography

| Filename pattern | Description |
|---|---|
| `radiography_x*.yaml` | 180° (or any scan): load, correct, save projections |
| `radiography_360_auto_overlap_x*.yaml` | 360°: auto overlap → stitch → save |
| `radiography_360_overlap_sweep_x*.yaml` | 360°: sweep overlap → stitch → save (thin slice) |
| `radiography_360_fixed_overlap_x*.yaml` | 360°: hardcoded overlap → stitch → save |

### TXM

| Filename pattern | Description |
|---|---|
| `radiography_TXM.yaml` | 180° projections |
| `180_TXM_auto_COR.yaml` | 180° tomo, auto COR (`find_center_vo`) |
| `180_TXM_COR_sweep.yaml` | 180° tomo, sweep COR (thin slice) |
| `180_TXM_fixed_COR.yaml` | 180° tomo, hardcoded COR |
| `radiography_360_*_TXM.yaml` | 360° projections (same three variants as standard) |
| `360_TXM_auto_COR.yaml` | 360° tomo, auto overlap + COR (`find_center_360`) |
| `360_TXM_COR_sweep.yaml` | 360° tomo, sweep COR after 360→180 (thin slice) |
| `360_TXM_fixed_COR.yaml` | 360° tomo, hardcoded overlap + COR |

---

## Pixel Sizes

### Standard objectives

| Objective | Effective pixel size |
|-----------|---------------------|
| 1.25×     | 2.600 µm            |
| 2×        | 1.625 µm            |
| 4×        | 0.8125 µm           |
| 10×       | 0.325 µm            |

Camera: pco.Edge 4.2 (3.25 µm native pixel pitch).

### TXM

The TXM effective pixel size depends on the zone plate configuration in use and changes between setups.
The COR position in pixels therefore also changes. Always check the TXM log or ask the beamline
scientist for the current configuration before filling in COR or overlap values.

---

## Recommended Workflow

### 180° Absorption Scans

1. **Auto COR** (`180_xN_absorption_auto_COR.yaml`): run first for a quick full reconstruction.
   Update `distortion_correction metadata_path`.
2. **COR sweep** (`180_xN_absorption_COR_sweep.yaml`): if auto-COR is poor, sweep a thin
   slice and pick the sharpest visually.
3. **Fixed COR** (`180_xN_absorption_fixed_COR.yaml`): full reconstruction with the COR from step 2.

### 180° Phase-Contrast (Paganin) Scans

4. **Paganin sweep — auto COR** (`180_xN_paganin_sweep.yaml`): sweep `ratio_delta_beta` with auto COR.
5. **Paganin sweep — fixed COR** (`180_xN_paganin_sweep_fixed_COR.yaml`): same sweep with a hardcoded COR.
6. **Paganin fixed** (`180_xN_paganin_fixed.yaml`): full reconstruction with the δ/β and COR from above.
7. **Paganin fixed — fixed COR** (`180_xN_paganin_fixed_COR.yaml`): as above, COR hardcoded.

### 360° Extended-FOV Tomography Scans

8. **Auto COR** (`360_xN_absorption_auto_COR.yaml`): `find_center_360` finds overlap and COR automatically.
9. **COR sweep** (`360_xN_absorption_COR_sweep.yaml`): manual overlap → sweep COR on thin slice.
10. **Fixed COR** (`360_xN_absorption_fixed_COR.yaml`): full reconstruction with known overlap and COR.
11–14. **Paganin variants** follow the same pattern: `360_xN_paganin_sweep.yaml`, `360_xN_paganin_sweep_fixed_COR.yaml`,
    `360_xN_paganin_fixed.yaml`, `360_xN_paganin_fixed_COR.yaml`.

### 360° Radiography

15. **Auto overlap** (`radiography_360_auto_overlap_x*.yaml`): run first — detects overlap and side
    automatically, stitches 360→180, saves projections.
16. **Overlap sweep** (`radiography_360_overlap_sweep_x*.yaml`): if auto-detection is unreliable, sweep
    the overlap value on a thin slice and pick the cleanest stitching visually.
17. **Fixed overlap** (`radiography_360_fixed_overlap_x*.yaml`): use once overlap and side are known.

### TXM 180°

18. **Radiography** (`radiography_TXM.yaml`): check sample/beam alignment — saves 180° projections.
19. **Auto COR** (`180_TXM_auto_COR.yaml`): full reconstruction; update `cor_initialisation_value` to
    ~half the detector width for your current TXM configuration.
20. **COR sweep** (`180_TXM_COR_sweep.yaml`): adjust sweep range around ~half the detector width.
21. **Fixed COR** (`180_TXM_fixed_COR.yaml`): full reconstruction with the COR from step 20.

### TXM 360°

22. **360° radiography — auto overlap** (`radiography_360_auto_overlap_TXM.yaml`): stitch and save
    projections with auto overlap detection.
23. **360° radiography — overlap sweep** (`radiography_360_overlap_sweep_TXM.yaml`): find best overlap visually.
24. **360° radiography — fixed overlap** (`radiography_360_fixed_overlap_TXM.yaml`): fast rerun with known overlap.
25. **Auto COR** (`360_TXM_auto_COR.yaml`): `find_center_360` finds overlap and COR, then reconstructs.
26. **COR sweep** (`360_TXM_COR_sweep.yaml`): set overlap/side from step 25, sweep COR on thin slice.
27. **Fixed COR** (`360_TXM_fixed_COR.yaml`): full reconstruction with known overlap and COR.

---

## Before Running Any Pipeline

Every file that requires user input is marked with `# CHANGE_HERE`. Search for this tag and update:

| Parameter | What to set |
|-----------|-------------|
| `metadata_path` | Replace `YEAR/VISIT` with the current visit, e.g. `2026/cm44160-2` |
| `distance` | `(detector_position_mm − 910) / 1000` (in metres) |
| `energy` | Beam energy in keV (e.g. `27.0`) |
| `center` | COR in pixels — from sweep or auto-find result |
| `overlap` | 360° overlap in pixels — from `find_center_360` or overlap sweep output |
| `side` | `'left'` or `'right'` — from `find_center_360` output |
| `ratio_delta_beta` | δ/β ratio — from Paganin sweep result |
| `cor_initialisation_value` | TXM only: set to ~half the detector width for your current configuration |

### Distortion Correction Paths (standard objectives only — not used in TXM)

| Objective | Path suffix |
|-----------|-------------|
| 1.25×     | `Pos1_125x/coefficients_bwfw.txt` |
| 2×        | `Pos2_2x/coefficients_bwfw.txt` |
| 4×        | `Pos4_4x/coefficients_bwfw.txt` |
| 10×       | `Pos5_10x/coefficients_bwfw.txt` |

Full path template:
```
/dls/i13/data/<YEAR>/<VISIT>/processing/distortion_corrections/results_4thOrder/<suffix>
```

---

## Proposals Folder

The `Proposals/` directory contains per-visit copies of the complete standard pipeline set
(180°, 360°, radiography for all objectives) with the `metadata_path` already set to the
correct visit directory. To add a new proposal:

1. Copy a recent `Proposals/<visit-id>/` folder.
2. Replace the visit path in all `distortion_correction metadata_path` fields.
3. Commit the new folder under `Proposals/<new-visit-id>/`.

---

## Recording Experiments with Git Branches

The `master` branch holds clean templates with `# CHANGE_HERE` placeholders and is never modified.
For each experiment, create a dedicated branch, fill in the actual values, and push it — giving you
a permanent, searchable record of every reconstruction you ran.

```bash
# Start from the clean master
git checkout master
git pull

# Create a branch for this experiment (use a descriptive name)
git checkout -b visit/cm44160-2-tomo-001

# Edit the pipeline(s) you are using — fill in all CHANGE_HERE values
# e.g. set metadata_path, distance, energy, center ...

# Commit the filled-in version with a short description
git add 180_degree/x4/180_x4_absorption_auto_COR.yaml
git commit -m "cm44160-2 tomo-001: 4x absorption, energy 27 keV, dist 0.12 m"

# Push the branch — master is left untouched
git push -u origin visit/cm44160-2-tomo-001
```

On GitHub you can browse any past experiment branch to see exactly which parameters were used,
and compare branches to spot differences between runs. Suggested branch naming:

```
visit/<visit-code>-<short-description>
# e.g.
visit/cm44160-2-bone-4x
visit/cm44160-2-mouse-360-paganin
visit/cm44160-3-calibration
```

---

## HTTomo

Documentation: https://diamondlightsource.github.io/httomo/

On a Diamond machine:

```bash
module load httomo
httomo --help
```

---

## Notes

- **Paganin pipelines** do **not** include `minus_log` — the Paganin filter handles the logarithm internally.
- **Absorption and radiography pipelines** include `minus_log` to convert normalised intensity to line integrals of attenuation.
- **TXM pipelines** do **not** include `distortion_correction_proj_discorpy` — zone-plate optics have negligible barrel distortion.
- **Sweep pipelines** (COR sweep, Paganin sweep, overlap sweep) produce one output per sweep value and run on a thin detector slice for speed; they omit `calculate_stats`, `rescale_to_int`, and the full `save_to_images` step.
- **360° radiography** applies `sino_360_to_180` before saving to avoid duplicated/mirrored projections from the second half of the scan.
- **Full-reconstruction pipelines** save **16-bit uint16 TIFF** images (1–99 percentile clipped).
- **Radiography pipelines** use `axis: 0` in `save_to_images` to save projections, not sinograms.
