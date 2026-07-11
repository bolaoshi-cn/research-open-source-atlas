# causalml

用机器学习估计个体处理效应并做uplift建模与策略优化

面向需要估计异质性因果效应和做个性化干预决策的研究者和数据科学家。提供元学习器、因果树/森林、uplift树、深度模型、工具变量、倾向性得分匹配和策略优化等统一接口。

- 官网详情：https://bolaoshi.cn/research-tools/causalml
- 原始来源：https://github.com/uber/causalml
- 读取版本：78176f86a594369dcc953fce8a3b6f3dbefa7040
- 核对日期：2026-07-10
- GitHub Stars：5,920（2026-07-11）
- 上游许可：Apache-2.0
- 验证程度：catalogued
- 编辑判断：worth-watching

## 什么时候使用

- 需要估计哪些个体对干预的反应更大（CATE/Uplift）时
- 需要为营销、定价或个性化推荐选择最优处理策略时
- 需要评估和比较多个CATE模型的排序质量（AUUC/Qini）时
- 需要用工具变量方法处理内生性问题时

## 先准备什么

- 准备特征矩阵X、处理指示向量（0/1）、结局变量y
- 如果是观察数据，准备倾向性得分估计；如果是实验数据则可省略
- 根据数据规模选择基模型：小数据用线性模型，大数据用XGBoost/LightGBM
- 确认无混杂假设是否成立（工具变量法还需确认工具外生性和排他约束）

## 它会怎样推进

1. **选择学习器**：根据数据和目标选S/T/X/R/DR learner或因果树/森林
2. **估计CATE**：用fit/predict接口估计个体处理效应
3. **评估模型**：用AUUC/Qini曲线和敏感性分析评估CATE质量
4. **优化策略**：基于CATE和成本收益学习最优处理分配策略

## 第一次这样开始

帮我安装这个库：https://github.com/uber/causalml
安装前先告诉我需要什么环境，安装后帮我跑通一个最小示例。

## 结果出来后先看什么

- 元学习器的ATE置信区间是否包含零（效应是否显著）
- AUUC/Qini曲线是否明显优于随机分配基线
- 敏感性分析的Placebo和Random Cause检验是否通过
- 倾向性得分分布是否在处理组和对照组间有足够重叠

## 常见误区

- 对观察数据使用元学习器时不检查无混杂假设是否成立
- 混淆AUUC排序（相对评估）和绝对因果效应（需要识别假设）
- R-learner/DR-learner依赖的倾向性得分估计不准导致CATE偏差
- 用Granger因果替代结构性因果解释

## 能力拆解

### 1. uplift树分类与排序

对二元结局的场景，用基于 uplift（增量响应）分裂准则的决策树和随机森林，直接从特征 X、处理指示 T、二元结局 y 学到每个样本的 uplift 分数（处理组转化率 − 对照组转化率），并支持树可视化与森林排序。

证据：source-reviewed

输入

- X：特征矩阵（np.array，uplift 树不接受 pandas 原生，需 np.array）。
- treatment：处理向量，含 controlname 与一个或多个处理组名。
- y：二元结局（0/1）。

输出

- predict 返回各处理组 uplift 分数（shape [nsamples, ntreatmentgroups]）与推荐处理组。
- 森林 predict(fulloutput=True) 返回更详细输出（含每棵树预测）。
- 树结构：fitteduplifttree（DecisionTree 实例，可被 uplifttreeplot 可视化）。

人工检查

- 分裂准则选择（KL/ED/Chi/CTS/DDP）是否符合业务目标。
- 树深度与叶节点最小样本是否合理（uplift 树易过拟合）。
- 多处理组下各准则的稳定性。

### 2. uplift评估曲线与AUUC

在真实 CATE 不可得的观察/实验数据上，用 uplift 曲线（gain/lift/qini）、AUUC、Qini 分数、Treatment Operating Characteristic（TOC）和基于 DR 伪结果的评分对多个 CATE 模型做排序与比较；并用 Placebo、Random Cause、Subset Data、Random Replace、Selection Bias 等敏感性方法检验估计对未观测混淆的稳健性。

证据：source-reviewed

输入

