# pgmpy

构建概率图模型，并完成结构学习、推理、因果干预和数据模拟。

覆盖贝叶斯网络、结构学习、参数学习、概率推理、do 演算和模型验证的 Python 图模型工具栈。

- 官网详情：https://bolaoshi.cn/research-tools/pgmpy
- 原始来源：https://github.com/pgmpy/pgmpy
- 读取版本：c7fa866a8b50a7eb781124e12411708dd86f2c25
- 核对日期：2026-07-10
- 上游许可：MIT
- 验证程度：catalogued
- 编辑判断：recommended

## 适合这些任务

- 需要贝叶斯网络和概率推理的研究者
- 研究图结构与因果干预的开发者

## 这些情况要谨慎

- 把结构学习结果直接解释成因果事实的任务
- 缺少概率图模型基础的快速分析

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

1. **核心处理链**：构建概率图模型，并完成结构学习、推理、因果干预和数据模拟。

## 开始方式

1. 先选择离散或连续图模型
2. 用官方小样例定义结构和 CPD
3. 再增加推理、学习或干预

## 环境要求

- Python
- 概率图模型基础
- 结构或数据输入

## 关键限制

- 本轮未完成本机运行
- 不同模型和估计器依赖差异大
- 结构与因果解释需要额外假设

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
