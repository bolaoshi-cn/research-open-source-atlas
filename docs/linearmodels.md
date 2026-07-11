# linearmodels

Python面板回归与工具变量估计库，处理内生性和多维面板数据

适合需要在Python中做面板数据回归、工具变量估计或因子定价的实证研究者。提供固定效应、随机效应、2SLS、LIML、GMM等估计器，配套稳健协方差和诊断检验。

- 官网详情：https://bolaoshi.cn/research-tools/linearmodels
- 原始来源：https://github.com/bashtage/linearmodels
- 读取版本：b535f5546081c27b4ff826ff03f326ffe379c58c
- 核对日期：2026-07-10
- GitHub Stars：1,052（2026-07-11）
- 上游许可：BSD-3-Clause
- 验证程度：catalogued
- 编辑判断：recommended

## 什么时候使用

- 需要做面板固定效应回归，控制不可观测的个体和时间异质性
- 需要用工具变量（2SLS、LIML、GMM）处理内生解释变量
- 需要做Fama-MacBeth两步估计或线性因子资产定价检验
- 需要在高维固定效应（如个体×企业×年份）下做吸收回归

## 先准备什么

- 整理好带实体-时间双索引的面板DataFrame，确保索引唯一
- 确定工具变量的外生性和相关性（理论论证+第一阶段F值）
- 明确面板效应的维度设置（个体效应、时间效应或双向）
- 准备好协方差类型选择依据（聚类、HAC或稳健标准误）

## 它会怎样推进

1. **构造模型**：根据研究设计选择对应模型类并传入数据和效应设定
2. **拟合估计**：调用.fit()选择协方差类型，系统自动完成去效应和系数估计
3. **诊断检验**：查看第一阶段F值、Wu-Hausman内生性检验、过度识别J统计量
4. **结果解读**：检查系数显著性、效应量和模型拟合优度
5. **报告输出**：用.summary()导出完整结果表，包含诊断检验统计量

## 第一次这样开始

帮我安装这个库：https://github.com/bashtage/linearmodels
安装前先告诉我需要什么环境，安装后帮我跑通一个最小示例。

## 结果出来后先看什么

- 检查第一阶段F值是否足够大（弱工具会严重偏倚2SLS结果）
- 确认Hausman检验是否支持用固定效应而非随机效应
- 验证选用的协方差类型（聚类/HAC/稳健）是否匹配数据的相关结构

## 常见误区

- 不做弱工具检验就直接用2SLS，弱工具下的2SLS偏倚可能比OLS更严重
- 在高维固定效应中把随时间不变的变量当作关心变量，它们会被吸收掉
- 用GMM的CUE作为默认估计器，CUE数值敏感且收敛不稳定，两步GMM通常够用

## 能力拆解

### 1. 工具变量估计

给定被解释变量、外生回归元、内生回归元和工具变量，估计线性工具变量模型：2SLS、LIML、k-class、GMM（含 continuously updating CUE）：并产出带异方差/自相关/聚类稳健协方差的系数、推断、弱工具检验和过度识别 J 统计量，让研究者判断工具变量是否有效、内生性是否被正确处理。

证据：source-reviewed

输入

- dependent：被解释变量（nobs×1）。
- exog：外生回归元（nobs×nexog）。
- endog：内生回归元（nobs×nendog）。

输出

- IVResults（params、stderrors、tstats、pvalues、confint、cov、resids、firstdiagnostics、wuhausman、sargan、limlkappa、summary）。
- IVGMMResults（额外 weightmat、weighttype、weightconfig、iteration、jstat）。
- OLSResults（纯外生退化路径）。

人工检查

- 内生性判断（Wu-Hausman）是否成立，是否真需要 IV。
- 工具变量外生性与相关性（第一阶段 F、Shea partial R²、理论论证）。
- 过度识别检验（Sargan/J）是否通过。

### 2. 系统回归与多方程

给定一组相互关联的方程（共享外生变量或误差项跨方程相关），联合估计系统：SUR/SURE（似乎不相关回归）、3SLS（三阶段最小二乘，含工具变量）、系统 GMM：利用跨方程误差协方差提升效率，并产出系统级系数、协方差、约束检验和系统 R²，让研究者判断联合估计是否优于单方程逐个估计。