- 评估 DataFrame（df）：列为 outcomecol（实际结局）、treatmentcol（处理指示）、treatmenteffectcol（真实 tau，可选）和多个模型预测列（CATE/uplift 估计）。
- outcomecol（默认 'y'）、treatmentcol（默认 'w'）、treatmenteffectcol（默认 'tau'）。
- normalize（默认 True，gain 曲线归一化）、tmle（默认 False，是否用 TMLE 版曲线）。

输出

- auucscore/qiniscore：每模型列的标量分数（Series）。
- plot：matplotlib Axes（gain/lift/qini 曲线）。
- getcumgain/getcumlift/getqini：累积统计 DataFrame。

人工检查

- AUUC/Qini 排序是否符合业务目标（Qini 对处理/对照比例更公平）。
- 是否有真实 tau（实验数据）以做绝对偏差评估。
- 敏感性方法选择是否覆盖主要威胁（Placebo 查假信号、Random Cause 查冗余特征、Subset Data 查样本依赖、Selection Bias 查未观测混淆）。

### 3. 倾向性得分估计与匹配

从特征 X 和二元处理 w 估计每个样本接受处理的概率（倾向性得分 p），并提供 isotonic 校准、边界裁剪和交叉验证；在倾向性得分基础上做最近邻匹配（PSM）生成平衡样本，并用 table one 与 SMD 报告匹配前后协变量平衡。

证据：source-reviewed

输入

- 倾向性模型：
- 匹配（match.py）：

输出

- fitpredict 返回 p：np.ndarray，shape [nsamples]，裁剪后倾向性得分 ∈ (clipbounds[0], clipbounds[1])。
- computepropensityscore 返回 (p, pmodel)：得分数组与拟合模型。
- createtableone 返回 pd.DataFrame：各组各特征均值（标准差）与 SMD。

人工检查

- 倾向性模型选择（ElasticNet 线性 vs GBM 非线性）是否符合处理分配机制。
- clipbounds 是否对 IPW/R-loss 合适（过窄会放大权重）。
- PSM 匹配后 SMD 是否全部 < 0.1（常规平衡阈值）。

### 4. 元学习器CATE估计

从实验或观察数据中的特征 X、处理指示 T、结局 Y 出发，用一组"元学习器"（S/T/X/R/DR/TMLE）把任意可替换的基模型组合成条件平均处理效应（CATE）估计，并给出 ATE、置信区间和特征重要性/SHAP 解释。

证据：source-reviewed

输入

- X：特征矩阵（np.array / pd.DataFrame / polars DataFrame / LazyFrame）。
- treatment：处理向量，取值为 controlname 或一个/多个处理组名。
- y：结局向量（回归为连续，分类为二元）。

输出

- predict 返回 te：np.ndarray，shape [nsamples, ntreatmentgroups]，CATE 估计。
- predict(returncomponents=True) 额外返回 yhatcs、yhatts（各处理组的反事实预测）。
- estimateate 返回：

人工检查

- learner 类型选择是否符合数据结构（连续/二元 outcome、单/多处理、观察/实验）。
- 基模型选择（线性 vs 树 vs 神经网络）是否匹配效应异质性复杂度。
- 倾向性得分模型是否充分（观察数据下倾向性估计偏差会污染 R/X/DR learner）。

### 5. 因果树与森林

用基于处理效应的分裂准则（causalmse）的决策树和随机森林，直接从特征 X、二元处理 w、结局 y 学到 CATE 的树结构估计，并提供叶节点级效应、树可视化与基于 forestci 的森林置信区间。

证据：source-reviewed

输入

- X：特征矩阵（np.array / pd.DataFrame）。
- treatment：二元处理向量（0/1）。
- y：连续结局向量。

输出

- predict 返回 te：np.ndarray，shape [nsamples]，叶节点级 CATE。
- 树结构：tree、featureimportances。
- 森林：estimators（树列表）、estimatorsamples、可选 CI（通过 forestci）。

人工检查

- criterion 选择（causalmse vs standardmse）是否符合分析目标。
- groupspenalty 调参：过高会欠拟合异质性，过低会出现叶节点单组主导。
- 叶节点最小样本是否足够支撑稳定的效应估计。

