# statsmodels

用经典统计推断方法做回归、时间序列、面板分析和假设检验。

面向需要R风格统计输出（系数表、标准误、p值、诊断图）的Python研究者。用公式接口建模，产出可发表级别的统计摘要表和残差诊断。

- 官网详情：https://bolaoshi.cn/research-tools/statsmodels
- 原始来源：https://github.com/statsmodels/statsmodels
- 读取版本：43066730ab0daa5f1586dc0dbd0a8ca7b79ec456
- 核对日期：2026-07-10
- GitHub Stars：11,503（2026-07-11）
- 上游许可：BSD-3-Clause
- 验证程度：catalogued
- 编辑判断：recommended

## 什么时候使用

- 需要R风格的统计摘要表（系数、SE、p值、置信区间）和残差诊断图
- 经济/社会科学研究中的面板数据回归和时间序列分析
- 需要对大量假设同时做多重比较校正（Bonferroni、FDR等）
- 需要状态空间模型做结构时间序列分解（趋势/季节/周期成分）

## 先准备什么

- 整理好的数据表格（pandas DataFrame），标注清楚因变量和自变量
- 模型公式（如 y ~ x1 + x2）或用矩阵指定endog/exog
- 对数据的基本理解：哪些是连续/分类变量、有无缺失值、有无组结构

## 它会怎样推进

1. **指定模型**：用公式或矩阵接口定义因变量、自变量、模型族（OLS/GLM/MixedLM等）
2. **拟合模型**：调用fit方法，输出系数估计、标准误、p值和置信区间
3. **诊断模型**：检查残差分布、异方差、影响点、VIF共线性等诊断指标
4. **假设检验**：对线性约束做Wald/LR检验，或对多组比较做多重校正
5. **预测与解释**：生成预测值和置信区间，整理成可发表的统计表格

## 第一次这样开始

帮我安装这个库：https://github.com/statsmodels/statsmodels
安装前先告诉我需要什么环境，安装后帮我跑通一个最小示例。

## 结果出来后先看什么

- 残差是否满足模型假设（正态性、同方差、独立性）
- VIF值是否低于常用的共线性阈值
- 多重比较校正后的显著性是否与校正前差异很大
- 时间序列的残差是否为白噪声（Ljung-Box检验）

## 常见误区

- 用OLS拟合面板数据而不考虑个体异质性——应使用MixedLM或GEE
- 在非平稳时间序列上直接跑回归——单位根检验（ADF/KPSS）不通过时需差分
- 对几十个假设不加多重比较校正就逐个做显著性推断——会严重高估显著发现
- 直接用summary表中的p值做因果结论——回归系数只有模型假设成立时才有因果含义

## 能力拆解

### 1. 假设检验与多重比较

给定一组样本、列联表或已拟合的模型，回答"组间/变量间是否存在差异或关联"，产出检验统计量、p 值、效应量、置信区间，并对成组比较或多次检验做多重比较校正，同时给出样本量与功效的正向与反向计算，让研究者把研究问题映射到正确的检验并完整报告。

证据：source-reviewed

输入

- 两组或多组样本数组（t-test、z-test、ANOVA）。
- 频数/计数（proportion 检验、卡方拟合优度、列联表）。
- 已拟合的 OLS/GLM Results 对象（anovalm 做模型比较或 Type I/II/III 平方和分解）。

输出

- 检验结果元组或对象（统计量、p 值、自由度、置信区间）。
- ANOVA 表（sumsq、df、F、PR(F)）。
- multipletests 四元组（reject、pvalscorrected、alphacSidak、alphacBonf）。

人工检查

- 选检验是否匹配数据类型（连续/计数/比例/分类）和设计（独立/配对/分层/重复测量）。
- ANOVA 的 typ 选择是否匹配不平衡设计与模型项。
- 多重比较方法是 FWER 还是 FDR，是否匹配领域惯例。

### 2. 时间序列与状态空间建模

给定一个或多个时间序列，识别其平稳性、阶数和协整关系，用状态空间框架（SARIMAX、UCM、动态因子、VARMAX）或经典框架（ARIMA、VAR、VECM）拟合，产出可预测、可诊断、可参数恢复的时序模型，并支持滤波、平滑、预测、冲击响应和方差分解。

证据：source-reviewed

输入

- 单变量或多变量时间序列（pandas Series/DataFrame，带 DatetimeIndex 或 PeriodIndex）。
- 模型阶数：order=(p,d,q)、seasonalorder=(P,D,Q,s)、order（VAR）、trend、exog（外生变量 / ARIMAX / SARIMAX）。
- 平稳性检验输入：序列、maxlag、regression（常数/趋势）、autolag。

输出

- MLEResults/VARResults 对象（params、aic/bic/hqic、llf、filterresults、smoothresults）。
- 预测与预测置信带（getforecast 返回 mean + confint）。
- 诊断图（plotdiagnostics：standardized resid、hist+KDE、QQ、correlogram）。

人工检查

- 差分阶数 d/D、季节周期 s、滞后阶数是否匹配领域结构。
- MLE 不收敛时是否换 startparams、简化阶数或换 method。
- 预测区间是否覆盖不确定性来源（参数 + 创新）。

### 3. 线性回归与诊断

