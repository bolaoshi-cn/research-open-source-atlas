# CausalPy

用统一接口完成常见准实验因果估计、诊断和报告。

PyMC Labs 维护的准实验因果推断库，连接识别策略、反事实构造、不确定性量化和稳健性诊断。

- 官网详情：https://bolaoshi.cn/research-tools/causalpy
- 原始来源：https://github.com/pymc-labs/CausalPy
- 读取版本：642f177d345bd65ad739e8a425ddbdf9faf6e2b1
- 核对日期：2026-07-10
- 上游许可：Apache-2.0
- 验证程度：catalogued
- 编辑判断：recommended

## 适合这些任务

- 使用 DiD、合成控制或断点回归的研究者
- 需要贝叶斯不确定性报告的用户

## 这些情况要谨慎

- 尚未明确识别策略的研究任务
- 只想快速得到单个效应数字的用户

## 能力拆解

### 1. 准实验因果效应估计

给定一个准实验设计（阈值规则、政策冲击、交错采用、地理对照、工具变量或无法随机化的选择场景）和对应的观测面板或时间序列数据，估计干预的因果效应，产出反事实预测、因果 impact（observed − counterfactual）和带不确定性的效应估计，让研究者把"识别策略"映射到正确的准实验方法。

证据：source-reviewed

输入

- DataFrame（面板或时间序列，index 强制改名为 obsind）。
- patsy 公式（formula），描述 outcome 与 predictors/交互项关系（DiD/RD/RKink/ITS/IPW）。
- 设计参数（因方法而异）：treatmentthreshold（RD）、treatmenttime/treatmentendtime（ITS/SC）、controlunits/treatedunits（SC/SDiD）、instrumentsformula/instrumentsdata（IV）、weightingscheme（IPW）、unitvariablename/timevariablename/treatmenttimevariablename（Staggered DiD）、bandwidth/donuthole（RD）、groupvariablename/pretreatmentvariablename（PrePostNEGD）。

输出

- causalimpact（DiD/RD：标量后验 DataArray chain×draw；ITS/SC：postimpact obsind×chain×draw）。
- ypredcounterfactual/ypredcontrol/ypredtreatment（预测 dict，贝叶斯含 posteriorpredictive.mu）。
- discontinuityatthreshold（RD）、gradientchange（RKink）。

人工检查

- 识别策略是否匹配设计：阈值规则→RD、政策冲击→ITS/DiD、交错采用→Staggered DiD、无法随机化→IV/IPW。
- 平行趋势假设（DiD/Staggered DiD）：pre-treatment 趋势是否平行。
- 外生性 + 排除限制（IV）：工具变量是否与内生解释变量相关、与误差项不相关、不直接影响结果。

### 2. 双后端模型拟合

给定一个准实验方法（声明了 supportsbayes/supportsols）和一个模型实例（PyMC 贝叶斯、sklearn OLS 或 None），统一构造一个可 fit/predict/score/calculateimpact 的后端适配器，让同一个实验类的 algorithm() 不感知后端差异，并在 None 时给出合理默认（总是贝叶斯）。

证据：source-reviewed

输入

- model（PyMCModel | RegressorMixin | None）：模型实例或 None。
- defaultmodelclass（实验类属性，如 LinearRegression、WeightedSumFitter、PropensityScore）：None 时的默认类。
- supportsbayes / supportsols（实验类属性）：后端支持声明。

输出

- fitted self.modelbackend（含 InferenceData 或 sklearn 系数）。
- self.model（公开句柄，同 backend）。
- predict(newx)（贝叶斯返回含 posteriorpredictive.mu 的 dict；OLS 返回 numpy）。

人工检查

- 后端选择是否匹配不确定性需求：需要后验/不确定性传播 → 贝叶斯；只要点估计 → OLS。
- samplekwargs 是否足够（draws/chains/targetaccept）保证收敛，需检查 rhat/divergences。
- None 默认总是贝叶斯，研究者是否意识到这会触发 MCMC（耗时）。

### 3. 可复现pipeline编排

给定输入数据、实验配置和一组步骤（估计效应、敏感性分析、生成报告），编排一个可复现的因果推断工作流：在任意拟合开始前先验证所有步骤配置，把每步产出累积到共享上下文，最终产出包含效应摘要、诊断结果和图表的自包含 HTML 报告，让分析从"一次性脚本"变成"可复查、可交接的产物链"。

证据：source-reviewed

输入

- data（pd.DataFrame）。
- experimentconfig（dict：method class + kwargs，供 EstimateEffect 构造实验）。
- 步骤列表（EstimateEffect、SensitivityAnalysis(checks=[...])、GenerateReport(includeplots, includeeffectsummary, outputfile)）。

输出

- PipelineResult（experiment/effectsummary/sensitivityresults/report）。
- HTML 报告（str，自包含，可选写入 outputfile）。
- 诊断图（随 sensitivityresults 的 CheckResult.figures）。

人工检查

- 步骤顺序是否合理（EstimateEffect 必须先于 SensitivityAnalysis 和 GenerateReport）。
- includesensitivity 是否开启（默认 False，诊断不进报告）。
- HTML 报告是否包含足够信息供复查（效应摘要 + 诊断图 + 假设）。

### 4. 效应摘要与不确定性量化

给定已拟合实验的因果 impact 后验（贝叶斯）或点估计 + 残差（OLS），产出决策就绪的效应摘要：点估计（mean/median）、不确定性区间（贝叶斯 HDI / OLS t-CI）、方向概率（P(effect0)）、实际显著性（ROPE）、相对效应（% vs counterfactual）、累积效应，并附多段自然语言 prose 报告，让研究者直接用于决策和沟通。

