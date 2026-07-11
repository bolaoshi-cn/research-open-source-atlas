# causal-learn

从观测数据中学习因果结构，并比较多类因果发现算法。

PyWhy 生态的因果发现 Python 库，覆盖基于约束、评分和函数因果模型的多种方法及图操作。

- 官网详情：https://bolaoshi.cn/research-tools/causal-learn
- 原始来源：https://github.com/py-why/causal-learn
- 读取版本：e30895b7ad41c429be73975a6a0b6bdea8146a7e
- 核对日期：2026-07-10
- 上游许可：MIT
- 验证程度：catalogued
- 编辑判断：recommended

## 适合这些任务

- 需要探索因果图假设的研究者
- 希望比较多种因果发现算法的开发者

## 这些情况要谨慎

- 把算法输出直接当成已识别因果结论的任务
- 缺少领域假设和稳健性检查的研究

## 能力拆解

### 1. 因果图操作与评分

为因果发现算法提供底层支撑：(1) 因果图的统一表示与操作（DAG/CPDAG/PAG/PDAG 的端点矩阵表示、转换、查询）；(2) 结构学习的评分函数库（BIC/BDeu/CV/marginal 六类局部评分）；(3) 发现结果与真值的评估指标（SHD、邻接/箭头混淆矩阵）。这些是算法能运行和被评价的基础设施。

证据：source-reviewed

输入

- 图类：节点列表 nodes（GraphNode 对象）、边定义（endpoint 组合）。
- DAG2CPDAG：G（Dag 对象）。
- PDAG2DAG：PDAG 邻接矩阵。

输出

- 图操作：GeneralGraph 对象（可被 drawpydotgraph 可视化、tonxgraph 转 networkx）。
- 转换：新的 GeneralGraph（CPDAG/DAG/PAG）。
- 评分：float（局部评分值，越大约好或越差取决于评分定义）。

人工检查

- 端点矩阵的读图约定是否正确（迁移/比较时）。
- 评分函数选择是否匹配数据（线性→BIC、离散→BDeu、非线性→CV/marginal）。
- SHD 评估时真值图与估计图是否在同一端点约定下。

### 2. 基于函数因果模型的因果发现

利用特定函数因果模型（Functional Causal Model, FCM）的可识别性约束：主要是非高斯性（LiNGAM）、非线性加性噪声（ANM）或后非线性（PNL）：把数据拟合到结构方程模型后，通过残差独立性检验判定因果方向，从而在有向无环图上确定（而不仅是等价类）因果结构。覆盖线性非高斯（LiNGAM 家族）、非线性加性噪声（ANM）、后非线性（PNL）。

证据：source-reviewed

输入

- X：numpy ndarray，shape=(nsamples, nfeatures)，观测数据。
- LiNGAM 共有（BaseLiNGAM）：randomstate（种子）、priorknowledge（先验因果顺序矩阵，0=禁路径/1=必路径/-1=未知）、applypriorknowledgesoftly、measure（'pwling' 或 'kernel'，DirectLiNGAM 用）。
- ICALiNGAM：maxiter（FastICA 最大迭代，默认 1000）。

输出

- LiNGAM：adjacencymatrix（B 矩阵，B[i,j]=j→i 的系数）、causalorder（因果顺序列表）。
- VARLiNGAM：结构 VAR 系数 + 同期因果邻接矩阵 + BootstrapResult。
- ANM：方向判定（X→Y 或 Y→X）。

人工检查

- 数据是否满足非高斯（LiNGAM）/非线性加性（ANM）/后非线性（PNL）假设：核心可识别性人工门。
- causalorder 是否与领域时间/逻辑顺序一致。
- VARLiNGAM 的滞后阶是否合理（用 BIC 选后仍需人工确认）。

### 3. 基于分数的因果发现

不从条件独立性出发，而是为每个候选 DAG 定义一个评分函数（BIC/BDeu/CV 似然/marginal 似然），在 DAG 等价类空间或排列空间中搜索评分最优的因果结构，输出 CPDAG。覆盖贪心等价搜索（GES）、精确搜索（ExactSearch，DP/A）、排列类搜索（BOSS/GRaSP）和连续优化类方法（CALM）。

证据：source-reviewed

输入

- X：numpy ndarray，shape=(nsamples, nfeatures)，观测数据。GES 支持用 cov（协方差矩阵）+ n（样本量）替代 X（仅 BIC 类评分）。
- scorefunc：评分函数名，localscoreBIC / localscoreBDeu / localscoreCVgeneral / localscoremarginalgeneral / localscoreCVmulti / localscoremarginalmulti / localscoreBICfromcov。
- maxP（GES）：允许的最大父节点数。

输出

- GES：Record 字典。
- ExactSearch：GeneralGraph（最优 DAG）。
- BOSS/GRaSP：GeneralGraph（CPDAG）。

人工检查

- scorefunc 是否匹配数据类型（线性/非线性、连续/离散）：核心人工门。
- lambdavalue / maxP 是否合理（控制稀疏与复杂度）。
- GES 贪心结果是否需要用 ExactSearch 复核全局最优。

### 4. 基于约束的因果发现

从观测数据出发，用条件独立性检验（conditional independence test）反复检验变量间是否独立，先学出无向骨架（skeleton），再按 v-结构（collider）和 Meek 规则定向，得到表示因果结构的 CPDAG（PC）或允许潜在混杂的 PAG（FCI），或处理非平稳/异质数据的增广骨架（CDNOD）。

证据：source-reviewed

输入