证据：source-reviewed

输入

- 方程字典：{eqname: (dependent, exog)} 或 {eqname: dict(dependent=..., exog=...)}；每个方程可有独立 endog/instruments（IV3SLS）。
- 或公式字典：{eqname: 'y ~ x1 + x2'}，支持跨方程共享变量。
- sigma（可选预设跨方程残差协方差矩阵；若不给则估计）。

输出

- SystemResults（系统 params 堆叠、cov、sigma 跨方程协方差、tstats、pvalues、confint、systemr2、constraints、每方程子结果、summary）。
- GMM 系统：额外 weightmat、weighttype、iterations。
- 跨方程残差协方差矩阵 Σ（用于判断系统相关性强度）。

人工检查

- 是否真需要系统估计（Σ 是否显著非对角，跨方程误差是否相关）。
- 选 SUR vs 3SLS（有无内生性，需不需要工具）。
- 选 OLS vs GLS 路径（共享外生 + 无约束时 OLS 等价且更简）。

### 3. 线性因子资产定价

给定一组资产的超额收益（或总收益）和一组因子，估计线性因子定价模型：交易因子的时间序列回归、2/3 步截面估计、或 GMM 因子定价：并产出因子风险溢价、资产 β、定价误差（alpha）、J 过度识别检验和协方差，让研究者判断因子是否能解释资产截面收益差异、哪些资产被错误定价。

证据：source-reviewed

输入

- portfolios：资产组合收益矩阵（nobs×nport），通常为测试资产的超额收益。
- factors：因子矩阵（nobs×nf），可为交易因子（组合）或非交易因子（宏观）。
- riskfree（默认 True）：True 时用超额收益（截距吸收），False 时保留无风险利率（3-step 估计）。

输出

- LinearFactorModelResults/TradedFactorModelResults（betas、riskpremia λ、alphas 定价误差、cov、rsquared 时间序列拟合、summary）。
- GMMFactorModelResults（额外 jstat 过度识别、iterations、weight 信息）。
- J 统计量（GMM/3-step 的过度识别检验）。

人工检查

- 选 TradedFactorModel（交易因子，时间序列）vs LinearFactorModel（非交易因子，2/3-step）vs GMM（GMM 定价）。
- riskfree 设定是否与收益输入一致。
- GMM steps/usecue（2 步通常够；CUE 数值敏感）。

### 4. 面板数据回归

给定一个带实体-时间双索引的面板数据（或可转成 MultiIndex 的 DataFrame），估计面板回归模型：固定效应（含双向）、随机效应、between、first difference、pooled OLS 或 Fama-MacBeth：并产出带聚类/异方差/自相关稳健协方差的系数、推断和设定检验，让研究者判断不随时间变化的异质性是否被正确吸收。

证据：source-reviewed

输入

- dependent：一维被解释变量，pandas Series / DataFrame / numpy / xarray。
- exog：外生回归元矩阵；公式接口下由 formulaic 解析。
- 面板索引：(entity, time) MultiIndex（要求 entity-time 唯一）；PanelData 内部规范化为 (entity × time) 堆叠二维数组 + 实体/时间 id。

输出

- PanelEffectsResults（params、stderrors、tstats、pvalues、confint、cov、resids、fittedvalues、estimatedeffects、rsquaredwithin/between/overall、fstatistic、summary）。
- PanelResults（PooledOLS/BetweenOLS/FirstDifferenceOLS，无 estimatedeffects）。
- RandomEffectsResults（含 variancedecomposition、theta、smallsample 修正）。

人工检查

- 选固定效应还是随机效应（Hausman 检验）。
- 聚类维度是否匹配真实组内相关结构（企业、地区、年份）。
- HAC 的 bandwidth/kernel 是否匹配面板时间序列长度。

### 5. 高维吸收回归

给定被解释变量、关心的回归元和一组高维（可能是百万级）的固定效应或连续控制变量，用吸收法估计模型：Frish-Waugh-Lovell 正交化 + LSMR 稀疏迭代求解，避免构造或求逆高维虚拟变量矩阵：并产出关心变量的系数与协方差，让研究者在高维固定效应（如个体×企业×县×年）下仍能得到可推断的估计。

证据：source-reviewed

输入

