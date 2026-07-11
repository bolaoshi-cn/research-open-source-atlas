# pgmpy

用图结构编码变量间的概率依赖和因果假设，做条件推断、因果发现与合成数据生成。

适合需要从数据中学变量关系图结构、或基于已知因果图做干预推断的研究者。从定义网络和条件概率表开始，逐步完成推理、验证和模拟。

- 官网详情：https://bolaoshi.cn/research-tools/pgmpy
- 原始来源：https://github.com/pgmpy/pgmpy
- 读取版本：c7fa866a8b50a7eb781124e12411708dd86f2c25
- 核对日期：2026-07-10
- GitHub Stars：3,295（2026-07-11）
- 上游许可：MIT
- 验证程度：catalogued
- 编辑判断：recommended

## 什么时候使用

- 需要从观测数据中学变量间的因果或依赖关系图
- 已有因果图，想知道干预某个变量后另一个变量的分布变化
- 需要批量生成符合已知联合分布的合成数据做方法验证
- 需要对比两个因果图结构与真实图的差距

## 先准备什么

- 观测数据表格（pandas DataFrame），列名与变量名一致
- 若手写因果图，需准备边列表和条件概率表的参数估计
- 若做因果推断，需明确哪些变量是干预变量、哪些是结果变量

## 它会怎样推进

1. **建立图结构**：定义变量节点和边方向（手写或从数据中学），构建有向无环图
2. **填充条件概率**：从数据估计或手写每个节点的条件概率分布（离散/线性高斯/函数式）
3. **校验模型**：检查概率表归一化、父子节点对应关系，确保模型可推
4. **查询推理**：在给定证据下查询后验概率，或用do操作计算干预分布
5. **生成合成数据**：从模型前向采样，生成用于方法验证或敏感性分析的模拟数据

## 第一次这样开始

帮我安装这个库：https://github.com/pgmpy/pgmpy
安装前先告诉我需要什么环境，安装后帮我跑通一个最小示例。

## 结果出来后先看什么

- 模型校验是否通过（概率归一、父子对应）
- 学到的图结构是否与领域知识大致吻合
- 干预查询结果与观测条件概率是否有实质性差异
- 合成数据的联合分布是否符合预期的条件依赖模式

## 常见误区

- 把观测相关当因果效应——只有do操作才能区分两者
- 把学到的图结构直接当真因果图——PC等方法学到的是等价类，部分方向不可识别
- 用离散独立性检验处理连续数据——需匹配数据类型选chi_square或fisher_z
- check_model通过就认为图方向正确——它只校验图和CPD的对应关系

## 能力拆解

### 1. 参数学习

给定一个已确定图结构的贝叶斯网络（结构可手写或由结构学习得到）和一份观测数据，估计每个节点的条件概率分布（CPD）参数，使模型能用于推理、预测和模拟。支持离散贝叶斯网络的 MLE/Bayesian/EM 估计与线性高斯贝叶斯网络的 MLE 估计，并能处理含隐变量或缺失数据的情况。

证据：source-reviewed

输入

- 已确定结构的模型对象（DiscreteBayesianNetwork / LinearGaussianBayesianNetwork，含边但 CPD 未填或待更新）。
- 观测数据：pandas.DataFrame，列名与节点名一致。
- 估计器选择：estimator="MLE"（默认）或 BayesianEstimator。

输出

- 填充 CPD 的模型对象（bn.getcpds() 返回 TabularCPD/LinearGaussianCPD 列表）。
- 可传入 pgmpy-能力-概率推理 做后验推断、pgmpy-能力-数据模拟 生成合成数据、pgmpy-能力-因果推断与do演算 做干预查询。

人工检查

- 估计器选择（MLE vs Bayesian vs EM）是否匹配样本量与缺失情况。
- Bayesian 先验类型与 pseudocount 是否反映合理领域信念。
- EM 收敛是否检查（对数似然轨迹、maxiter 是否触及）。

### 2. 因果推断与do演算

给定一个因果图（DAG，方向编码因果假设）和已填充的 CPD，计算干预分布 P(Y | do(X))：即"如果人为设定 X 而非仅观测 X，Y 的分布会怎样"。这区别于条件概率 P(Y | X)，需要先用后门/前门准则识别调整集，再做 do 操作（移除干预变量的入边）和调整后的概率推断，让研究者能区分观测相关与因果效应。