- data：numpy ndarray，shape=(nsamples, nfeatures)，观测数据矩阵。
- alpha：显著性水平，默认 0.05。
- indeptest：条件独立性检验方法名，fisherz/chisq/gsq/kci 等（见 causal-learn-能力-条件独立性检验）。

输出

- PC/CDNOD：CausalGraph 对象。
- FCI：GeneralGraph 对象（PAG），含 endpoint 矩阵、ambiguoustriples、underlinetriples。

人工检查

- CI 检验方法是否匹配数据类型（线性/非线性、连续/离散）：核心人工门。
- causal sufficiency 是否成立（决定用 PC 还是 FCI）。
- alpha 是否合理（过小漏真因果边，过大引入假边）。

### 5. 时间序列因果发现

给定多维时间序列，判断一个变量的过去值是否对另一个变量的当前值有因果（预测性或结构性）影响。Granger 用"加入 X 的滞后是否显著降低 Y 的预测残差"做假设检验；VAR-LiNGAM 用非高斯性把 VAR 残差分解成同期结构因果，区分 X→Y（同期）和 X 的过去→Y（滞后）。

证据：source-reviewed

输入

- Granger：
- VARLiNGAM：

输出

- Granger：pvaluematrix（逐滞后阶的 p 值矩阵）、adjmatrix（逐滞后阶的二值邻接矩阵）。
- VAR-LiNGAM：adjacencymatrices（同期因果 B + 滞后 AR 系数）、causalorder、TimeseriesBootstrapResult（bootstrap 稳定性）。

人工检查

- 时间序列是否平稳（决定 Granger 可用性）：核心人工门。
- VAR-LiNGAM 残差是否满足非高斯（决定同期因果可识别性）。
- 滞后阶 maxlag/lags 是否合理（BIC 选择后仍需人工确认）。

### 6. 条件独立性检验

为约束类因果发现（PC/FCI/CDNOD）和函数因果模型方向判定（ANM/PNL）提供统一的条件独立性检验入口：给定变量集 X、Y 和条件集 Z，返回 X⫫Y|Z 是否成立的统计判断（pvalue 和布尔结果），并支持多种检验方法（参数/非参数/核/离散），带结果缓存避免重复计算。

证据：source-reviewed

输入

- data：numpy ndarray，shape=(nsamples, nfeatures)，完整观测数据矩阵（初始化 CIT 时传入一次）。
- method：检验方法名，'fisherz'/'mvfisherz'/'mcfisherz'/'kci'/'fastkci'/'rcit'/'chisq'/'gsq' 之一。
- Xs、Ys：要检验独立性的变量索引列表。

输出

- pvalue（float）：条件独立性检验的 p 值。
- CIT 实例可被 pc/fci/cdnod 直接作为 indeptest 参数传入（传方法名字符串或 CIT 对象）。
- 缓存的中间对象（相关矩阵、核矩阵）。

人工检查

- method 是否匹配数据类型（线性连续→fisherz、非线性→kci/rcit、离散→chisq/gsq、含缺失→mvfisherz）：核心人工门。
- alpha 阈值是否合理。
- 大数据用 kci 是否需要降采样或改用 rcit/fastkci。

## 处理机制

1. **核心处理链**：从观测数据中学习因果结构，并比较多类因果发现算法。

## 开始方式

1. 先明确数据类型和因果假设
2. 从一种算法和小样本开始
3. 比较图结构与领域知识

## 环境要求

- Python
- 适合的观测数据
- 因果发现基础

## 关键限制

- 本轮未安装运行
- 算法依赖不同统计假设
- 输出需要领域和稳健性复核

## 已核对范围

- 静态读取 causallearn/graph/（GeneralGraph、Dag、Edge、Endpoint、GraphClass、GraphUtils、SHD、AdjacencyConfusion、ArrowConfusion）
- 静态读取 causallearn/utils/DAG2CPDAG.py、DAG2PAG.py、PDAG2DAG.py、TXT2GeneralGraph.py
- 静态读取 causallearn/score/LocalScoreFunction.py（6 类评分）、LocalScoreFunctionClass.py
- 静态读取 causallearn/utils/GraphUtils.py
- 静态读取 causallearn/search/FCMBased/lingam/（directlingam、icalingam、varlingam、varmalingam、rcd、CAMUV、multigroupdirectlingam、longitudinallingam、causaleffect、bottomupparcelingam）
- 静态读取 causallearn/search/FCMBased/ANM/ANM.py
- 静态读取 causallearn/search/FCMBased/PNL/PNL.py
- 静态读取 causallearn/search/FCMBased/lingam/hsic.py、hsic2.py（HSIC 独立性检验）

## 仍待核对

- 未运行图转换（dag2cpdag/pdag2dag），未验证转换输出
- 未运行 SHD 评估，未验证指标计算
- 未验证 LocalScoreFunction 各类的实际评分值
- 未运行任何 LiNGAM/ANM/PNL，未验证邻接矩阵和因果顺序输出
- 未验证 ICA-LiNGAM 与 Direct-LiNGAM 在同一数据上的差异
- 未验证 VAR-LiNGAM 的时间序列滞后因果输出
- 未运行 ges/exactsearch/boss/grasp，未验证实际评分搜索输出
- 未验证 BIC 与 BDeu 在离散数据上的评分差异

## 许可边界

本页的能力分析、结构与说明由 bo老师整理，采用 CC BY 4.0。上游项目、代码、名称和商标继续遵循各自许可。这里不重新发布上游代码。
