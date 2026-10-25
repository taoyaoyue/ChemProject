# Chem Project —— 含能材料 QSAR（爆热 DH / 热分解温度 TDT）

用分子结构（SMILES）预测两个性质：

1. 爆热 Q（cal/g，n = 88）

2. 热分解温度 Td（°C，主表 1426 条）

技术栈：RDKit（分子解析、3D 构象）、Mordred（2D 描述符）、xTB GFN2（几何优化 / WBO / BDE）、scikit-learn + XGBoost（建模）、pyGAD（GA 特征选择）。

---

## 0. 读者想要复现什么？（先看这张表）

| 想复现                  | 需要仓库外的文件吗                       | 跑哪本 notebook                                          | 产物 / 输出                    | 耗时                  |
| -------------------- | ------------------------------- | ----------------------------------------------------- | -------------------------- | ------------------- |
| **TDT 数据清洗与溯源**      | 不需要                             | `data_prep/` 里按顺序 2 本（fixed_provenance → source_vs_td）                       | `TDT/out/` 两个 xlsx         | 半分钟（实测 10 s + 17 s） |
| **DH 模型结果**          | 不需要                             | `DH/notebook/RandomForestRegressor.ipynb`             | notebook 里打印 R² / RMSE     | 约 30 秒（实测 27 s）     |
| **TDT 模型结果**         | 需要 `Parents/` + `xtb_work/BDE/` | `TDT/notebook/modeling/test.ipynb`                    | notebook 里打印 R² / RMSE     | 约 1 分钟（实测 46 s）     |
| **TDT 从零重算几何 / BDE** | 需要 xTB 程序                       | `modeling/3D_try.ipynb`、`modeling/BDE_Producer.ipynb` | `Parents/`、`xtb_work/BDE/` | 数小时～数天              |

---

## 1. 环境（从零开始）

### 1.1 需要什么

| 项      | 要求                   | 说明                                       |
| ------ | -------------------- | ---------------------------------------- |
| Python | **3.11**（实测 3.11.15） | 别用太新的（3.13 上 `rdkit`、`mordred` 可能没有现成轮子） |
| 系统     | Windows / Linux 都行   | 本项目在 Windows 11 上实测；xTB 各平台都有发布版         |
| 磁盘     | 装完依赖约 1 GB           | 若要跑 TDT 模型，另需 `Parents/` + `BDE/` 几个 GB  |
| 显卡     | **不需要**              | 纯 CPU，8 核够用                              |

### 1.2 建环境（环境名不用一样）

作者本机用的是 conda 环境叫 `cheminfo`，读者不必叫这个名字，用 venv 也完全可以：

```bash
# 方式 A：conda
conda create -n chem python=3.11 -y
conda activate chem

# 方式 B：venv（没装 conda 就用这个）
python -m venv .venv
.venv\Scripts\activate          # Windows
# source .venv/bin/activate     # macOS / Linux
```

建好、激活之后，按 1.3 那张表决定要装哪些包。

### 1.3 跑什么 → 要装什么

| 想跑                        | 必需包                                                        | 还需要别的东西吗                            |
| ------------------------- | ---------------------------------------------------------- | ----------------------------------- |
| 第 2 节 数据清洗                | `pandas` `numpy` `scipy` `scikit-learn` `rdkit` `openpyxl` | 不需要                                 |
| 第 3.1 节 DH 模型             | 上面 + `mordredcommunity` `scikit-fingerprints`              | 不需要                                 |
| 第 3.2 节 TDT 模型            | 上面 + `xgboost` `joblib`                                    | `Parents/` + `xtb_work/BDE/`（见 1.6） |
| TDT 特征筛选（`data_selector`） | 上面 + `pygad` `matplotlib`                                  | 同上；GA 很慢，可选                         |
| 从零重算几何 / BDE              | 上面全部                                                       | xTB 可执行文件（见 1.6）                    |

### 1.4 装包

按 1.3 那张表挑需要的装。

**如果只跑数据清洗**（第 2 节），6 个包就够：

```bash
pip install pandas numpy scipy scikit-learn rdkit openpyxl
```

**如果要跑全部代码**（含模型）：

```bash
pip install -r requirements.txt
```