证据：source-reviewed

输入

- causalimpact（DiD/RD：标量后验 chain×draw）或 postimpact（ITS/SC：obsind×chain×draw）或 gradientchange（RKink）。
- postpred（反事实预测，用于相对效应分母）。
- 报告参数：direction（increase/decrease/two-sided）、alpha（默认 0.05 → 95% HDI/CI）、cumulative（默认 True，ITS/SC 累积效应）、relative（默认 True，% 变化）、mineffect（ROPE 阈值，仅贝叶斯）、window（"post"/tuple/slice，ITS/SC 时间窗）、treatedunit（多单元实验）、period（intervention/post/comparison，三阶段 ITS）、prefix（prose 前缀）。

输出

- EffectSummary.table（pd.DataFrame：行 average/cumulative，列 mean/median/hdilower/hdiupper/pgt0/plt0/ptwosided/probofeffect/prope/relativemean/relativehdilower/relativehdiupper）。
- EffectSummary.text（多段 prose：观察 vs 反事实 + 累积 + 可信度 + 假设建议）。
- Staggered DiD 额外：atteventtime 表（含 identified 列）。

人工检查

- alpha/hdiprob 是否匹配领域惯例（0.94 是 ArviZ 默认，不是 0.95）。
- direction 是否匹配研究假设（预期增加→increase，预期减少→decrease，无方向→two-sided）。
- mineffect（ROPE 阈值）是否有领域依据，不能任意设。

### 5. 稳健性诊断检查

给定一个已拟合的准实验实验对象，运行可插拔的稳健性/敏感性诊断检查套件（安慰剂检验、留一法、密度检验、凸包检验、先验敏感性、持续性检验等），每项检查产出 pass/fail 判定 + 诊断统计表 + prose + 图表，让研究者快速判断因果结论是否稳健，是否依赖特定观测、特定先验或特定 donor。

证据：source-reviewed

输入

- 已拟合实验对象（BaseExperiment 子类实例）。
- PipelineContext（提供 experimentconfig、data，供诊断派生实验工厂）。
- 各检查自有参数（如 PlaceboInTime 的 nfolds、fold 选择策略、assurance 的 expected-effect prior；LeaveOneOut 的留出单元；McCrary 的 bin 数；PriorSensitivity 的先验列表）。

输出

- CheckResult(checkname, passed, table, text, figures, metadata)（每项检查一个）。
- SensitivitySummary(results, allpassed, text)（聚合）。
- 诊断图（matplotlib figures，随 CheckResult.figures）。

人工检查

- 哪些检查适用于当前设计（RD→McCrary/Bandwidth，SC→LeaveOneOut/ConvexHull/PlaceboInSpace，ITS→PlaceboInTime，贝叶斯→PriorSensitivity）。
- PlaceboInTime 的 fold 数和选择策略是否足够（太少→不可靠，太多→计算贵）。
- assurance 的 expected-effect prior 是否有领域依据。

## 处理机制

1. **核心处理链**：用统一接口完成常见准实验因果估计、诊断和报告。

## 开始方式

1. 先写清识别策略和估计对象
2. 选择官方示例数据跑通一种方法
3. 检查诊断和效应摘要

## 环境要求

- Python
- PyMC 或 scikit-learn 后端
- 准实验研究设计

## 关键限制

- 本轮未完成本机运行
- 方法适用条件差异很大
- 诊断通过也不能代替识别判断

## 已核对范围

- 读取 causalpy/experiments/diffindiff.py（DifferenceInDifferences：algorithm、反事实构造、输入校验、effectsummary）
- 读取 causalpy/experiments/syntheticcontrol.py（SyntheticControl：treatmenttime、control/treated units、mindonorcorrelation）
- 读取 causalpy/experiments/regressiondiscontinuity.py（RegressionDiscontinuity：treatmentthreshold、epsilon、bandwidth、donuthole）
- 读取 causalpy/experiments/interruptedtimeseries.py（InterruptedTimeSeries：双/三阶段、treatmentendtime）
- 读取 causalpy/experiments/instrumentalvariable.py（InstrumentalVariable：两阶段、vspriortype、binarytreatment）
- 读取 causalpy/experiments/inversepropensityweighting.py（InversePropensityWeighting：propensity 模型、weightingscheme）
- 读取 causalpy/experiments/staggereddid.py（StaggeredDifferenceInDifferences：BJS 2024 imputation 估计量）
- 读取 ARCHITECTURE.md 实验清单表（12 个实验类 + 后端支持矩阵）

## 仍待核对

- 未运行本机 PyMC smoke
- 未深读 PrePostNEGD、RegressionKink、PiecewiseITS、PanelRegression、SyntheticDifferenceInDifferences 的算法细节
- 未核对每个实验的 OLS fallback 分支
- 未深读 pymcmodels.py 各模型类的 buildmodel 实现（LinearRegression/WeightedSumFitter/InstrumentalVariableRegression/PropensityScore/BayesianBasisExpansionTimeSeries/StateSpaceTimeSeries）
- 未深读 sklmodels.py createcausalpycompatibleclass 的 patch 细节
- 未运行本机双后端对比 smoke
- 未深读 Pipeline.run 的验证前置逻辑（"all steps validated before any fitting"）
- 未深读 GenerateReport 的 HTML 渲染细节

## 许可边界

本页的能力分析、结构与说明由 bo老师整理，采用 CC BY 4.0。上游项目、代码、名称和商标继续遵循各自许可。这里不重新发布上游代码。
