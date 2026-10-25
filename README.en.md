# Chem Project — QSAR for Energetic Materials (DH / TDT)

Predict two properties from molecular structure (SMILES): **heat of detonation Q (cal/g, n = 88)** and **thermal decomposition temperature Td (°C, main table 1426 rows; modeling uses the first 713)**.

Stack: RDKit (parsing, 3D conformers), Mordred (2D descriptors), xTB GFN2 (geometry optimization / WBO / BDE), scikit-learn + XGBoost (modeling), pyGAD (GA feature selection).

---

## 0. What do you want to reproduce? (read this table first)

| Goal | Needs files outside the repo? | Which notebook | Output | Time |
|---|---|---|---|---|
| **TDT data cleaning & provenance** | No | the 2 notebooks in `data_prep/`, in order (fixed_provenance → source_vs_td) | two xlsx in `TDT/out/` | ~30 s (10 s + 17 s measured) |
| **DH model results** | No | `DH/notebook/RandomForestRegressor.ipynb` | R² / RMSE printed in the notebook | ~30 s (27 s measured) |
| **TDT model results** | Yes: `Parents/` + `xtb_work/BDE/` | `TDT/notebook/modeling/test.ipynb` | R² / RMSE printed in the notebook | ~1 min (46 s measured) |
| **Recompute TDT geometries / BDE from scratch** | Yes: the xTB binary | `modeling/3D_try.ipynb`, `modeling/BDE_Producer.ipynb` | `Parents/`, `xtb_work/BDE/` | hours to days |

---

## 1. Environment (from zero)

### 1.1 What you need

| Item | Requirement | Notes |
|---|---|---|
| Python | **3.11** (tested 3.11.15) | avoid very new releases: `rdkit` / `mordred` wheels may not exist for 3.13 |
| OS | Windows / Linux | developed and tested on Windows 11; xTB ships builds for both |
| Disk | ~1 GB for the Python packages | plus a few GB for `Parents/` + `BDE/` if you run the TDT model |
| GPU | **not needed** | CPU only; 8 cores is plenty |

### 1.2 Create the environment (any name works)

My local environment is a conda env named `cheminfo`; call yours whatever you like, or use venv:

```bash
# option A: conda
conda create -n chem python=3.11 -y
conda activate chem

# option B: venv (if you do not have conda)
python -m venv .venv
.venv\Scripts\activate          # Windows
# source .venv/bin/activate     # macOS / Linux
```

After creating and activating the environment, decide what to install from the table in 1.3.

### 1.3 What to install for what

| Goal | Required packages | Anything else? |
|---|---|---|
| Section 2, data cleaning | pandas numpy scipy scikit-learn rdkit openpyxl | no |
| Section 3.1, DH model | above + `mordredcommunity` `scikit-fingerprints` | no |
| Section 3.2, TDT model | above + `xgboost` `joblib` | `Parents/` + `xtb_work/BDE/` (see 1.6) |
| TDT feature selection (`data_selector`) | above + `pygad` `matplotlib` | same; the GA is slow, optional |
| Recompute geometries / BDE | all of the above | the xTB binary (see 1.6) |

### 1.4 Install

Pick what you need from the table in 1.3.

**Data cleaning only** (section 2) — six packages are enough:

```bash
pip install pandas numpy scipy scikit-learn rdkit openpyxl
```

**Everything** (including the models):

```bash
pip install -r requirements.txt
```

> **Already have these libraries? Skip this step.** `pip install -r` is idempotent: satisfied packages only print `Requirement already satisfied`; nothing gets upgraded or downgraded.

### 1.5 Check the installation (all must pass)

```bash
# only the six data-cleaning packages:
python -c "import pandas, numpy, scipy, sklearn, rdkit, openpyxl; print('OK')"

# everything from requirements.txt:
python -c "import pandas, numpy, scipy, sklearn, rdkit, openpyxl, mordred, skfp, xgboost, pygad, matplotlib; print('OK')"

python config.py
```

The `python -c "import ..."` line with no error = packages are complete; `python config.py` printing `exists=True/False` for each path = paths resolve.

> Two easy traps: `import mordred` comes from the package **`mordredcommunity`** (the original `mordred` does not support recent Python); `import skfp` comes from **`scikit-fingerprints`**.

### 1.6 Two things pip cannot install

1. **xTB** (only for "recompute geometries / BDE from scratch"): download the build for your platform from https://github.com/grimme-lab/xtb/releases, unpack it, and point the environment variable `CHEM_XTB` to `xtb.exe` (or `xtb` on Linux).
2. **The large files outside the repo** (only for the TDT model): `Parents/` (1426 `parent_XXXX/` folders with `conf.xyz` and `wbo`) and `xtb_work/BDE/` (single-point energies per molecule). Too large to commit — **ask the project author**, then point `CHEM_PARENTS` / `CHEM_XTB_WORK` at them.