> **已经装过这些库的话，这一步可以直接跳过。** `pip install -r` 是幂等的：已满足的包只会显示 `Requirement already satisfied`，不会把你的版本升级或降级。

### 1.5 装完自检（都通过就算齐了）

```bash
#只装了数据清洗那 6 个包：
python -c "import pandas, numpy, scipy, sklearn, rdkit, openpyxl; print('OK')"

#装了 requirements.txt 全部：
python -c "import pandas, numpy, scipy, sklearn, rdkit, openpyxl, mordred, skfp, xgboost, pygad, matplotlib; print('OK')"

python config.py
```

`python -c "import ..."` 没报错 = 包装齐了；

`python config.py` 打印出各路径的“存在=True/False” = 路径通不通。

> 两个包名要注意：代码里 `import mordred` 实际装的是 **`mordredcommunity`**（原版 `mordred` 不支持新 Python）；`import skfp` 实际装的是 **`scikit-fingerprints`**。

### 1.6 两个 pip 装不了的东西

1. **xTB**（只有“从零重算几何 / BDE”才需要）：到 https://github.com/grimme-lab/xtb/releases 下对应平台的包，解压后把 `xtb.exe`（Linux 是 `xtb`）的完整路径设给环境变量 `CHEM_XTB`。
2. **仓库外的大文件**（只有跑 TDT 模型才需要）：`Parents/`（1426 个 `parent_XXXX/`，含 `conf.xyz` 和 `wbo`）与 `xtb_work/BDE/`（每分子三类键的单点能）。体积大、不进仓库，**向项目作者索要**，拿到后设 `CHEM_PARENTS` / `CHEM_XTB_WORK` 指过去。

```powershell
$env:CHEM_PARENTS  = "D:\Parents"
$env:CHEM_XTB_WORK = "D:\xtb_work"
$env:CHEM_XTB      = "D:\xtb\bin\xtb.exe"      # 只有重算才要
```

### 1.7 实测的版本（仅供参考，不必逐一对齐）

下表是 2026-10-24 在本机跑通时的版本；`requirements.txt` 里只写了下界，不强制这个组合。

| 包                   | 实测版本     |
| ------------------- | -------- |
| Python              | 3.11.15  |
| pandas              | 3.0.5    |
| numpy               | 2.4.6    |
| scipy               | 1.17.1   |
| scikit-learn        | 1.9.0    |
| rdkit               | 2026.3.5 |
| openpyxl            | 3.1.5    |
| mordredcommunity    | 2.0.7    |
| scikit-fingerprints | 2.1.0    |
| xgboost             | 3.2.0    |
| pygad               | 3.7.0    |

---

## 2. 复现数据清洗结果（只要下载仓库）

### 输入（都在仓库里）

- `TDT/data/fixed.xlsx`：主表，1426 行，两列 `Td(°C)` / `SMILES`
- `TDT/data/source/`：两篇来源文献的原始表（`datasetwithheader.csv` 1022 行、`Td_zcy.csv` 886 行）

### 步骤

1. 打开 `TDT/notebook/data_prep/fixed_provenance.ipynb` → 全部运行
   → 产出 `TDT/out/fixed_去10行_带来源.xlsx`（产出为 3 个表。文件名里的行数随实际剔除数自动变）
2. 打开 `TDT/notebook/data_prep/source_vs_td.ipynb` → 全部运行（读第 1. 步的产物）
   → 产出 `TDT/out/实验组别与Td关系.xlsx`（5 个表）

**用 `git clone` 拿仓库的话，不用改任何路径**：notebook 会从自己所在目录往上爬，找到仓库根再 `import config`，所有路径都从 `config.py` 取。

> ⚠️ 但这两本认的"仓库根标记"是 **`.git`**，不是 `config.py`（`unique` 和 `modeling/` 下 5 本才认 `config.py`）。
> 所以：**用 `git clone` 拿仓库 → 两本都能直接跑；下载 zip（没有 `.git`）→ 这两本会一路爬到盘符根，然后报 `ModuleNotFoundError: No module named 'config'`**，此时把这两本第 2 格的 `.git` 改成 `config.py` 即可。

### 跑对了应该看到