证据：source-reviewed

输入

- 因果图（DiscreteBayesianNetwork / LinearGaussianBayesianNetwork / SEMGraph / DAG），边的方向编码因果假设。
- 已填充的 CPD（离散）或线性高斯参数。
- variables：查询目标变量（Y）。

输出

- 干预分布 DiscreteFactor（P(Y | do(X), evidence)）。
- 调整集识别结果（Adjustment.identify 返回 (调整图, 是否存在)）。
- mutilated DAG（do 操作后的图）。

人工检查

- 因果图方向是否由领域知识/结构学习+专家定向确定（不能凭数据相关定向）。
- 后门/前门调整集识别是否成功，不可识别时是否报告。
- variant 选择（minimal vs all vs minimalvariance）是否匹配估计目标。

### 3. 数据模拟

给定一个已填充 CPD 的贝叶斯网络，生成符合该网络联合分布的合成数据，并支持在生成时施加干预（do）、证据（evidence）、软证据（virtualevidence）、软干预（virtualintervention）、缺失机制（missingprob）和部分固定样本（partialsamples），让研究者能生成可复现的合成数据用于结构学习/参数学习的方法验证、敏感性分析和因果干预模拟。

证据：source-reviewed

输入

- 已填充 CPD 的模型对象（DiscreteBayesianNetwork/LinearGaussianBayesianNetwork/FunctionalBayesianNetwork）。
- nsamples：要生成的样本数。
- do：硬干预 {var: state}（生成时强制设定，切断父节点影响）。

输出

- pandas.DataFrame（合成数据），可直接用于结构学习/参数学习的方法验证、敏感性分析。
- 干预模拟的样本可用于验证 pgmpy-能力-因果推断与do演算 的 do 查询结果。
- 含缺失的样本（returnfull=True）可用于测试 EM 参数学习。

人工检查

- do 干预模拟的设定是否符合研究问题的因果干预场景。
- missingprob 的 MAR/MNAR 假设是否匹配真实缺失机制。
- 采样方法（forward/rejection/likelihoodweighted/Gibbs）是否匹配 evidence 与精度要求。

### 4. 概率推理

给定一个已填充 CPD 的贝叶斯网络，计算后验条件概率 P(query | evidence)、最大后验（MAP）、边际分布或极大边际，让研究者能在给定观测证据下做概率查询。支持精确推理（变量消元、联结树置信传播）与近似推理（采样近似、Gibbs 采样）。

证据：source-reviewed

输入

- 已填充 CPD 的模型对象（DiscreteBayesianNetwork 或 DiscreteMarkovNetwork，含完整 CPD）。
- query：要查询的变量列表。
- evidence：观测证据 {var: state} 字典。

输出

- DiscreteFactor（后验条件分布 P(query | evidence)）。
- MAP 状态字典（后验最大的状态组合）。
- 极大边际（maxmarginal）。

人工检查

- 精确（VE/BP）vs 近似（采样）的选择是否匹配网络规模与精度要求。
- 消元顺序是否合理（影响性能）。
- 采样的 nsamples/burnin/seed 是否报告，收敛是否检查。

### 5. 结构学习

给定一份观测数据（离散或连续），从数据中学习贝叶斯网络的有向无环图结构（因果发现），输出满足条件独立性或评分最优的 DAG，让研究者不必手写图结构就能用数据驱动地得到因果假设候选，并支持注入专家知识约束边的方向。

证据：source-reviewed

输入

- 观测数据：pandas.DataFrame，列名为变量名，离散/连续。
- citest（PC 用）："chisquare"、"gsq"、"loglikelihood"、"pearsonr"、"powerdivergence" 等（离散）或 fisherz（连续）。
- scoringmethod（评分搜索用）："bic-g"（高斯 BIC）、"bdeu"、"k2"、"aic" 等。

输出

- 学到的 DAG（DiscreteBayesianNetwork 或 networkx.DiGraph），可传入 pgmpy-能力-参数学习 学 CPD、pgmpy-能力-概率推理 做推断、pgmpy-能力-因果推断与do演算 做因果查询。
- PDAG/CPDAG 等价类（PC returntype=pdag），部分边方向不定。
- 结构评分值（可对比模型）。

人工检查

- 学到的因果方向是否与领域知识一致；等价类内不可识别方向是否需专家定向。
- CI 检验/评分方法是否匹配数据类型与样本量。
- significancelevel/maxindegree 是否按领域调整。

