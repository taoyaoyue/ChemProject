# Chem Project - Chemical Engineering Data Analysis Project

## Project Overview

Chem Project is a machine learning project focused on data analysis in the chemical engineering field, containing two main modules:

- **DH Module** - Contains Jupyter notebooks for data selection, Random Forest Regression, and XGBoost models
- **TDT Module** - Jupyter notebook for data selection and analysis

## Project Structure

```
chem_project/
├── DH/
│   ├── data/
│   │   └── test.xlsx          # Test Dataset
│   └── notebook/
│       ├── DataSelector.ipynb # Data Selector
│       ├── RandomForestRegressor.ipynb  # Random Forest Regression Model
│       ├── XGBoost.ipynb      # XGBoost Model Implementation
│       ├── contrast.ipynb     # Model Comparison Analysis
│       └── test.ipynb         # Test Notebook
├── TDT/
│   ├── data/
│   │   └── test.xlsx          # Test Dataset
│   └── notebook/
│       ├── data_selector.ipynb # Data Selector
│       └── test.ipynb         # Test Notebook
├── .gitignore
└── .vscode/settings.json
```

## Main Features

### 1. Data Processing
- Data selection and preprocessing
- Excel data file reading and analysis

### 2. Machine Learning Models
- **Random Forest Regressor** - Ensemble learning method based on decision trees
- **XGBoost** - Extreme Gradient Boosting algorithm

### 3. Model Comparison Analysis
- Performance comparison of different models
- Evaluation metric analysis

## Technology Stack

- Python 3.x
- Jupyter Notebook
- Pandas (Data Processing)
- NumPy (Numerical Computing)
- Scikit-learn (Machine Learning)
- XGBoost (Gradient Boosting)
- Excel File Handling

## Usage Instructions

### Environment Requirements
Ensure the following Python libraries are installed:
```bash
pip install pandas numpy scikit-learn xgboost openpyxl
```

### Running the Project
1. Start the Jupyter Notebook server:
```bash
jupyter notebook
```

2. Open the corresponding notebook file in the browser:
- DH Module: `DH/notebook/DataSelector.ipynb`
- TDT Module: `TDT/notebook/data_selector.ipynb`

## Project Module Descriptions

### DH Module
Contains the complete machine learning workflow:
- Data loading and preprocessing
- Feature selection
- Model training and evaluation
- Result visualization and comparison

### TDT Module
Focused on data selection and basic analysis.

## Data Format

The project uses Excel format (.xlsx) as the data source, including:
- Test Dataset: TDT/data/test.xlsx, DH/data/test.xlsx

## Development Environment

- VS Code Configuration: .vscode/settings.json
- Version Control: Git

## License

This project is for learning and research purposes only.

## Contributors

Thanks to all developers who have contributed to the project.