```powershell
$env:CHEM_PARENTS  = "D:\Parents"
$env:CHEM_XTB_WORK = "D:\xtb_work"
$env:CHEM_XTB      = "D:\xtb\bin\xtb.exe"      # only for recomputation
```

### 1.7 Versions I tested (for reference only — no need to match)

Versions that worked on my machine on 2026-10-24. `requirements.txt` only sets lower bounds, it does not pin this combination.

| Package | Tested version |
|---|---|
| Python | 3.11.15 |
| pandas | 3.0.5 |
| numpy | 2.4.6 |
| scipy | 1.17.1 |
| scikit-learn | 1.9.0 |
| rdkit | 2026.3.5 |
| openpyxl | 3.1.5 |
| mordredcommunity | 2.0.7 |
| scikit-fingerprints | 2.1.0 |
| xgboost | 3.2.0 |
| pygad | 3.7.0 |

---

## 2. Reproduce the data-cleaning results (repo only)

### Inputs (all inside the repo)
- `TDT/data/fixed.xlsx`: main table, 1426 rows, two columns `Td(°C)` / `SMILES`
- `TDT/data/source/`: the datasets of the two source papers (`datasetwithheader.csv` 1022 rows, `Td_zcy.csv` 886 rows)

### Steps
1. Open `TDT/notebook/data_prep/fixed_provenance.ipynb` and run all
   → writes `TDT/out/fixed_去10行_带来源.xlsx` (3 sheets; the row count in the name follows the actual exclusions)