### 6. 结构验证与模型检测

给定一个学到的/假设的因果图结构（或多个候选图）和真实图/数据，评估该结构与真实图或数据的契合程度：结构层面（SHD 结构汉明距离、邻接/定向混淆矩阵）和数据层面（图隐含的条件独立性是否被数据支持、相关性评分、Fisher-C 评分、BIC/AIC 结构评分），并支持 d-分离、模型一致性（checkmodel）等图级检测，让研究者能判断学到的因果结构是否可靠、哪个候选图更优。

证据：source-reviewed

输入

- 结构对比类（需真实图）：truecausalgraph（DAG/networkx）、estcausalgraph（学到的 DAG）。
- 数据契合类（需数据）：X（DataFrame）、causalgraph（待验证的图）、citest（用于隐含 CI 检验）。
- 评分类：data + scoringmethod（AIC/BIC/K2/BDeu 等）。

输出

- SHD 距离值（标量）。
- 邻接/定向混淆矩阵（TP/FP/FN/TN + precision/recall/F1）。
- ImpliedCIs 检验结果（图隐含 CI 的支持/违背统计）。

人工检查

- 结构对比（SHD/混淆矩阵）还是数据契合（ImpliedCIs/评分）更匹配验证目标。
- SHD 的 edgereversepenalty 是否合理反映定向错误严重性。
- citest 是否匹配数据类型。

## 处理机制

1. **核心处理链**：用图结构编码变量间的概率依赖和因果假设，做条件推断、因果发现与合成数据生成。

## 开始方式

1. 建立图结构：定义变量节点和边方向（手写或从数据中学），构建有向无环图
2. 填充条件概率：从数据估计或手写每个节点的条件概率分布（离散/线性高斯/函数式）
3. 校验模型：检查概率表归一化、父子节点对应关系，确保模型可推
4. 查询推理：在给定证据下查询后验概率，或用do操作计算干预分布
5. 生成合成数据：从模型前向采样，生成用于方法验证或敏感性分析的模拟数据

## 环境要求

- 观测数据表格（pandas DataFrame），列名与变量名一致
- 若手写因果图，需准备边列表和条件概率表的参数估计
- 若做因果推断，需明确哪些变量是干预变量、哪些是结果变量

## 关键限制

- 把观测相关当因果效应——只有do操作才能区分两者
- 把学到的图结构直接当真因果图——PC等方法学到的是等价类，部分方向不可识别
- 用离散独立性检验处理连续数据——需匹配数据类型选chi_square或fisher_z
- check_model通过就认为图方向正确——它只校验图和CPD的对应关系

## 已核对范围

- 读取 pgmpy/estimators/MLE.py（MaximumLikelihoodEstimator getparameters/estimatecpd 签名）
- 读取 pgmpy/estimators/BayesianEstimator.py（BayesianEstimator getparameters/estimatecpd 签名）
- 读取 pgmpy/estimators/EM.py（ExpectationMaximization）
- 读取 pgmpy/parameterestimator/（discretebayesian/discretemle/discreteem/lineargaussianmle）
- 读取 DiscreteBayesianNetwork.py:592 fit / :653 fitupdate 签名
- 读取 LinearGaussianBayesianNetwork.py:880 fit 签名
- 读取 README Parameter Learning 特性与 example notebooks 主题
- 读取 pgmpy/inference/CausalInference.py（CausalInference 类、query(query, do, evidence, adjustmentset, inferencealgo) 签名）

## 仍待核对

- 未运行本机参数学习 smoke
- 未验证 BayesianEstimator 各先验类型（BDeu/Dirichlet/K2）的数值差异
- 未覆盖 SEMEstimator/IVEstimator 的 SEM 参数估计内部机制（见待复核卡）
- 未运行本机因果查询 smoke
- 未验证 minimal/all/minimalvariance 三种调整集变体的数值差异
- 未覆盖 DoubleMLRegressor/NaiveAdjustmentRegressor/NaiveIVRegressor（因果效应估计的预测回归器，见待复核卡）
- 未运行本机模拟 smoke
- 未验证 missingprob 的 MAR/MNAR 缺失机制实现细节

## 许可边界

本页的能力分析、结构与说明由 bo老师整理，采用 CC BY 4.0。上游项目、代码、名称和商标继续遵循各自许可。这里不重新发布上游代码。