| 检查点          | 期望值                                                          |
| ------------ | ------------------------------------------------------------ |
| 数据行数         | 1426 → **1416**（剔 10 行：两个来源都查不到）                             |
| 来源分布（1416 行） | 只在 2025：**560**；只在 2023：**450**；两篇都有：**406**                 |
| 来源组别 vs Td   | Pearson r = **+0.1614**（n = 1010，p = 2.5e-07）；η² = **1.88%** |
| 控制分子组成后      | 两组残差均值差 **+2.20 °C**，置换 p ≈ **0.50** → 差异不显著，由分子组成解释，不是测定方法  |

产物里有什么：

| 表               | 内容                                                         |
| --------------- | ---------------------------------------------------------- |
| `fixed_去10行带来源` | 1416 行主表，8 列：`Td(°C)` / `SMILES` + `来源` + `A_2025_Td` / `B_2023_Td` + `Td与A一致` / `Td与B一致` + `主表新编号` |
| `剔除的10行`        | 10 行 × 9 列：`原行号(1起)` / `原0起下标` + 每行剔除原因 + **要排除的目录**（`fixed` 第 i 行 ↔ `parent_{i:04d}`，一一对应） |
| `两篇温度冲突`        | 106 行；其中 \|Δ\|>5 的真冲突只有 34 行，别把 106 当冲突数                   |

> 溯源只管"每条来自哪篇、要不要剔、两篇哪边对不上"；**来源与 Td 的相关性、η²、控制组成后的残差检验都在 `source_vs_td.ipynb`**，两个 notebook 不重复算。

---

## 3. 复现模型结果

### 3.1 DH（爆热）：只要仓库

`DH/notebook/RandomForestRegressor.ipynb` → 10 折 CV，**R² = 0.791 ± 0.059，RMSE 171.0，MAE 132.9**（n = 88）；另有 Murcko 骨架 8:2 留出 **R² = 0.787**。
（`DataSelector.ipynb` 是特征筛选 + GA、`XGBoost.ipynb` 是另一个模型，按需跑。）

### 3.2 TDT（Td）：要仓库外的两个目录

这两个目录太大，不进仓库，用环境变量指过去：

| 目录              | 里面是什么                                                       | 环境变量            |
| --------------- | ----------------------------------------------------------- | --------------- |
| `Parents/`      | 1426 个 `parent_XXXX/`，每个含 `conf.xyz`（xTB 优化后的 3D 几何）和 `wbo` | `CHEM_PARENTS`  |
| `xtb_work/BDE/` | 每个分子 `BDE_{i:04d}/`，含三类键（C-/N-/O-NO₂）的单点能 json              | `CHEM_XTB_WORK` |

```powershell
$env:CHEM_PARENTS  = "D:\Parents"
$env:CHEM_XTB_WORK = "D:\xtb_work"
```

然后跑 `TDT/notebook/modeling/test.ipynb`：读 `fixed.xlsx[0:713]`（713 条）→ 45 个 Mordred 描述符 + `max_wbo` + 3 个 BDE = 49 维 → XGBoost → 10 折 CV。

预期：**R² = 0.630 ± 0.044，RMSE 40.4 °C，MAE 30.4 °C**（10 折 CV，n = 713）；另有 Murcko 骨架 8:2 留出 **R² = 0.554**。

> - `modeling/data_selector.ipynb`：特征筛选 + GA（426 条子集），可选，GA 很慢。
> - `modeling/3D_try.ipynb` / `BDE_Producer.ipynb`：从零重算几何 / BDE，要 xTB 程序（`CHEM_XTB`），耗时按天算，一般不需要跑。

> 上面两节的指标都是 **2026-10-24 用本仓库代码实测**的输出（本机 16 线程），不是抄报告的；若你改了特征列或超参，以 notebook 实际打印为准。

---

## 4. 目录结构

```
chem_project/
├── config.py                    # 唯一的路径出口（改路径只改这里）
├── requirements.txt
├── DH/                          # 爆热模块（只要仓库就能跑）
│   ├── data/test.xlsx           # 88 条
│   └── notebook/                # DataSelector / RandomForest / XGBoost / contrast / test
└── TDT/                         # 热分解温度模块
    ├── data/
    │   ├── fixed.xlsx           # 主表 1426 行（两列：Td(°C) + SMILES）
    │   ├── test.xlsx            # 原始 1504
    │   ├── random.xlsx          # 随机化后 1504
    │   ├── unique_random.xlsx   # 去重后 1435
    │   └── source/              # 两篇来源文献的原始表（溯源用）
    ├── notebook/
    │   ├── data_prep/           # 数据清洗与核对：unique / fixed_provenance / source_vs_td
    │   └── modeling/            # 3D_try / BDE_Producer / data_selector / test
    └── out/                     # notebook 产物（已 gitignore，跑完自动生成）
```