2. Open `TDT/notebook/data_prep/source_vs_td.ipynb` and run all (it reads step 1's output)
   → writes `TDT/out/实验组别与Td关系.xlsx` (5 sheets)

**With `git clone`, no path editing is needed**: each notebook walks up from its own directory to the repository root and takes every path from `config.py`.

> ⚠️ But note which marker these two notebooks look for: `fixed_provenance` / `source_vs_td` walk up looking for **`.git`**, not `config.py` (only `unique` and the 5 notebooks under `modeling/` look for `config.py`).
> So: **`git clone` → both run as-is; a downloaded ZIP (no `.git`) → these two climb to the drive root and fail with `ModuleNotFoundError: No module named 'config'`**. Fix: change `.git` to `config.py` in cell 2 of those two notebooks.

### What you should see if it worked

| Check | Expected |
|---|---|
| Rows | 1426 → **1416** (10 removed: no traceable source) |
| Source split (1416 rows) | 2025 only: **560**; 2023 only: **450**; both: **406** |
| Source group vs Td | Pearson r = **+0.1614** (n = 1010, p = 2.5e-07); η² = **1.88%** |
| After controlling composition | mean residual difference **+2.20 °C**, permutation p ≈ **0.50** → not significant; the gap is explained by molecular composition, not by different measurements |

Sheets in the outputs:

| Sheet | Content |
|---|---|
| `fixed_去10行带来源` | 1416-row main table, 8 columns: `Td(°C)` / `SMILES` + `来源` + `A_2025_Td` / `B_2023_Td` + `Td与A一致` / `Td与B一致` + `主表新编号` |
| `剔除的10行` | 10 rows × 9 columns: `原行号(1起)` / `原0起下标` + removal reason + **the directory to exclude** (`fixed` row i ↔ `parent_{i:04d}`, one-to-one) |
| `两篇温度冲突` | 106 rows; only 34 of them are real conflicts (\|Δ\|>5) |

> Provenance only answers "which paper each row comes from, which rows to drop, and where the two papers disagree". **Source-vs-Td correlation, η² and the composition-controlled residual test all live in `source_vs_td.ipynb`** — the two notebooks do not duplicate each other.

---

## 3. Reproduce the model results

### 3.1 DH (heat of detonation): repo only

`DH/notebook/RandomForestRegressor.ipynb` → 10-fold CV, **R² = 0.791 ± 0.059, RMSE 171.0, MAE 132.9** (n = 88); plus Murcko-scaffold 8:2 hold-out **R² = 0.787**.
(`DataSelector.ipynb` = feature selection + GA, `XGBoost.ipynb` = another model; optional.)

### 3.2 TDT (Td): needs two directories outside the repo

Too large to commit; point to them with environment variables:

| Directory | Contents | Environment variable |
|---|---|---|
| `Parents/` | 1426 `parent_XXXX/` folders, each with `conf.xyz` (xTB-optimized geometry) and `wbo` | `CHEM_PARENTS` |
| `xtb_work/BDE/` | one `BDE_{i:04d}/` per molecule with the single-point json of three bond types | `CHEM_XTB_WORK` |

```powershell
$env:CHEM_PARENTS  = "D:\Parents"
$env:CHEM_XTB_WORK = "D:\xtb_work"
```

Then run `TDT/notebook/modeling/test.ipynb`: reads `fixed.xlsx[0:713]` (713 rows) → 45 Mordred descriptors + `max_wbo` + 3 BDEs = 49 features → XGBoost → 10-fold CV.

Expected: **R² = 0.630 ± 0.044, RMSE 40.4 °C, MAE 30.4 °C** (10-fold CV, n = 713); plus Murcko-scaffold 8:2 hold-out **R² = 0.554**.

> - `modeling/data_selector.ipynb`: feature filtering + GA on a 426-row subset; optional, GA is slow.
> - `modeling/3D_try.ipynb` / `BDE_Producer.ipynb`: recompute geometries / BDE from scratch; needs the xTB binary (`CHEM_XTB`) and takes days. Usually not needed.

> All numbers above were **measured with the code of this repository on 2026-10-24** (16 threads), not copied from a report. If you change the feature list or hyper-parameters, trust what the notebook prints.

---

## 4. Layout

```
chem_project/
├── config.py                    # the only place paths live
├── requirements.txt
├── DH/                          # heat of detonation (repo only)
│   ├── data/test.xlsx           # 88 rows
│   └── notebook/                # DataSelector / RandomForest / XGBoost / contrast / test
└── TDT/                         # thermal decomposition temperature
    ├── data/
    │   ├── fixed.xlsx           # main table, 1426 rows (Td(°C) + SMILES)
    │   ├── test.xlsx            # raw 1504
    │   ├── random.xlsx          # randomized 1504
    │   ├── unique_random.xlsx   # deduplicated 1435
    │   └── source/              # raw tables of the two source papers
    ├── notebook/
    │   ├── data_prep/           # cleaning: unique / fixed_provenance / source_vs_td
    │   └── modeling/            # 3D_try / BDE_Producer / data_selector / test
    └── out/                     # notebook outputs (gitignored, created on run)
```

Data cleaning and modeling are kept apart: `data_prep/` runs from the repository alone, `modeling/` needs the 3D geometries and xTB.

---

## 5. Paths (`config.py`)

TDT notebooks never hard-code absolute paths; they `import config` (**the 5 DH notebooks are the exception**: they read data via `Path.cwd().parent / "data" / "test.xlsx"` and do not import `config`, so open them from `DH/notebook`):

| Variable | Meaning | Environment override |
|---|---|---|
| `config.DATA_TDT` / `config.DATA_DH` | data directories inside the repo | — |
| `config.SOURCE` | raw tables of the two papers (`TDT/data/source/`) | — |
| `config.OUT` | output directory, default `TDT/out/` | `CHEM_OUT` |
| `config.PARENTS` | outside: directory holding `parent_XXXX/conf.xyz` | `CHEM_PARENTS` |
| `config.XTB` | outside: `xtb.exe` | `CHEM_XTB` |
| `config.XTB_WORK` | outside: working directory for BDE calculation | `CHEM_XTB_WORK` |

Run `python config.py` at the repository root: it prints whether every path exists on this machine. Do that first on a new machine.

---

## 6. Data

| File | Notes |
|---|---|
| **`TDT/data/fixed.xlsx`** | **main table**: 1426 rows, after deduplication + SMILES repair + Td-conflict removal; 2 Td values corrected on 2026-10-24 (see below) |
| `TDT/data/unique_random.xlsx` | previous step: 1435 rows |
| `TDT/data/random.xlsx` | raw 1504 rows, shuffled (with original indices) |
| `TDT/data/test.xlsx` | raw 1504 rows |
| `TDT/data/source/*.csv` | raw tables of the two source papers (A: 2025; B: 2023) |

> **2026-10-24 correction**: Td of **0-based rows 87 and 434** in `fixed.xlsx` (= `parent_0087` / `parent_0434`, i.e. Excel rows 89 and 436) was changed from 334.1 / 221.0 back to **324.5 / 257.0**.
> Reason: dataset A (the 2025 paper) contains the same compound twice (one copy drawn as a radical form), and the cleaning kept the copy with the wrong temperature.
> Full trace: `TDT/notebook/data_prep/fixed_provenance.ipynb`, section 6.

⚠️ Do not mix up the two:
- **1426 rows = main table**, used for modeling.
- **1416 rows (`fixed_去10行_带来源.xlsx`) = auxiliary table**, only for per-source stratification / provenance. Do not substitute it for the main table.

---

## 7. Troubleshooting

| Symptom | Cause / fix |
|---|---|
| `ModuleNotFoundError: No module named 'config'` | the notebook was copied out of the repo; open it inside the repo. If it happens in the two `data_prep/` notebooks, it is usually a downloaded ZIP (no `.git`) → see the ⚠️ note in section 2 |
| `FileNotFoundError: ... fixed_去*行_带来源.xlsx` | run `data_prep/fixed_provenance.ipynb` first (section 2, step 1) |
| `parent_0012` / `BDE_0012` not found | `CHEM_PARENTS` / `CHEM_XTB_WORK` not set or pointing elsewhere |
| Where are the outputs? | `TDT/out/` by default; set `CHEM_OUT` to change it (before `import config`) |
| Check the paths | run `python config.py` at the repository root |

---

For study and research use only.
