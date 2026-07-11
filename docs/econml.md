# EconML

异质性处理效应估计库，用正交机器学习估计个体化因果效应

适合需要估计处理对不同个体差异化影响的研究者和数据科学家。它把任意sklearn模型用作干扰和最终阶段，通过交叉拟合去偏，给出带置信区间的个体化因果效应、策略推荐和SHAP解释。

- 官网详情：https://bolaoshi.cn/research-tools/econml
- 原始来源：https://github.com/py-why/EconML
- 读取版本：e546416862280c90f9b21e95c298d00d599c75c8
- 核对日期：2026-07-10
- GitHub Stars：4,708（2026-07-11）
- 上游许可：MIT
- 验证程度：partially-verified
- 编辑判断：recommended

## 什么时候使用

- 想估计处理对不同人群的差异化影响而不仅是平均效应
- 观察数据中存在大量控制变量需要非参数控制
- 希望得到个体化处理推荐（决策树/森林策略）
- 需要比较多个CATE模型的样本外表现并选择最佳模型

## 先准备什么

- 准备结果变量Y、处理变量T、异质性特征X和控制变量W
- 如用工具变量方法需额外准备工具变量Z
- 根据T的连续/离散类型选择对应估计器族
- 选定用作干扰模型的sklearn估计器和最终阶段模型

## 它会怎样推进

1. **选择估计器**：根据处理类型和研究目标选DML/DR/元学习器/IV/因果森林
2. **拟合CATE**：用交叉拟合去除干扰参数偏差，在X上学习异质性处理效应
3. **获取效应**：调用effect和effect_interval获取逐样本的CATE点估计和置信区间
4. **解释与策略**：用SHAP解释异质性来源，用决策树输出个体化处理推荐
5. **模型选择**：在验证集上用RScorer比较候选CATE模型，选最优或加权集成

## 第一次这样开始

帮我安装这个库：https://github.com/py-why/EconML
安装前先告诉我需要什么环境，安装后帮我跑通一个最小示例。

## 结果出来后先看什么

- 干扰模型（model_y/model_t）是否在交叉验证中有足够的拟合度
- 置信区间的宽度是否合理，bootstrap重采样次数是否足够
- 离散处理的baseline类别选择是否与实际问题一致
- 工具变量估计的有效性是否基于合理的外生性判断

## 常见误区

- 在离散处理场景忘记设置discrete_treatment=True
- 用元学习器时未把混淆变量W拼入特征矩阵X
- 把SHAP值解释为因果归因而非对CATE预测的贡献度量
- 在无有效工具变量的场景强行使用IV估计器

## 能力拆解

### 1. DML族CATE估计

用 Double Machine Learning（DML / R-Learner）框架，把任意 sklearn 模型同时用作 nuisance（E[Y|X,W]、E[T|X,W]）和 final stage（τ(X)），估计连续或离散处理下的异质性效应，并按 final-stage 类型（线性/debiased lasso/kernel/非参数/因果森林）给出对应置信区间。

证据：source-reviewed

输入

- modely：E[Y|X,W] 的 nuisance 模型（RegressorMixin 或 'auto'）。
- modelt：E[T|X,W] 的 nuisance 模型（RegressorMixin/ClassifierMixin 或 'auto'，离散 T 用分类器）。
- modelfinal：final-stage τ(X) 模型（连续 T 专用，DML/NonParamDML 用）。

输出

- effect/effectinterval/effectinference（继承 BaseCateEstimator）。
- 线性 final：coef(T)、intercept(T)、coefinterval、coefinference、summary()。
- CausalForestDML：shapvalues、featureimportances。

人工检查

- nuisance 模型（modely/modelt）是否充分拟合（DML 的弱 nuisance 风险）。
- final-stage 参数化是否匹配真实异质性（线性 vs 森林）。
- 离散 T 的 baseline 类别选择。

### 2. 元学习器CATE估计

用元学习器（S-Learner / T-Learner / X-Learner / Domain Adaptation Learner）把任意现成监督模型（如 GradientBoostingRegressor/Classifier）组合成离散处理的 CATE 估计器，无需显式 nuisance 残差化，适合低维处理、强基线模型或对正交学习框架不依赖的场景。

证据：source-reviewed

输入