数据清洗与建模**分目录、不混**：`data_prep/` 只要下载仓库就能跑，`modeling/` 需要仓库外的 3D 几何与 xTB。

---

## 5. 路径约定（`config.py`）

notebook 里不写死绝对路径，需要路径就 `import config`（**DH 的 5 本例外**：它们用 `Path.cwd().parent / "data" / "test.xlsx"` 取数据、不 import config，所以要在 `DH/notebook` 目录下打开）：

| 变量                                   | 含义                              | 环境变量覆盖          |
| ------------------------------------ | ------------------------------- | --------------- |
| `config.DATA_TDT` / `config.DATA_DH` | 仓库内的数据目录                        | —               |
| `config.SOURCE`                      | 两篇文献的原始表（`TDT/data/source/`）    | —               |
| `config.OUT`                         | 产物目录，默认 `TDT/out/`              | `CHEM_OUT`      |
| `config.PARENTS`                     | 仓库外：`parent_XXXX/conf.xyz` 所在目录 | `CHEM_PARENTS`  |
| `config.XTB`                         | 仓库外：`xtb.exe`                   | `CHEM_XTB`      |
| `config.XTB_WORK`                    | 仓库外：BDE 计算工作目录                  | `CHEM_XTB_WORK` |

在仓库根跑 `python config.py`，会逐条打印这台机器上各路径是否存在 —— 换机器先跑它自查。

---

## 6. 数据说明

| 文件                            | 说明                                                                    |
| ----------------------------- | --------------------------------------------------------------------- |
| **`TDT/data/fixed.xlsx`**     | **主表**：1426 行，去重 + 修复 SMILES + 删 Td 冲突之后的版本；2026-10-24 修正过 2 行 Td（见下） |
| `TDT/data/unique_random.xlsx` | 主表的上一步：1435 行                                                         |
| `TDT/data/random.xlsx`        | 原始 1504 行随机打乱后（带原下标）                                                  |
| `TDT/data/test.xlsx`          | 原始 1504 行                                                             |
| `TDT/data/source/*.csv`       | 两篇来源文献的原始表（A：2025 那篇；B：2023 那批）                                       |

> **2026-10-24 修正**：`fixed.xlsx` **0 起下标 87、434**（= `parent_0087` / `parent_0434`，即 Excel 第 89、436 行）的 Td，由 334.1 / 221.0 改回 **324.5 / 257.0**。
> 原因：A（2025 那篇）把同一化合物录了两遍（一条画成自由基写法），清洗时留下了温度不对的那条；
> 完整溯源见 `TDT/notebook/data_prep/fixed_provenance.ipynb` 第 6 节。

⚠️ 两个口径别混：

- **1426 行 = 主表**，建模用的就是它。
- **1416 行（`fixed_去10行_带来源.xlsx`）= 辅助表**，只用于按来源分层 / 溯源统计，不要当主表替换。

---

## 7. 跑不通先看这里

| 报错 / 现象                                         | 原因与处理                                                     |
| ----------------------------------------------- | --------------------------------------------------------- |
| `ModuleNotFoundError: No module named 'config'` | notebook 被单独拷出仓库了，要在仓库里面打开；若是在 `data_prep/` 两本上报的，多半是下载的 zip（没有 `.git`）→ 见 2 节那条 ⚠️ |
| `FileNotFoundError: ... fixed_去*行_带来源.xlsx`     | 先跑 `data_prep/fixed_provenance.ipynb`（第 2 节第 1 步）         |
| 找不到 `parent_0012` / `BDE_0012`                  | `CHEM_PARENTS` / `CHEM_XTB_WORK` 没设或指错目录                  |
| 产物在哪                                            | 默认 `TDT/out/`；想换就设 `CHEM_OUT`（环境变量要在 `import config` 之前设） |
| 想确认路径                                           | 仓库根跑 `python config.py`                                   |

---

本项目仅供学习和研究使用。