- dependent：被解释变量（nobs×1）。
- exog：关心参数 β 对应的回归元（nobs×nexog）。
- absorb：高维效应，DataFrame（分类列作固定效应，连续列作连续控制）或 Interaction(cat=..., cont=...)。

输出

- AbsorbingLSResults（params、stderrors、tstats、pvalues、confint、cov、resids、rsquared、absorbed 诊断、summary）。
- 吸收项诊断（哪些 exog 列被完全吸收、冗余检测）。
- absorbedeffects（被吸收效应的隐含估计）。

人工检查

- 关心变量（exog）是否在理论上不会被吸收项完全吸收。
- 选 AbsorbingLS（高维吸收）vs PanelOLS（标准面板，≤2 维固定效应）：前者支持任意高维，后者更稳但限维度。
- LSMR 精度是否足够（与 LSDV 在子样本上对比）。

## 处理机制

1. **核心处理链**：Python面板回归与工具变量估计库，处理内生性和多维面板数据

## 开始方式

1. 构造模型：根据研究设计选择对应模型类并传入数据和效应设定
2. 拟合估计：调用.fit()选择协方差类型，系统自动完成去效应和系数估计
3. 诊断检验：查看第一阶段F值、Wu-Hausman内生性检验、过度识别J统计量
4. 结果解读：检查系数显著性、效应量和模型拟合优度
5. 报告输出：用.summary()导出完整结果表，包含诊断检验统计量

## 环境要求

- 整理好带实体-时间双索引的面板DataFrame，确保索引唯一
- 确定工具变量的外生性和相关性（理论论证+第一阶段F值）
- 明确面板效应的维度设置（个体效应、时间效应或双向）
- 准备好协方差类型选择依据（聚类、HAC或稳健标准误）

## 关键限制

- 不做弱工具检验就直接用2SLS，弱工具下的2SLS偏倚可能比OLS更严重
- 在高维固定效应中把随时间不变的变量当作关心变量，它们会被吸收掉
- 用GMM的CUE作为默认估计器，CUE数值敏感且收敛不稳定，两步GMM通常够用

## 已核对范围

- 读取 linearmodels/iv/model.py 的 IVModelBase、IVLSModelBase、IVLIML、IV2SLS、IVGMM、IVGMMCUE、OLS 类与 fit/estimateparameters/estimatekappa 方法
- 读取 linearmodels/iv/gmm.py 的 HomoskedasticWeightMatrix、HeteroskedasticWeightMatrix、KernelWeightMatrix、OneWayClusteredWeightMatrix、IVGMMCovariance
- 读取 linearmodels/iv/covariance.py、linearmodels/iv/data.py、linearmodels/iv/common.py（IVFormulaParser）
- 读取 README IV 段、doc/source/iv/、examples/ivbasic-examples.ipynb、examples/ivadvanced-examples.ipynb、examples/ivusing-formulas.ipynb
- 读取 linearmodels/system/model.py 的 SystemModelBase、LSSystemModelBase、IV3SLS、SUR、IVSystemGMM 类与 fit/fromformula/glsestimate/glsfinalize 方法
- 读取 linearmodels/system/covariance.py 的 HomoskedasticCovariance、HeteroskedasticCovariance、KernelCovariance、ClusteredCovariance、GMM 权重矩阵族
- 读取 linearmodels/system/gmm.py、linearmodels/system/results.py
- 读取 README System 段、doc/source/system/、examples/systemexamples.ipynb、examples/systemthree-stage-ls.ipynb、examples/systemformulas.ipynb

## 仍待核对

- 未运行本机 IV 拟合 smoke
- 未核对 GMM 迭代收敛判据在不同初值下的稳健性
- 未展开 CUE 数值优化的 solver 选择细节
- 未运行本机系统回归 smoke
- 未核对 GLS vs OLS 路径在 commonexog 时的等价性边界
- 未展开 IVSystemGMM 迭代权重矩阵的收敛判据细节
- 未运行本机因子定价 smoke
- 未核对 GMM 因子模型 steps2 迭代的收敛判据

## 许可边界

本页的能力分析、结构与说明由 bo老师整理，采用 CC BY 4.0。上游项目、代码、名称和商标继续遵循各自许可。这里不重新发布上游代码。
