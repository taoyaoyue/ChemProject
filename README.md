# Chem Project - 含能材料 QSAR 项目

## 项目简介

Chem Project 是一个专注于化工领域数据分析的机器学习项目，包含两个主要模块：

- **DH 模块** —— 包含数据选择、随机森林回归和 XGBoost 模型的 Jupyter 笔记本，用于爆热预测
- **TDT 模块** —— 数据选择与分析的 Jupyter 笔记本，用于热分解温度预测

## 项目结构

```text
chem_project/
├── DH/
│   ├── data/
│   │   └── test.xlsx          # 测试数据集
│   └── notebook/
│       ├── DataSelector.ipynb # 数据选择器
│       ├── RandomForestRegressor.ipynb  # 随机森林回归模型
│       ├── XGBoost.ipynb      # XGBoost 模型实现
│       ├── contrast.ipynb     # 模型对比分析
│       └── test.ipynb         # 测试笔记本
├── TDT/
│   ├── data/
│   │   └── test.xlsx          # 测试数据集
│   └── notebook/
│       ├── data_selector.ipynb # 数据选择器
│       └── test.ipynb         # 测试笔记本
├── .gitignore
└── .vscode/settings.json
```

## 主要功能

### 1. 数据处理

- 数据选择与预处理：过滤-嵌入-包裹三步
- Excel 数据文件读取与分析

### 2. 机器学习模型

- **随机森林回归器 (RandomForestRegressor)** —— 基于决策树的集成学习方法
- **XGBoost** —— 极端梯度提升算法

### 3. 模型对比分析

- 不同模型的性能对比
- 评估指标分析

## 技术栈

- Python 3.x
- Jupyter Notebook
- Pandas（数据处理）
- NumPy（数值计算）
- Scikit-learn（机器学习）
- XGBoost（梯度提升）
- Excel 文件处理

## 使用方法

### 环境要求

确保已安装以下 Python 库：

```bash
pip install pandas numpy scikit-learn xgboost openpyxl
```

### 运行项目

1. 启动 Jupyter Notebook 服务器：

```bash
jupyter notebook
```

2. 在浏览器中打开相应的笔记本文件：
   - DH 模块：`DH/notebook/DataSelector.ipynb`
   - TDT 模块：`TDT/notebook/data_selector.ipynb`

## 项目模块说明

### DH 模块

包含完整的机器学习流程：

- 数据加载与预处理
- 特征选择
- 模型训练与评估
- 结果可视化与对比

### TDT 模块

专注于数据选择与基础分析。

## 数据格式

项目使用 Excel 格式（.xlsx）作为数据源，包含：

- 测试数据集：`TDT/data/test.xlsx`、`DH/data/test.xlsx`

## 开发环境

- VS Code 配置：`.vscode/settings.json`
- 版本控制：Git

## 许可证

本项目仅供学习和研究使用。

## 贡献者

感谢所有为项目做出贡献的开发者。