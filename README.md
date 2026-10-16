# Chem Project - 含能材料 QSAR 项目

## 项目简介

Chem Project 是一个专注于化工领域数据分析的机器学习项目，主要用于含能材料的定量构效关系（QSAR）研究。项目包含两个核心预测模块：爆热（DH）预测和热分解温度（TDT）预测。

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
│       └── test.ipynb           # 测试笔记本
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
│       └── test.ipynb           # 测试笔记本
├── .gitignore
└── .vscode/settings.json        # VS Code 配置
```

## 主要功能

### 1. 数据处理

- **数据选择与预处理**：采用过滤-嵌入-包裹三步法进行特征选择
- **数据清洗**：处理缺失值、异常值和数据标准化
- **多格式支持**：支持 Excel（.xlsx）数据文件读取与分析

### 2. 机器学习模型

- **随机森林回归器 (RandomForestRegressor)**：基于决策树的集成学习方法，适用于回归任务
- **XGBoost**：极端梯度提升算法，具有高效的并行计算能力和优秀的预测性能

### 3. 模型对比分析

- 不同模型的性能对比评估
- 多维度评估指标分析（R²、RMSE、MAE 等）

## 技术栈

- **Python 3.x**：主要编程语言
- **Jupyter Notebook**：交互式开发环境
- **Pandas**：数据处理与分析
- **NumPy**：数值计算
- **Scikit-learn**：机器学习算法库
- **XGBoost**：梯度提升框架
- **openpyxl**：Excel 文件处理

## 使用方法

### 环境要求

确保已安装以下 Python 库：

```bash
pip install pandas numpy scikit-learn xgboost openpyxl matplotlib seaborn
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

3. 在浏览器中打开相应的笔记本文件：
   - **DH 模块**：`DH/notebook/DataSelector.ipynb`（数据选择）
   - **DH 模块**：`DH/notebook/RandomForestRegressor.ipynb`（随机森林模型）
   - **DH 模块**：`DH/notebook/XGBoost.ipynb`（XGBoost 模型）
   - **TDT 模块**：`TDT/notebook/data_selector.ipynb`（数据选择）

## 项目模块说明

### DH 模块（爆热预测）

包含完整的机器学习预测流程：

- 数据加载与预处理
- 特征工程与选择
- RandomForest 模型训练与评估
- XGBoost 模型训练与评估
- 模型性能对比分析
- 结果可视化

### TDT 模块（热分解温度预测）

专注于数据选择与基础分析：

- 多版本数据集处理（fixed、random、unique_random）
- 数据唯一性分析
- 3D 可视化尝试
- 键解离能（BDE）计算分析

## 数据说明

项目使用 Excel 格式（.xlsx）作为数据源：

| 数据文件 | 描述 |
|---------|------|
| DH/data/test.xlsx | DH 模块测试数据 |
| TDT/data/test.xlsx | TDT 模块测试数据 |
| TDT/data/fixed.xlsx | TDT 模块固定数据集 |
| TDT/data/random.xlsx | TDT 模块随机数据集 |
| TDT/data/unique_random.xlsx | TDT 模块唯一随机数据集 |

## 开发环境

- **IDE**：VS Code（配置见 `.vscode/settings.json`）
- **版本控制**：Git
- **代码托管**：Gitee

## 许可证

本项目仅供学习和研究使用。

## 贡献者

感谢所有为项目做出贡献的开发者。

---

*如有问题或建议，欢迎提交 Issue 或 Pull Request。*