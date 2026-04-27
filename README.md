# HTTomo Pipeline Collection — I13-2

HTTomo YAML pipelines for the Diamond Light Source **I13-2** (coherence branch) beamline.
Covers all standard experimental configurations used at I13-2.

---

## Directory Structure

```
pipelines/
├── radiography/          4 files  — save projections without reconstruction
├── 180_degree/          28 files  — standard 180° tomography
└── 360_degree/          28 files  — extended-FOV 360° tomography
```

---

## Naming Convention

```
<scan>_<objective>_<phase>_<COR>.yaml
```

| Part | Values |
|------|--------|
| `scan` | `radiography`, `180`, `360` |
| `objective` | `x125`, `x2`, `x4`, `x10` |
| `phase` | `absorption`, `paganin_sweep`, `paganin_fixed` |
| `COR` | `auto_COR`, `COR_sweep`, `fixed_COR` (omitted for radiography and paganin_fixed full recon) |

---

## Pixel Sizes

| Objective | Effective pixel size |
|-----------|---------------------|
| 1.25×     | 2.600 µm            |
| 2×        | 1.625 µm            |
| 4×        | 0.8125 µm           |
| 10×       | 0.325 µm            |

Camera: pco.Edge 4.2 (3.25 µm native pixel pitch).

---

## Recommended Workflow

### 180° Absorption Scans

1. **Auto COR** (`180_xN_absorption_auto_COR.yaml`): run first to get a full reconstruction quickly.
   Update only `distortion_correction metadata_path`.
2. **COR sweep** (`180_xN_absorption_COR_sweep.yaml`): if auto-COR gives a poor result, sweep a thin
   slice and find the correct value visually.
3. **Fixed COR** (`180_xN_absorption_fixed_COR.yaml`): full reconstruction with the COR from step 2.

### 180° Phase-Contrast (Paganin) Scans

4. **Paganin sweep — auto COR** (`180_xN_paganin_sweep.yaml`): sweep `ratio_delta_beta` with auto COR.
5. **Paganin sweep — fixed COR** (`180_xN_paganin_sweep_fixed_COR.yaml`): same sweep but with a
   known, hardcoded COR (use when auto-finding is unreliable).
6. **Paganin fixed** (`180_xN_paganin_fixed.yaml`): full reconstruction with the δ/β and COR from above.
7. **Paganin fixed — fixed COR** (`180_xN_paganin_fixed_COR.yaml`): as above, COR hardcoded.

### 360° Extended-FOV Scans

8. **Auto COR** (`360_xN_absorption_auto_COR.yaml`): uses `find_center_360` to find overlap and COR automatically.
9. **COR sweep** (`360_xN_absorption_COR_sweep.yaml`): manual overlap → sweep COR on thin slice.
10. **Fixed COR** (`360_xN_absorption_fixed_COR.yaml`): full reconstruction with known overlap and COR.
11. **Paganin sweep** (`360_xN_paganin_sweep.yaml`): 360° + Paganin δ/β sweep with auto COR.
12. **Paganin sweep — fixed COR** (`360_xN_paganin_sweep_fixed_COR.yaml`): 360° + Paganin sweep, hardcoded COR.
13. **Paganin fixed** (`360_xN_paganin_fixed.yaml`): full 360° + Paganin reconstruction.
14. **Paganin fixed — fixed COR** (`360_xN_paganin_fixed_COR.yaml`): as above, COR hardcoded.

---

## Before Running Any Pipeline

Every file that requires user input is marked with `# CHANGE_HERE`. Search for this tag and update:

| Parameter | What to set |
|-----------|-------------|
| `metadata_path` | Replace `YEAR/VISIT` with the current visit, e.g. `2026/cm44160-2` |
| `distance` | `(detector_position_mm − 910) / 1000` (in metres) |
| `energy` | Beam energy in keV (e.g. `27.0`) |
| `center` | COR in pixels — from sweep or auto-find result |
| `overlap` | 360° overlap in pixels — from `find_center_360` output |
| `side` | `'left'` or `'right'` — from `find_center_360` output |
| `ratio_delta_beta` | δ/β ratio — from Paganin sweep result |

### Distortion Correction Paths

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
git add 180_degree/180_x4_absorption_auto_COR.yaml
git commit -m "cm44160-2 tomo-001: 4x absorption, energy 27 keV, dist 0.12 m"

# Push the branch — master is left untouched
git push -u origin visit/cm44160-2-tomo-001
```

On GitHub you can then browse any past experiment branch to see exactly which parameters were used,
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

check the documentiona at: https://diamondlightsource.github.io/httomo/

and, if you are already on a Diamond machine, start with this :

```bash
module load httomo
httomo --help
```

---

## Notes

- **Paganin pipelines** do **not** include `minus_log` — the Paganin filter handles the logarithm internally.
- **Sweep pipelines** (COR sweep and Paganin sweep) produce one output slice per sweep value; they do **not** include `calculate_stats`, `rescale_to_int`, or `save_to_images`.
- **Radiography pipelines** use `axis: 0` in `save_to_images` to save projections, not sinograms.
- Typically all full-reconstruction pipelines save **16-bit uint16 TIFF** images (percentile-clipped).

