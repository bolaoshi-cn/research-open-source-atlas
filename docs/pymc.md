# PyMC

用代码声明贝叶斯概率模型，通过MCMC或变分推断估计参数后验分布。

适合需要在不确定条件下估计参数、比较模型或做预测的研究者。先声明先验和似然，再用采样器推断参数后验，最后做模型诊断和预测检查。

- 官网详情：https://bolaoshi.cn/research-tools/pymc
- 原始来源：https://github.com/pymc-devs/pymc
- 读取版本：c870b5c3f88b30ac6db6211711f12c6e7c2d6ddc
- 核对日期：2026-07-10
- GitHub Stars：9,674（2026-07-11）
- 上游许可：Apache-2.0
- 验证程度：catalogued
- 编辑判断：recommended

## 什么时候使用

- 需要用后验分布表达参数不确定性而非单一点估计
- 观测数据较少，希望引入先验知识约束参数估计
- 需要比较多个竞争模型的预测能力（WAIC/LOO）
- 想通过先验预测检查确认模型在未观测数据下的行为是否合理

## 先准备什么

- 观测数据（x, y），缺失值可让PyMC自动填补
- 对待估参数的先验分布形式及其超参数
- 模型结构假设——哪些是参数、哪些是似然、哪些是确定性关系
- 选择采样策略：MCMC精确但慢，变分推断快速但是近似

## 它会怎样推进

1. **声明模型**：用with块定义参数先验、似然和数据容器，指定维度坐标
2. **选择推断方法**：默认NUTS连续采样，离散变量用Metropolis，大数据用ADVI变分推断
3. **运行采样**：设置链数、采样量、目标接受率，监控收敛诊断（R-hat、ESS）
4. **检查后验**：查看摘要表（均值/HDI/ESS/R-hat），画轨迹图和后验密度图
5. **模型验证**：做后验预测检查（PPC），确认模型能生成与观测数据特征相似的样本
6. **预测新场景**：用set_data替换输入，或do操作做反事实干预推演

## 第一次这样开始

帮我安装这个库：https://github.com/pymc-devs/pymc
安装前先告诉我需要什么环境，安装后帮我跑通一个最小示例。

## 结果出来后先看什么

- R-hat是否接近1.0（<1.01），ESS是否足够大
- 发散（divergence）数是否接近0——大量发散说明后验几何不良
- 先验预测是否覆盖了合理的数据范围
- 后验预测能否重现观测数据的关键统计量

## 常见误区

- 不检查收敛诊断就把采样结果当作可信后验
- 在强相关参数上使用MeanField ADVI——它假设参数独立，会低估方差
- 直接用MAP点估计代表后验——多维/非对称后验的众数不具有代表性
- 模型比较时只看WAIC/LOO的数值而不看ELPD差的标准误

## 能力拆解

### 1. MCMC后验采样

给定一个已声明的 pymc 概率模型，用 MCMC（主要是 NUTS，辅以 HMC、Metropolis、Slice 等步骤方法）从后验分布抽取样本，产出带收敛诊断（R-hat、ESS、发散、树深度）的 InferenceData/DataTree 对象，让研究者据此做后验推断和预测。

证据：source-reviewed

输入

- 已声明的 pm.Model 对象（见 pymc-能力-概率模型构建）。
- draws（每条链的样本数，默认 1000）、tune（调谐样本数，默认 1000，丢弃）、chains（链数）、cores（并行核数）。
- step（步骤方法，不指定时由 sample 自动按 RV 离散/连续性选择）。

输出

- InferenceData/DataTree（默认）：含 posterior（后验样本，chain×draw×vardims）、samplestats（diverging、acceptancerate、treedepth、tune 标记）、observeddata、可选 loglikelihood。
- MultiTrace（returninferencedata=False）：原始多链 trace（旧式，功能少）。
- 收敛诊断 SamplerWarning 列表（R-hat、ESS、发散、树深度警告）。

人工检查

- 后验是否收敛（R-hat<1.01、ESS 足够、无大量发散）。
- targetaccept/init/参数化是否针对后验几何调优。
- 离散/连续混合模型的 CompoundStep 顺序是否合理。

### 2. 先验后验预测采样