给定一个被解释变量和设计矩阵（或 R 风格公式），拟合并诊断线性回归模型，产出可解释的系数、推断、稳健协方差、设定检验和影响点，让研究者判断模型是否满足高斯–马尔可夫假设、哪些观测在驱动结论。

证据：source-reviewed

输入

- endog：一维被解释变量数组或 pandas Series。
- exog：设计矩阵（不含常数，需 sm.addconstant 显式加截距）。
- 或 R 风格公式 formula + data（DataFrame），经 patsy 解析成设计矩阵。

输出

- RegressionResults 对象（params、bse、tvalues、pvalues、confint、rsquared、adj-rsquared、fvalue、fpvalue、aic、bic、mseresid）。
- 稳健结果对象（替换 SE/t/p）。
- 诊断检验结果元组（统计量、p 值、自由度或 F 值）。

人工检查

- 诊断检验违背后是否换稳健协方差、换模型族（GLS/RLM）或换设定（加变量、变换）。
- 影响点是否剔除、加权还是保留并解释。
- HAC 的 maxlag、聚类的 groups 是否匹配真实数据结构。

### 4. 面板与混合效应模型

给定纵向、聚类或分层（面板）数据，在考虑组内相关和分层方差结构的前提下拟合模型，产出固定效应估计、随机效应或工作相关结构下的推断，让研究者处理重复测量、聚类设计和非独立观测，而不会把组内相关误当独立样本。

证据：source-reviewed

输入

- endog：结果变量（连续、二项、计数、有序、名义）。
- exog：固定效应设计矩阵或公式。
- 分组键 groups（GEE 必须）、vcformula/reformula（MixedLM 随机效应）。

输出

- MixedLMResults（feparams、covre、vcomp、randomeffects dict、bsefe、bsere、optimretvals、llf/aic/bic）。
- GEEResults（params、covrobust、covnaive、bse、workingcorrelation、score test 结果）。
- BayesMixedGLM 结果（femean、fesd、vcpmean、vcpsd、vcpmean、random effects 后验摘要）。

人工检查

- 随机效应结构（哪些变量在哪个层级有斜率）是否匹配实验/抽样设计。
- GEE 的 covstruct 是否反映真实相关结构。
- 方差成分接近零或协方差非正定时，是否简化随机效应。

## 处理机制

1. **核心处理链**：用经典统计推断方法做回归、时间序列、面板分析和假设检验。

## 开始方式

1. 指定模型：用公式或矩阵接口定义因变量、自变量、模型族（OLS/GLM/MixedLM等）
2. 拟合模型：调用fit方法，输出系数估计、标准误、p值和置信区间
3. 诊断模型：检查残差分布、异方差、影响点、VIF共线性等诊断指标
4. 假设检验：对线性约束做Wald/LR检验，或对多组比较做多重校正
5. 预测与解释：生成预测值和置信区间，整理成可发表的统计表格

## 环境要求

- 整理好的数据表格（pandas DataFrame），标注清楚因变量和自变量
- 模型公式（如 y ~ x1 + x2）或用矩阵指定endog/exog
- 对数据的基本理解：哪些是连续/分类变量、有无缺失值、有无组结构

## 关键限制

- 用OLS拟合面板数据而不考虑个体异质性——应使用MixedLM或GEE
- 在非平稳时间序列上直接跑回归——单位根检验（ADF/KPSS）不通过时需差分
- 对几十个假设不加多重比较校正就逐个做显著性推断——会严重高估显著发现
- 直接用summary表中的p值做因果结论——回归系数只有模型假设成立时才有因果含义

## 已核对范围

- 读取 statsmodels/stats/anova.py（anovalm、AnovaRM 类在 anova 模块）
- 读取 statsmodels/stats/weightstats.py（ttestind、ttostind、ttostpaired、ztest、zconfint）
- 读取 statsmodels/stats/proportion.py（proportionsztest、proportionschisquare、powerproportions2indep）
- 读取 statsmodels/stats/multitest.py（multipletests、fdrcorrection、fdrcorrectiontwostage）
- 读取 statsmodels/stats/multicomp.py（MultiComparison、TukeyHSD）存在
- 读取 statsmodels/stats/power.py（TTestPower、TTestIndPower、NormalIndPower、FTestPower、FTestAnovaPower、GofChisquarePower 的 solvepower）
- 读取 statsmodels/stats/contingencytables.py（Table、SquareTable、Table2x2、StratifiedTable）
- 确认 examples/notebooks/interactionsanova.ipynb、chi2fitting.ipynb 存在

## 仍待核对

- 未运行本机 t-test/ANOVA/multipletests smoke
- 未核对 contingency tables 的精确检验分支
- 未覆盖非参数检验（Wilcoxon/Mann-Whitney 在 scipy，statsmodels 部分在 sandbox）
- 未运行本机 SARIMAX/VAR/单位根 smoke
- 未逐一核对 Kalman filter/smoother 的数值实现
- 未覆盖 Markov switching、ETS、Holt-Winters、exponential smoothing 的内部细节
- 未运行本机 OLS 拟合 smoke
- 未逐个验证每个诊断检验的统计口径

## 许可边界

本页的能力分析、结构与说明由 bo老师整理，采用 CC BY 4.0。上游项目、代码、名称和商标继续遵循各自许可。这里不重新发布上游代码。
