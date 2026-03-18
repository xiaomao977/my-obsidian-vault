# Prompt：CR-MSLM 融合实现任务

将以下内容作为完整 prompt 交给 AI 执行：

---

## 任务说明

请根据以下融合路线图，在**新文件夹 `CR-MSLM`** 中实现三个版本的融合代码。`CR-MSLM` 必须放在与 `CNDRR` 和 `four-stage-AYC` 同级或独立的位置，**不能**放在这两个项目目录内。

---

## 背景与目标

将 CNDRR 的鲁棒思想融合进 four-stage-AYC 的 MSLM 模型，形成 Capped-Robust MSLM（CR-MSLM）。融合后的目标函数为：

$$\min_{L, S, t} \|L\|_* + \eta\|L\|_{2,1} + \lambda \sum_j w_j \|S_{:,j}\|_2 + \gamma \cdot \text{tr}(L^\top \Delta_L L)$$
$$\text{s.t.} \quad P(D \circ t) = L + S, \quad w_j = \mathbf{1}[\|S_{:,j}\|_2 \leq \varepsilon]$$

其中：
- $\|L\|_*$：低秩项
- $\eta\|L\|_{2,1}$：行稀疏项（对应 CNDRR 的 $\eta$）
- $w_j \|S_{:,j}\|_2$：Capped L2,1 项（对应 CNDRR 的 A 矩阵）
- $\gamma \cdot \text{tr}(L^\top \Delta_L L)$：LPP 空间一致性项（对应 CNDRR 的 $\gamma$）

---

## 实现要求

### 1. 输出目录

- 新建文件夹：`CR-MSLM`
- 路径示例：`E:\CR-MSLM\` 或与 CNDRR、four-stage-AYC 同级
- 所有新代码只放在 `CR-MSLM` 中，不修改原项目目录

### 2. 三个优先级对应三个实现版本

| 版本 | 文件名 | 融合内容 | 对应优先级 |
|------|--------|----------|------------|
| **CR-MSLM_v1** | `MSLM_CR_v1.m` | 仅 Capped L2,1-norm（S 列方向 + dt 更新中异常 patch 置零） | 优先级 1 ★★★ |
| **CR-MSLM_v2** | `MSLM_CR_v2.m` | v1 + 核范数 + L2,1-norm 联合正则（L 行稀疏） | 优先级 2 ★★ |
| **CR-MSLM_v3** | `MSLM_CR_v3.m` | v2 + 自适应 ε + LPP 空间一致性正则 | 优先级 3 ★ |

### 3. 代码规范

- 在关键步骤处添加中文注释，说明：当前步骤、对应公式、参数含义
- 在文件开头用注释块说明：版本、融合项、主要参数及默认值
- 参数命名与 CNDRR 保持一致：`epsilon`（capped 阈值）、`eta`（L 行稀疏）、`gamma`（LPP 项）、`lambda`（S 稀疏）

---

## 详细实现说明

### 版本 1：MSLM_CR_v1.m（优先级 1）

**修改点：**

1. **Capped L2,1 权重**  
   - 对每列 $j$ 计算 $\|S_{:,j}\|_2$  
   - 若 $\|S_{:,j}\|_2 > \varepsilon$，则令 $S_{:,j} = \mathbf{0}$（该 patch 视为异常，不参与低秩估计）

2. **几何对齐更新**  
   - 在更新 `dt` 时，若 `col_norms(j) > epsilon`，则 `dt(j) = 0`，否则按原逻辑更新

3. **参数**  
   - 新增 `epsilon`（默认建议 0.5–2.0，可调）

**注释要求：** 在 S 更新、dt 更新处标注「CNDRR Capped Norm 融合」及对应公式。

---

### 版本 2：MSLM_CR_v2.m（优先级 2）

**在 v1 基础上增加：**

1. **L 的联合正则**  
   - 将 L 的更新从纯 SVD soft-thresholding 改为 nuclear + L2,1 联合正则  
   - 参考 four-stage-AYC 中的 `solve_nuclear_l21_joint_norm.m` 实现或调用

2. **参数**  
   - 新增 `eta`（L 的 L2,1 系数，默认 1e-2）

**注释要求：** 在 L 更新处说明「对应 CNDRR 的 $\eta\|P\|_{2,1}$ 项」。

---

### 版本 3：MSLM_CR_v3.m（优先级 3）

**在 v2 基础上增加：**

1. **自适应 ε**  
   - 每次外循环开始时，用当前 S 的列范数计算 $\mu_S + 3\sigma_S$  
   - 将 `epsilon` 更新为 `min(epsilon_init, mu_S + 3*sigma_S)`，或采用类似策略

2. **LPP 空间一致性**  
   - 构造 patch 空间邻域图：相邻 patch 权重为 1，其余为 0  
   - 在目标函数中加入 $\gamma \cdot \|L \cdot (D - W)\|_F^2$，其中 $D$ 为度矩阵，$W$ 为邻接矩阵

3. **参数**  
   - 新增 `gamma`（LPP 系数，默认 0.01）  
   - 新增 `epsilon_init`（ε 初始值）

**注释要求：** 在 ε 更新和 LPP 项处说明「对应 CNDRR 的 LPP 项与自适应 ε」。

---

## 依赖与接口

- 三个版本均需保持与 `AYC.m` 的调用接口一致：`[L, S, t] = MSLM_CR_vX(bi, m, n, num, t_init, ...)`  
- 额外参数通过 `varargin` 或 `options` 传入，如 `epsilon`, `eta`, `gamma`  
- 复用 four-stage-AYC 中的：`vec.m`, `Binarization.m`, `Patching.m`, `Counting.m`, `project_dual.m` 等，通过 `addpath` 引入，或将这些依赖复制到 `CR-MSLM` 并注明来源

---

## 验收标准

1. 三个 `.m` 文件均位于 `CR-MSLM` 文件夹内  
2. 每个文件有清晰的版本说明和参数注释  
3. 能独立运行或通过主脚本调用，主脚本也放在 `CR-MSLM` 中  
4. 提供 `main_CR_MSLM.m` 作为入口，可选择运行 v1/v2/v3，并输出计数结果与运行时间

---

## 执行顺序

1. 创建 `CR-MSLM` 文件夹  
2. 实现 `MSLM_CR_v1.m`（优先级 1）  
3. 实现 `MSLM_CR_v2.m`（优先级 2）  
4. 实现 `MSLM_CR_v3.m`（优先级 3）  
5. 实现 `main_CR_MSLM.m` 主脚本  
6. 复制或引用必要依赖，确保可运行

请按上述顺序完成实现，并在每个文件中加入足够的中文注释，便于理解和后续修改。
