# Chem Project — QSAR for Energetic Materials

## Overview

Chem Project is a machine learning project focused on data analysis in the field of chemical artificial intelligence. Its primary purpose is quantitative structure–activity relationship (QSAR) studies of energetic materials. The project contains two core prediction modules: heat of detonation (DH) prediction and thermal decomposition temperature (TDT) prediction.

## Project Structure

```
chem_project/
├── DH/                          # Heat of detonation prediction module
│   ├── data/
│   │   └── test.xlsx            # Test dataset
│   └── notebook/
│       ├── DataSelector.ipynb   # Data selector
│       ├── RandomForestRegressor.ipynb  # Random forest regression model
│       ├── XGBoost.ipynb        # XGBoost implementation
│       ├── contrast.ipynb       # Model comparison analysis
│       └── test.ipynb           # Test notebook
├── TDT/                         # Thermal decomposition temperature prediction module
│   ├── data/
│   │   ├── test.xlsx            # Test dataset
│   │   ├── fixed.xlsx           # Fixed dataset
│   │   ├── random.xlsx          # Random dataset
│   │   └── unique_random.xlsx   # Unique random dataset
│   └── notebook/
│       ├── data_selector.ipynb  # Data selector
│       ├── 3D_try.ipynb         # 3D visualization experiments
│       ├── BDE_Producer.ipynb   # Bond dissociation energy calculation
│       ├── unique.ipynb         # Uniqueness analysis
│       └── test.ipynb           # Test notebook
├── .gitignore
└── .vscode/settings.json        # VS Code configuration
```

## Key Features

### 1. Data Processing

- **Data selection and preprocessing**: three-step filter–embed–wrapper approach for feature selection
- **Data cleaning**: correcting invalid SMILES, removing tautomers, handling missing values and outliers, and standardizing the data
- **Data file reading**: reading and analyzing Excel (.xlsx) data files

### 2. Machine Learning Models

- **Random forest regressor (RandomForestRegressor)**: a decision-tree-based ensemble learning method suitable for regression tasks
- **XGBoost**: an extreme gradient boosting algorithm with efficient parallel computing and strong predictive performance

### 3. 3D Molecular Conformer Generation

- Generating conformers directly with RDKit for use with xTB
- Writing the 3D conformer information from the .xyz files produced by xTB optimization back into the Mol objects

## Tech Stack

### 1. Base Environment

- **Python 3.11**: primary programming language (conda environment `cheminfo`)
- **Jupyter Notebook**: interactive development environment
- **Conda**: Python environment management (including a separate `xtb` environment)

### 2. Data Processing and Numerical Computing

- **Pandas**: data reading, cleaning, and tabular processing (including `read_excel`)
- **NumPy**: numerical computing and matrix operations
- **openpyxl**: Excel (.xlsx) file read/write engine
- **re / pathlib / os / shutil / subprocess**: regular expression parsing, path management, and external program invocation

### 3. Cheminformatics

- **RDKit**: molecular parsing and processing (SMILES → Mol, hydrogen addition, SMARTS substructure matching, PandasTools rendering)
  - 3D conformer generation: `AllChem.ETKDGv3` + `EmbedMultipleConfs`
  - Conformer pre-optimization: `UFFOptimizeMoleculeConfs`
  - Geometry/conformer encapsulation: `Conformer`, `Point3D`, `MolToXYZFile`
- **Mordred (mordredcommunity)**: 2D molecular descriptor calculation (`Calculator` + `descriptors`, 1613 dimensions in total)
- **scikit-fingerprints (skfp)**: scaffold-based dataset splitting (`randomized_scaffold_train_test_split`)

### 4. Quantum Chemistry / Semiempirical Calculations

- **xTB (GFN2-xTB)**: semiempirical quantum chemistry engine, invoked from the command line
  - Uses: geometry optimization (`--opt`), single-point energy (`--scc`), bond-order analysis (`--wbo`), JSON output (`--json`), multi-core parallelism (`P 4`)
  - Targets: bond dissociation energy (BDE), Wiberg bond order (WBO), total energy (in Hartree, converted to kJ/mol)

### 5. Machine Learning

- **Scikit-learn**: machine learning library
  - Models: `RandomForestRegressor`, `XGBRegressor` (used as a cross-validation baseline)
  - Feature engineering: `VarianceThreshold`, correlation-based redundancy removal, feature embedding based on random forest importance
  - Model evaluation: `cross_validate`, `KFold`, `RepeatedKFold`, R² / MAE / RMSE
- **XGBoost**: gradient boosting framework (the final regression model, used to predict thermal decomposition temperature Td)
- **pygad**: genetic algorithm (GA) for feature selection (population size 140, tournament selection, two-point crossover, elitism)

### 6. Parallelism and Visualization

- **Joblib**: multiprocessing parallelism (`Parallel` + `delayed`, `n_jobs=-1`)
- **Matplotlib**: visualization (e.g., fitness curves)

### 7. Data and Workflow

- **Data source**: `fixed.xlsx` (SMILES + Td(°C) labels)
- **Target variable**: thermal decomposition temperature Td (°C)
- **Features**: Mordred descriptors + maximum WBO + BDE (for C-NO<sub>2</sub>, N-NO<sub>2</sub>, and O-NO<sub>2</sub> bonds)

## Usage

### Requirements

#### Python Version

- Python 3.11 (`cheminfo` environment)

#### Dependencies

Make sure the following Python packages are installed:

```bash
pip install pandas numpy scikit-learn xgboost openpyxl matplotlib joblib rdkit mordredcommunity scikit-fingerprints pygad
```

### Running the Project

1. Clone the repository:

```bash
git clone https://gitee.com/peach_moon/chem_project.git
cd chem_project
```

2. Start the Jupyter Notebook server:

```bash
jupyter notebook
```

3. Open the relevant notebook in your browser:
   
   - **DH module**: `DH/notebook/RandomForestRegressor.ipynb` (random forest model)
   
   - **TDT module**: `TDT/notebook/test.ipynb` (data selection)

4. Request the xTB output files from the developer, or delete the notebook cells that include the quantum chemistry descriptors.

## Module Overview

### DH Module (Heat of Detonation Prediction)

Covers the complete machine learning prediction pipeline:

- Data loading and preprocessing
- Feature engineering and feature selection
- Training and evaluation of the RandomForest model
- Training and evaluation of the XGBoost model
- Model performance comparison
- Result visualization

### TDT Module (Thermal Decomposition Temperature Prediction)

Focuses on data selection and basic analysis:

- Processing of multiple dataset versions (fixed, random, unique_random)
- Data uniqueness analysis
- 3D visualization experiments
- Bond dissociation energy (BDE) calculation and analysis

## Data Description

The project uses Excel (.xlsx) files as its data source:

| Data File                   | Description                                                    |
| --------------------------- | -------------------------------------------------------------- |
| **DH/data/test.xlsx**       | Original test dataset for the DH module                        |
| TDT/data/test.xlsx          | Original test dataset for the TDT module                       |
| **TDT/data/fixed.xlsx**     | TDT test dataset: deduplicated, then SMILES repaired           |
| TDT/data/random.xlsx        | TDT test dataset after randomization                           |
| TDT/data/unique_random.xlsx | TDT test dataset after randomization followed by deduplication |

Note: Entries in bold are the datasets ultimately used.

## Development Environment

- **IDE**: VS Code (configuration in `.vscode/settings.json`)
- **Version control**: Git
- **Code hosting**: Gitee

## License

This project is intended for learning and research purposes only.

---

*If you have any questions or suggestions, feel free to open an Issue or submit a Pull Request.*