- SLearner(overallmodel)：单一模型，把 T 当作普通特征。
- TLearner(models)：每处理臂一个模型。
- XLearner(models, propensitymodel, catemodels)：先 T-Learner 预测反事实，再用分类器学 imputation，最后按倾向性加权合并。

输出

- effect(X)、effectinterval（bootstrap）、effectinference（bootstrap）。

人工检查

- W 是否已正确拼入 X。
- 倾向性模型（XLearner/DA）是否充分。
- bootstrap 次数是否足够。

### 3. 双重稳健CATE估计

在离散处理（多处理臂 vs baseline）下，用双重稳健（Doubly Robust）校正处理选择偏差：第一阶段交叉拟合倾向性 pt(X,W)=Pr[T=t|X,W] 和回归 h(X,W,T)=E[Y|X,W,T]，构造 pseudo-outcome Y^DRt；第二阶段在 X 上拟合 final 模型得到 τt(X)。只要倾向性或回归 nuisance 之一正确指定，CATE 估计即一致。

证据：source-reviewed

输入

- modelpropensity：Pr[T|X,W] 分类器（默认 'auto'，折间选线性+森林最优）。
- modelregression：E[Y|X,W,T] 回归器（离散 outcome 时为分类器，默认 'auto'）。
- modelfinal：final-stage τt(X) 回归器。

输出

- effect(X, T0, T1)：任意两处理臂 contrast 的 CATE。
- effectinterval/effectinference（按 final-stage mixin 的 CI）。
- 线性 final：coef(T)/intercept(T)/coefinterval/coefinference/summary(T)（按处理臂）。

人工检查

- propensity 模型是否充分（多臂分类质量）。
- regression 模型是否充分（E[Y|X,W,T]）。
- baseline 处理臂选择。

### 4. 工具变量CATE估计

当存在未观测混淆、但存在有效工具变量 Z（影响 T、不直接影响 Y）时，用 IV 正交学习（OrthoIV / DMLIV / DRIV / IntentToTreat / SieveTSLS）估计连续或离散处理下的 CATE，通过交叉拟合 E[Y|X,W]、E[T|X,W]、E[Z|X,W]（和 IV-DR 的额外 nuisance）去除混淆和弱工具偏差，并给出置信区间。

证据：source-reviewed

输入

- Y、T、Z（工具变量）、X、W。
- discretetreatment、discreteinstrument（决定 nuisance 模型类型）。
- DML-IV（OrthoIV/DMLIV）：modelyxw、modeltxw、modelzxw、modeltxwz、modelfinal、projection（是否把 θ 投影到 Z 空间）。

输出

- effect/effectinterval/effectinference（按 final-stage mixin 的 CI；NonParamDMLIV 无解析 CI）。
- 线性 final：coef/intercept/coefinterval/summary()。

人工检查

- 工具变量是否满足外生性和相关性假设（库不验证）。
- 工具变量强度（弱工具风险）。
- discreteinstrument/discretetreatment 声明正确。

### 5. 正交机器学习CATE估计

在观察数据（或实验数据）上，用任意机器学习模型估计处理 T 对结果 Y 的条件平均处理效应 τ(X)=E[Y(1)−Y(0)|X]，同时用正交化（cross-fitting nuisance）控制高维混杂 W 的偏差，并尽量给出有效置信区间。这是 econml 全部 CATE 估计器的公共底座。

证据：source-reviewed

输入

- Y：(n, dy) 结果变量，可连续或离散（discreteoutcome）。
- T：(n, dt) 处理变量，可连续或离散（discretetreatment）。
- X：(n, dx) 异质性特征，CATE 随之变化。

输出

- effect(X, T0=0, T1=1)：(m,) 或 (m, dy, dt) CATE 点估计。
- effectinterval(...)：(lower, upper) 置信区间。
- effectinference(...)：InferenceResults，含 pointestimate、stderr、confint(alpha)、zstat、pvalue、summaryframe、populationsummary。

人工检查

- 是否满足无未观测混淆（X,W 控制住所有共同原因）或工具变量假设。
- nuisance 模型（modely/modelt）选择是否充分拟合 E[Y|X,W]、E[T|X,W]（DML 的弱 nuisance 会导致偏差）。
- final-stage 参数化（linear/sparse/nonparam/forest）是否匹配真实异质性形态。