给定一个已声明的 pymc 模型（和/或其后验样本），从前验预测分布或后验预测分布生成前向样本，用于先验合理性检查（数据未观测前的先验预测是否合理）、后验预测检查（模型能否重现观测数据的特征）和预测（在新数据/新坐标下基于后验做情景预测与因果干预推演）。

证据：source-reviewed

输入

- 已声明的 pm.Model（见 pymc-能力-概率模型构建）。
- 后验样本 trace/idata（来自 pm.sample 或 pm.fit，供后验预测条件化）。
- varnames：要重新生成的变量名（默认观测 RV）。

输出

- InferenceData 的 prior/priorpredictive（先验预测）、posteriorpredictive（PPC）、predictions（外推预测）组。
- 经 extendinferencedata 并入原 idata 的预测样本。
- pm.draw 的 numpy 数组（快速抽样）。

人工检查

- 先验预测检查是否通过（先验预测覆盖合理数据范围，不过宽/过窄）。
- 后验预测检查是否能重现观测数据的关键统计量（均值、方差、尾部分布）。
- 预测新数据时 setdata 的形状/坐标是否正确。

### 3. 变分推断

给定一个已声明的 pymc 概率模型，用变分推断（VI）快速估计后验的近似分布，用一个小批量大样本数据下可扩展的优化过程替代 MCMC 采样，产出 Approximation 对象（可从中抽取近似后验样本），适用于大数据、快速探索或作为 NUTS 的初始化。

证据：source-reviewed

输入

- 已声明的 pm.Model 对象（见 pymc-能力-概率模型构建）。
- n（迭代次数，默认 10000）、method（"advi" / "fullrankadvi" / "svgd" / "asvgd"）。
- start / startsigma（ADVI 起始均值与标准差）、randomseed、infkwargs、backend（"numba"/"c"/"jax"）。

输出

- Approximation 对象（MeanField/FullRank/Empirical），可 .sample(n) 抽近似后验样本。
- .sample(n) 产出的 InferenceData（结构与 MCMC 的 posterior 组兼容，可复用下游分析与预测）。
- ELBO/损失追踪（经 callbacks），用于诊断收敛。

人工检查

- 近似族（MeanField vs FullRank）是否匹配后验几何。
- ELBO 是否收敛、近似后验是否与 NUTS 抽样对照（重要推断应交叉验证）。
- 小批量 ADVI 的 batchsize 是否足够估梯度。

### 4. 后验分析与诊断

给定 pymc 采样产出的后验 InferenceData，做后验摘要（均值、标准差、HDI、分位数）、收敛诊断（R-hat、ESS、发散、树深度）、模型比较（WAIC、LOO）和后验报告/绘图，让研究者判断模型是否可信、参数后验如何解释、多个候选模型哪个更优，并产出可进入论文或报告的统计结果。

证据：source-reviewed

输入

- 后验样本 idata（InferenceData/DataTree，来自 pm.sample 或 pm.fit().sample()）。
- varnames（要摘要/诊断的变量）、hdiprob（HDI 概率，默认 0.94）。
- 模型比较所需：loglikelihood 组（经 computeloglikelihood 在采样后填入 idata）、多个候选模型的 idata。

输出

- 后验摘要表（mean/sd/HDI/MCSE/ESS/R-hat）。
- 收敛诊断（SamplerWarning：R-hat、ESS、发散、树深度）。
- 信息准则结果（ELPDData：WAIC/LOO 的 ELPD、peff、SE）。

人工检查

- 收敛诊断是否全部通过（R-hat<1.01、ESS 足够、无大量发散）。
- 报告时是否同时给出后验均值、HDI、ESS、R-hat（而非只 mean）。
- 模型比较时是否报告 ELPD 差的 SE（差 < 2SE 视为无显著差异）。

### 5. 概率模型构建

给定一个贝叶斯模型的问题结构（参数、先验、似然、维度关系），用 with pm.Model() 上下文声明一个完整的概率图模型，让后续的采样、推断和预测都能引用同一套随机变量、确定性变量、势函数和坐标，并支持先验/似然声明、维度命名、数据容器、条件化干预和缺失值自动插补。

证据：source-reviewed

输入

