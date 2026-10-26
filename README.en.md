# Chem Project — QSAR for Energetic Materials (Heat of Detonation DH / Thermal Decomposition Temperature TDT)

Predict two properties from molecular structure (SMILES):

1. heat of detonation Q (cal/g, n = 88)

2. thermal decomposition temperature Td (°C, main table 1426 rows)

Stack: RDKit (parsing, 3D conformers), Mordred (2D descriptors), xTB GFN2 (geometry optimization / WBO / BDE), scikit-learn + XGBoost (modeling), pyGAD (GA feature selection).

> English mirror of `README.md` (as of 2026-10-26). If the two ever disagree, the Chinese `README.md` is authoritative.

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

The table below lists the versions that worked on my machine on 2026-10-24.

`requirements.txt` only sets lower bounds; it does not pin this combination.

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
- `TDT/data/source/`: raw tables of the two source papers (`datasetwithheader.csv` 1022 rows, `Td_zcy.csv` 886 rows)

### Steps

1. Open `TDT/notebook/data_prep/fixed_provenance.ipynb` → Run All
   → writes `TDT/out/fixed_去10行_带来源.xlsx` (3 sheets; the row count in the filename `去N行` follows the actual number of exclusions)
2. Open `TDT/notebook/data_prep/source_vs_td.ipynb` → Run All (it reads step 1's output)
   → writes `TDT/out/实验组别与Td关系.xlsx` (5 sheets)

**If you got the repo with `git clone`, no path editing is needed**: each notebook walks up from its own directory, finds the repository root, then does `import config`; every path comes from `config.py`.

> ⚠️ Note which "repo root marker" these two notebooks accept: **`.git`**, not `config.py` (only `unique` and the 5 notebooks under `modeling/` accept `config.py`).
> So: **repo obtained via `git clone` → both notebooks run as-is; a downloaded ZIP (no `.git`) → these two climb all the way to the drive root and then fail with `ModuleNotFoundError: No module named 'config'`.** Fix: change `.git` to `config.py` in cell 2 of those two notebooks.

### What you should see if it worked

Each of the two notebooks has its own set of checkpoints; **finish notebook 1 before starting notebook 2**. Both tables matching = reproduction succeeded.

#### Notebook 1 `fixed_provenance.ipynb`: provenance + exclusion

| Checkpoint | Expected |
|---|---|
| Tables read (`fixed ... \| A ... \| B ...`) | `fixed (1426, 2)`, A `(1022, 4)`, B `(886, 2)` |
| Parse failures (`无法解析： ...`) | `fixed 0 \| A 0 \| B 0`; anything other than 0 means local RDKit cannot read some SMILES and the source split below will shift accordingly |
| Source split (`来源` value_counts) | `-1`: 10, `0`: 560, `1`: 450, `2`: 406 (0 = 2025 only, 1 = 2023 only, 2 = both papers) |
| Rows to drop (`要剔 N 行`) | **10 rows**, with `剔除原因` all equal to "source = A, InChIKey changed after SMILES repair so matching failed" (i.e. after the SMILES repair neither source table matches) |
| Rows kept (`剔 10 → 保留 N`) | 1426 − 10 = **1416** |
| Td conflicts between the two papers (`... 条，其中 \|差\|>5： ...`) | **106 rows**, of which only **34** have `\|Δ\|>5` |

`fixed_去10行_带来源.xlsx` has 3 sheets:

`fixed_去10行带来源`, `剔除的10行`, and `两篇温度冲突`.

#### Notebook 2 `source_vs_td.ipynb`: source group vs Td

| Checkpoint | Expected |
|---|---|
| Which product it read (`读： ...`) | `读： fixed_去10行_带来源.xlsx` (the `去N行` in the filename follows the exclusion count from the previous step) |
| Sample size | `n = 1416` (three groups); when comparing the "2025 only / 2023 only" pair, `n = 1010` (560 + 450, minus the 406 present in both papers) |
| Composition descriptor shape (`描述符维度 ...`) | `(1416, 13)` (7 functional-group types + heavy atom count / ring count / molecular weight / C, N, O counts) |
| Binary group vs Td (Q1) | `观测 r = +0.1614`, `p(公式) = 2.523e-07`, `置换 p = 0.000000`; the observed value falls **outside** the random-permutation range `[-0.1373, +0.1307]` |
| Three groups vs Td (Q1) | `η²(观测) = 0.0188` → the group explains only **1.88%** of the Td variance; ANOVA p = 1.525e-06 |
| Residuals after controlling composition (Q2) | `RF 5 折 CV R² = 0.411`, `两组残差均值差 = +2.20 °C`, `置换 p = 0.5030` → **not significant**; the gap is explained by molecular composition, not by the measurement method |

`实验组别与Td关系.xlsx` has 5 sheets:

`汇总`, `各组Td分布`, `置换落点分布`, `Q1与Q2对照`, and `两篇分子组成对比`.

> Provenance only answers "which paper each row comes from, which rows to drop, where the two papers disagree" (notebook 1); **the source-vs-Td correlation, η² and the composition-controlled residual test all live in `source_vs_td.ipynb`** (notebook 2).
>
> The numbers above are **measured readings re-run on the 1416-row basis on 2026-10-26**. If you switch numpy / sklearn versions the last decimals may move while the magnitudes do not — trust the `汇总` sheet printed by notebook 2.

---

## 3. Reproduce the model results

### 3.1 DH (heat of detonation): repo only

`DH/notebook/RandomForestRegressor.ipynb` →

10-fold CV, **R² = 0.791 ± 0.059, RMSE 171.0, MAE 132.9** (n = 88);

plus Murcko-scaffold 8:2 hold-out **R² = 0.787**.

(`DataSelector.ipynb` is feature selection + GA, `XGBoost.ipynb` is another model; run them on demand.)

### 3.2 TDT (Td): needs the two directories outside the repo

These two directories are too large to commit; point to them with environment variables:

| Directory | Contents | Environment variable |
|---|---|---|
| `Parents/` | 1426 `parent_XXXX/` folders, each with `conf.xyz` (xTB-optimized 3D geometry) and `wbo` | `CHEM_PARENTS` |
| `xtb_work/BDE/` | one `BDE_{i:04d}/` per molecule, holding the single-point json of three bond types (C-/N-/O-NO₂) | `CHEM_XTB_WORK` |

```powershell
$env:CHEM_PARENTS  = "D:\Parents"
$env:CHEM_XTB_WORK = "D:\xtb_work"
```

Then run `TDT/notebook/modeling/test.ipynb`: it reads the first 713 rows of `fixed.xlsx` (you can adjust how many molecules are taken with `BEGIN` and `END` in cell 2) → 45 Mordred descriptors + `max_wbo` + 3 BDEs = 49 dimensions → XGBoost → 10-fold CV.

Expected with all 713 molecules:

 **R² = 0.630 ± 0.044, RMSE 40.4 °C, MAE 30.4 °C** (10-fold CV, n = 713);

plus Murcko-scaffold 8:2 hold-out **R² = 0.554**.

> - `modeling/data_selector.ipynb`: feature selection + GA (426-row subset), optional, the GA is slow.
> - `modeling/3D_try.ipynb` / `BDE_Producer.ipynb`: recompute geometries / BDE from scratch; needs the xTB binary (`CHEM_XTB`) and takes hours; usually not needed.

> All metrics in the two sections above are outputs **measured on 2026-10-24 with the code of this repository** (16 threads on my machine). If you change the feature columns or hyper-parameters, trust what the notebook actually prints.

---

## 4. Layout

```
chem_project/
├── config.py                    # the single exit point for paths (change paths only here)
├── requirements.txt
├── DH/                          # heat of detonation module (repo only)
│   ├── data/test.xlsx           # 88 rows
│   └── notebook/                # DataSelector / RandomForest / XGBoost / contrast / test
└── TDT/                         # thermal decomposition temperature module
    ├── data/
    │   ├── fixed.xlsx           # main table, 1426 rows (two columns: Td(°C) + SMILES)
    │   ├── test.xlsx            # raw 1504
    │   ├── random.xlsx          # randomized 1504
    │   ├── unique_random.xlsx   # deduplicated 1435
    │   └── source/              # raw tables of the two source papers (for provenance)
    ├── notebook/
    │   ├── data_prep/           # data cleaning & checks: unique / fixed_provenance / source_vs_td
    │   └── modeling/            # 3D_try / BDE_Producer / data_selector / test
    └── out/                     # notebook outputs (gitignored, created on run)
```

Data cleaning and modeling live in separate directories: `data_prep/` runs from a plain repo checkout, `modeling/` needs the 3D geometries outside the repo plus xTB.

---

## 5. Paths (`config.py`)

Notebooks never hard-code absolute paths; when they need a path they `import config` (**the 5 DH notebooks are the exception**: they take data via `Path.cwd().parent / "data" / "test.xlsx"` and do not import `config`, so open them from the `DH/notebook` directory):

| Variable | Meaning | Environment override |
|---|---|---|
| `config.DATA_TDT` / `config.DATA_DH` | data directories inside the repo | — |
| `config.SOURCE` | raw tables of the two papers (`TDT/data/source/`) | — |
| `config.OUT` | output directory, default `TDT/out/` | `CHEM_OUT` |
| `config.PARENTS` | outside: directory holding `parent_XXXX/conf.xyz` | `CHEM_PARENTS` |
| `config.XTB` | outside: `xtb.exe` | `CHEM_XTB` |
| `config.XTB_WORK` | outside: BDE calculation working directory | `CHEM_XTB_WORK` |

Run `python config.py` at the repository root: it prints, one by one, whether every path exists on this machine — run it first when you move to a new machine.

---

## 6. Data

| File | Notes |
|---|---|
| **`TDT/data/fixed.xlsx`** | **main table**: 1426 rows, after deduplication + SMILES repair + Td-conflict removal. Two Td values were corrected on 2026-10-24 (see below) |
| `TDT/data/unique_random.xlsx` | the step before the main table: 1435 rows |
| `TDT/data/random.xlsx` | raw 1504 rows after random shuffling (with original indices) |
| `TDT/data/test.xlsx` | raw 1504 rows |
| `TDT/data/source/*.csv` | raw tables of the two source papers (A is the 2025 paper, B is the 2023 batch) |

> **2026-10-24 correction**: the Td of **0-based rows 87 and 434** in `fixed.xlsx` (= `parent_0087` / `parent_0434`, i.e. Excel rows 89 and 436) was changed from 334.1 / 221.0 back to **324.5 / 257.0**.
> Reason: paper A (the 2025 one) records the same compound twice (one copy drawn as a radical form), and the cleaning kept the copy with the wrong temperature;
> full trace in `TDT/notebook/data_prep/fixed_provenance.ipynb`, section 6.

⚠️ Do not mix up the two bases:

- **1426 rows = main table**: this is what modeling uses.
- **1416 rows (`fixed_去10行_带来源.xlsx`) = auxiliary table**: only for per-source stratification / provenance statistics; do not substitute it for the main table.

---

## 7. Troubleshooting

| Symptom | Cause / fix |
|---|---|
| `ModuleNotFoundError: No module named 'config'` | the notebook was copied out of the repo; open it inside the repo. If it shows up in the two `data_prep/` notebooks, it is most likely a downloaded ZIP (no `.git`) → see the ⚠️ note in section 2 |
| `FileNotFoundError: ... fixed_去*行_带来源.xlsx` | run `data_prep/fixed_provenance.ipynb` first (section 2, step 1) |
| `parent_0012` / `BDE_0012` not found | `CHEM_PARENTS` / `CHEM_XTB_WORK` not set or pointing at the wrong directory |
| Where are the outputs? | `TDT/out/` by default; set `CHEM_OUT` to change it (the environment variable must be set before `import config`) |
| Want to confirm the paths | run `python config.py` at the repository root |

---

For study and research use only.