### 6. 策略学习与CATE解释

在 CATE 估计之外，提供三类下游能力：(1) 直接从观察数据学习处理策略（不显式拟合 CATE），输出可解释的决策树/森林策略；(2) 把任意已拟合 CATE 模型解释为单棵决策树（CATE 树、策略树）或 SHAP 值，便于向决策者传达异质性来源；(3) 用 RScorer 做 CATE 模型选择与集成，比较多个异质估计器的样本外表现。

证据：source-reviewed

输入

- Policy Learning：Y、T、X、W、honest、maxdepth、minimpuritydecrease、mciters、cv（DRPolicyTree/DRPolicyForest 走 DR nuisance）。
- CATE 解释器：已拟合的 CATE estimator（est）、X、includemodeluncertainty、risklevel、sampletreatmentcosts、maxdepth、minsamplesleaf。
- SHAP：已拟合 CATE estimator、X、backgroundsamples。

输出

- 策略：predict(X) 推荐处理、predictvalue(X) 期望价值、predictproba(X) 概率、plot() 树图、featureimportances。
- 解释器：plot(featurenames=...) 树图（CATE 树或策略树）。
- SHAP：每特征对 CATE 的贡献数组 + shap.summaryplot。

人工检查

- 策略树深度与可解释性权衡（深度↑可解释性↓）。
- honest splitting 的样本充分性。
- RScorer 验证集独立性。

## 处理机制

1. **核心处理链**：异质性处理效应估计库，用正交机器学习估计个体化因果效应

## 开始方式

1. 选择估计器：根据处理类型和研究目标选DML/DR/元学习器/IV/因果森林
2. 拟合CATE：用交叉拟合去除干扰参数偏差，在X上学习异质性处理效应
3. 获取效应：调用effect和effect_interval获取逐样本的CATE点估计和置信区间
4. 解释与策略：用SHAP解释异质性来源，用决策树输出个体化处理推荐
5. 模型选择：在验证集上用RScorer比较候选CATE模型，选最优或加权集成

## 环境要求

- 准备结果变量Y、处理变量T、异质性特征X和控制变量W
- 如用工具变量方法需额外准备工具变量Z
- 根据T的连续/离散类型选择对应估计器族
- 选定用作干扰模型的sklearn估计器和最终阶段模型

## 关键限制

- 在离散处理场景忘记设置discrete_treatment=True
- 用元学习器时未把混淆变量W拼入特征矩阵X
- 把SHAP值解释为因果归因而非对CATE预测的贡献度量
- 在无有效工具变量的场景强行使用IV估计器

## 已核对范围

- 静态读取 dml/dml.py DML/LinearDML/SparseLinearDML/KernelDML/NonParamDML/BaseDML 类继承与 fit 构造
- 静态读取 dml/rlearner.py RLearner/ModelNuisance/ModelFinal 机制
- 静态读取 dml/causalforest.py CausalForestDML 继承与 BLB inference
- 静态读取 metalearners/metalearners.py SLearner/TLearner/XLearner/DomainAdaptationLearner 全部类
- 静态读取各类的 model 构造、fit、effect 与 bootstrap inference
- 静态读取 dr/drlearner.py DRLearner 及 ModelNuisance/ModelFinal/BaseDR
- 静态读取 DRLearner estimating equations（propensity + regression nuisance）
- 静态读取 LinearDRLearner/SparseLinearDRLearner/ForestDRLearner 的 final-stage mixin

## 仍待核对

- 未运行任一 DML 估计器，未验证 CATE、CI、coef 实际数值
- 未验证 final-stage 选择对推断的影响
- 未运行任一元学习器，未验证 CATE、bootstrap CI 实际数值
- 未验证 DomainAdaptationLearner 的倾向性加权 final 行为
- 未运行任一 DR 估计器，未验证 CATE、CI、coef 实际数值
- 未验证多处理臂（非 baseline）的 final multi-task 行为
- 未运行任一 IV 估计器，未验证 CATE、CI 实际数值
- 未验证 IntentToTreat（二元 Z、二元 T）的特殊 final 路径

## 许可边界

本页的能力分析、结构与说明由 bo老师整理，采用 CC BY 4.0。上游项目、代码、名称和商标继续遵循各自许可。这里不重新发布上游代码。