- 模型定义代码（with pm.Model() as model: 块内）。
- 随机变量声明：分布类调用 pm.Normal("x", mu=0, sigma=1)，或 .dist() 创建无名称 RV。
- 观测数据：observed= 关键字把 RV 变成似然节点；或 pm.Data("x", array, dims=...) 注册可变数据容器。

输出

- Model 对象，可被 pm.sample、pm.fit、pm.samplepriorpredictive、pm.sampleposteriorpredictive 引用。
- RV/Deterministic/Potential 节点的枚举（model.basicRVs 等）供采样器、logp 计算和报告引用。
- 经 observe/do 产出的新模型对象（生成式建模、因果干预、情景预测）。

人工检查

- 先验选择是否合理（避免过度 informative 或完全无信息先验导致采样困难）。
- coords/dims 命名是否与后续报告规范一致。
- observed 数据是否经过清洗（缺失值 pymc 可自动插补，但插补机制需研究者知情）。

## 处理机制

1. **核心处理链**：用代码声明贝叶斯概率模型，通过MCMC或变分推断估计参数后验分布。

## 开始方式

1. 声明模型：用with块定义参数先验、似然和数据容器，指定维度坐标
2. 选择推断方法：默认NUTS连续采样，离散变量用Metropolis，大数据用ADVI变分推断
3. 运行采样：设置链数、采样量、目标接受率，监控收敛诊断（R-hat、ESS）
4. 检查后验：查看摘要表（均值/HDI/ESS/R-hat），画轨迹图和后验密度图
5. 模型验证：做后验预测检查（PPC），确认模型能生成与观测数据特征相似的样本
6. 预测新场景：用set_data替换输入，或do操作做反事实干预推演

## 环境要求

- 观测数据（x, y），缺失值可让PyMC自动填补
- 对待估参数的先验分布形式及其超参数
- 模型结构假设——哪些是参数、哪些是似然、哪些是确定性关系
- 选择采样策略：MCMC精确但慢，变分推断快速但是近似

## 关键限制

- 不检查收敛诊断就把采样结果当作可信后验
- 在强相关参数上使用MeanField ADVI——它假设参数独立，会低估方差
- 直接用MAP点估计代表后验——多维/非对称后验的众数不具有代表性
- 模型比较时只看WAIC/LOO的数值而不看ELPD差的标准误

## 已核对范围

- 读取 pymc/sampling/mcmc.py 的 sample()、initnuts()、sampleexternalnuts、samplemany、sample
- 读取 pymc/stepmethods/ 的 NUTS、HamiltonianMC、Metropolis、BinaryMetropolis、BinaryGibbsMetropolis、CategoricalGibbsMetropolis、DEMetropolis、DEMetropolisZ、Slice、CompoundStep、BlockedStep
- 读取 pymc/stats/convergence.py 的 runconvergencechecks、SamplerWarning、warndivergence、warntreedepth
- 读取 pymc/tuning/starting.py 的 findMAP、pymc/tuning/scaling.py 的 mass matrix 辅助函数
- 读取 pymc/backends/ 的 NDArray、MultiTrace、toinferencedata
- 读取 pymc/sampling/forward.py 的 draw、samplepriorpredictive、sampleposteriorpredictive
- 读取 sampleposteriorpredictive 的 samplevars/freezevars/predictions/extendinferencedata 机制
- 读取 README.rst 线性回归样例的 priorpredictive - do/observe - posteriorpredictive - predictions 链路

## 仍待核对

- 未运行本机 pm.sample smoke
- 未逐个核对 NUTS 内部 leapfrog/树构建/对偶平均的数值细节
- 未覆盖外部采样器（nutpie/numpyro/blackjax）的安装与调用实测
- 未运行本机 prior/posterior predictive smoke
- 未核对 ImplicitFreezeWarning 的触发与 freezevars/samplevars 的完整交互
- 未覆盖 pm.Data + setdata 在预测新坐标时的完整字段变化
- 未运行本机 pm.fit smoke
- 未核对 opvi/operators 的算子（KL、KS）内部实现

## 许可边界

本页的能力分析、结构与说明由 bo老师整理，采用 CC BY 4.0。上游项目、代码、名称和商标继续遵循各自许可。这里不重新发布上游代码。
