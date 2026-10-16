# Chem Project —— 含能材料 QSAR 项目

## 项目简介

Chem Project 是一个专注于化学人工智能领域数据分析的机器学习项目，主要用于含能材料的定量构效关系（QSAR）研究。项目包含两个核心预测模块：爆热（DH，Detonation Heat）预测和热分解温度（TDT）预测。

## 项目结构

```
chem_project/
├── DH/                          # 爆热预测模块
│   ├── data/
│   │   └── test.xlsx            # 测试数据集
│   └── notebook/
│       ├── DataSelector.ipynb   # 数据选择器
│       ├── RandomForestRegressor.ipynb  # 随机森林回归模型
│       ├── XGBoost.ipynb        # XGBoost 模型实现
│       ├── contrast.ipynb       # 模型对比分析
│       └── test.ipynb           # 测试 Notebook
├── TDT/                         # 热分解温度预测模块
│   ├── data/
│   │   ├── test.xlsx            # 测试数据集
│   │   ├── fixed.xlsx           # 固定数据集
│   │   ├── random.xlsx          # 随机数据集
│   │   └── unique_random.xlsx   # 唯一随机数据集
│   └── notebook/
│       ├── data_selector.ipynb  # 数据选择器
│       ├── 3D_try.ipynb         # 3D 可视化尝试
│       ├── BDE_Producer.ipynb   # 键解离能计算
│       ├── unique.ipynb         # 唯一性分析
│       └── test.ipynb           # 测试 Notebook
├── .gitignore
└── .vscode/settings.json        # VS Code 配置
```

## 主要功能

### 1. 数据处理

- **数据选择与预处理**：采用过滤—嵌入—包裹三步法进行特征选择
- **数据清洗**：修复错误的 SMILES、删除互变异构体，处理缺失值与异常值，并对数据进行标准化
- **数据文件读取**：支持读取与分析 Excel（.xlsx）格式的数据文件

### 2. 机器学习模型

- **随机森林回归器（RandomForestRegressor）**：基于决策树的集成学习方法，适用于回归任务
- **XGBoost**：极端梯度提升算法，具有高效的并行计算能力和优秀的预测性能

### 3. 3D 分子构型生成

- 使用 RDKit 直接生成可供 xTB 使用的构象
- 将 xTB 优化后生成的 .xyz 文件中的 3D 构象信息回填到 Mol 对象中

## 技术栈

### 一、基础环境

- **Python 3.11**：主要编程语言（conda 环境 `cheminfo`）
- **Jupyter Notebook**：交互式开发环境
- **Conda**：Python 环境管理（含独立的 `xtb` 环境）

### 二、数据处理与数值计算

- **Pandas**：数据读取、清洗与表格处理（含 `read_excel`）
- **NumPy**：数值计算与矩阵运算
- **openpyxl**：Excel（.xlsx）文件读写引擎
- **re / pathlib / os / shutil / subprocess**：正则解析、路径管理与外部程序调用

### 三、化学信息学

- **RDKit**：分子解析与处理（SMILES→Mol、加氢、SMARTS 子结构匹配、PandasTools 渲染）
  - 3D 构象生成：`AllChem.ETKDGv3` + `EmbedMultipleConfs`
  - 构象预优化：`UFFOptimizeMoleculeConfs`
  - 几何/构象封装：`Conformer`、`Point3D`、`MolToXYZFile`
- **Mordred（mordredcommunity）**：2D 分子描述符计算（`Calculator` + `descriptors`，共 1613 维）
- **scikit-fingerprints（skfp）**：基于分子骨架划分数据集（`randomized_scaffold_train_test_split`）

### 四、量子化学 / 半经验计算

- **xTB（GFN2-xTB）**：半经验量子化学计算引擎，通过命令行调用
  - 用途：几何优化（`--opt`）、单点能（`--scc`）、键级分析（`--wbo`）、JSON 结果输出（`--json`）、多核并行（`P 4`）
  - 计算目标：键解离能（BDE）、Wiberg 键级（WBO）、总能量（Hartree，并转换为 kJ/mol）

### 五、机器学习

- **Scikit-learn**：机器学习算法库
  - 模型：`RandomForestRegressor`、`XGBRegressor`（用作交叉验证基线）
  - 特征工程：`VarianceThreshold`、相关性去冗余、基于随机森林重要性的特征嵌入
  - 模型评估：`cross_validate`、`KFold`、`RepeatedKFold`、R² / MAE / RMSE
- **XGBoost**：梯度提升框架（最终的回归模型，用于预测热分解温度 Td）
- **pygad**：用于特征选择的遗传算法（GA）（种群规模 140、锦标赛选择、两点交叉、精英保留）

### 六、并行与可视化

- **Joblib**：多进程并行（`Parallel` + `delayed`，`n_jobs=-1`）
- **Matplotlib**：可视化（如适应度曲线）

### 七、数据与工作流

- **数据源**：`fixed.xlsx`（SMILES + Td(°C) 标签）
- **目标变量**：热分解温度 Td（°C）
- **特征构成**：Mordred 描述符 + 最大 WBO + BDE（C-NO<sub>2</sub>、N-NO<sub>2</sub>、O-NO<sub>2</sub> 三种键）

## 使用方法

### 环境要求

#### Python 版本

- Python 3.11（`cheminfo` 环境）

#### 依赖库

请确保已安装以下 Python 库：

```bash
pip install pandas numpy scikit-learn xgboost openpyxl matplotlib joblib rdkit mordredcommunity scikit-fingerprints pygad
```

### 运行项目

1. 克隆项目到本地：

```bash
git clone https://gitee.com/peach_moon/chem_project.git
cd chem_project
```

2. 启动 Jupyter Notebook 服务器：

```bash
jupyter notebook
```

3. 在浏览器中打开对应的 Notebook 文件：
   
   - **DH 模块**：`DH/notebook/RandomForestRegressor.ipynb`（随机森林模型）
   
   - **TDT 模块**：`TDT/notebook/test.ipynb`（数据选择）

4. 向开发者索取 xTB 计算的结果文件，或删除 Notebook 中包含量子化学描述符的代码单元。

## 项目模块说明

### DH 模块（爆热预测）

包含完整的机器学习预测流程：

- 数据加载与预处理
- 特征工程与特征选择
- RandomForest 模型的训练与评估
- XGBoost 模型的训练与评估
- 模型性能对比分析
- 结果可视化

### TDT 模块（热分解温度预测）

专注于数据选择与基础分析：

- 多版本数据集处理（fixed、random、unique_random）
- 数据唯一性分析
- 3D 可视化尝试
- 键解离能（BDE）计算与分析

## 数据说明

项目使用 Excel 格式（.xlsx）作为数据源：

| 数据文件                        | 描述                         |
| --------------------------- | -------------------------- |
| **DH/data/test.xlsx**       | DH 模块的原始测试数据集              |
| TDT/data/test.xlsx          | TDT 模块的原始测试数据集             |
| **TDT/data/fixed.xlsx**     | TDT 模块去重后再修复 SMILES 的测试数据集 |
| TDT/data/random.xlsx        | TDT 模块随机化后的测试数据集           |
| TDT/data/unique_random.xlsx | TDT 模块随机化后再去重的测试数据集        |

注：加粗的为最终使用的数据集。

## 开发环境

- **IDE**：VS Code（配置见 `.vscode/settings.json`）
- **版本控制**：Git
- **代码托管**：Gitee

## 许可证

本项目仅供学习和研究使用。

---

*如有问题或建议，欢迎提交 Issue 或 Pull Request。*