### 6. 处理优化与策略学习

在已有 CATE 估计或结局概率的基础上，把"处理效应估计"转化为"可执行的处理分配策略"：学习最优处理分配策略（PolicyLearner）、基于反事实逻辑选目标单元（CounterfactualUnitSelector）、在多处理+成本约束下算反事实价值（CounterfactualValueEstimator）、计算概率必然性边界（PNS bounds）。

证据：source-reviewed

输入

- outcomelearner（默认 GradientBoostingRegressor）。
- treatmentlearner（默认 GradientBoostingClassifier）。
- policylearner（默认 DecisionTreeClassifier，需接受 sampleweight）。

输出

- PolicyLearner predict：处理分配（0/1）数组；predictvalue：策略价值估计。
- CounterfactualUnitSelector：最优目标单元标识。
- CounterfactualValueEstimator predictbest：最优处理索引；predictall：各处理价值。

人工检查

- 策略学习的 DR 伪结果是否依赖正确的结局/倾向模型。
- 四类依从者收益是否符合业务目标。
- 价值优化的成本与价值估计是否准确（影响分配）。

## 处理机制

1. **核心处理链**：用机器学习估计个体处理效应并做uplift建模与策略优化

## 开始方式

1. 选择学习器：根据数据和目标选S/T/X/R/DR learner或因果树/森林
2. 估计CATE：用fit/predict接口估计个体处理效应
3. 评估模型：用AUUC/Qini曲线和敏感性分析评估CATE质量
4. 优化策略：基于CATE和成本收益学习最优处理分配策略

## 环境要求

- 准备特征矩阵X、处理指示向量（0/1）、结局变量y
- 如果是观察数据，准备倾向性得分估计；如果是实验数据则可省略
- 根据数据规模选择基模型：小数据用线性模型，大数据用XGBoost/LightGBM
- 确认无混杂假设是否成立（工具变量法还需确认工具外生性和排他约束）

## 关键限制

- 对观察数据使用元学习器时不检查无混杂假设是否成立
- 混淆AUUC排序（相对评估）和绝对因果效应（需要识别假设）
- R-learner/DR-learner依赖的倾向性得分估计不准导致CATE偏差
- 用Granger因果替代结构性因果解释

## 已核对范围

- 静态读取 inference/tree/uplift.pyx 中 UpliftTreeClassifier、UpliftRandomForestClassifier、DecisionTree 的类与方法签名
- 静态读取 inference/tree/init.py 的 uplift 导出与序列化补丁
- 静态读取 inference/tree/plot.py 与 utils.py 的导出
- 静态读取 metrics/init.py 全部导出
- 静态读取 metrics/visualize.py 的 plot/getcumlift/getcumgain/getqini/auucscore/qiniscore 签名与机制
- 静态读取 metrics/rate.py 的 gettoc/ratescore/plottoc
- 静态读取 metrics/catescoring.py 的 computedrpseudooutcomes/drscore/plugintscore
- 静态读取 metrics/sensitivity.py 的 Sensitivity 类与 SensitivitySelectionBias 等

## 仍待核对

- 未运行 UpliftTreeClassifier/UpliftRandomForestClassifier，未验证 uplift 分数与分裂实际行为
- 未读取 Cython uplift.pyx 内部分裂准则（如 KL/ED/Chi/CTS）的完整实现，仅静态确认类签名
- 未运行任何评估函数，未验证 AUUC/Qini/TOC 数值与曲线绘制
- 未运行敏感性分析，未验证 Selection Bias / Random Cause 等方法的实际输出
- 未运行任何倾向性模型，未验证校准后的倾向性得分数值
- 未运行 PSM 匹配，未验证匹配后平衡表（table one）实际效果
- 未运行任何 learner.fit/estimateate，未验证任一 learner 实际 CATE/ATE 数值输出
- 未验证 bootstrap CI、多处理组、effectmodifiers 分层估计的实际行为

## 许可边界

本页的能力分析、结构与说明由 bo老师整理，采用 CC BY 4.0。上游项目、代码、名称和商标继续遵循各自许可。这里不重新发布上游代码。
