# DoWhy

端到端因果推断库，用四步流水线统一因果建模、识别、估计与反驳

适合需要在观察数据上进行严谨因果推断的研究者和数据科学家。它用建模、识别、估计、反驳四步流水线统一因果图模型和潜在结果框架，并提供GCM模块支持异常归因和根因分析。

- 官网详情：https://bolaoshi.cn/research-tools/dowhy
- 原始来源：https://github.com/py-why/dowhy
- 读取版本：a53f80c7501aa81fbe7db261e1774246a8053699
- 核对日期：2026-07-10
- GitHub Stars：8,208（2026-07-11）
- 上游许可：MIT
- 验证程度：partially-verified
- 编辑判断：recommended

## 什么时候使用

- 有观察数据、有因果假设但无法做随机实验
- 需要从数据中估计某个处理对结果的因果效应大小
- 想检验因果结论是否对未观测混淆稳健
- 需要定位异常的根因或量化各因素对分布变化的贡献

## 先准备什么

- 准备包含处理、结果和协变量的表格数据
- 基于领域知识画出变量间的因果DAG（有向无环图）
- 选定estimand类型（ATE、CATE、ATE等）和估计方法

## 它会怎样推进

1. **因果图建模**：把处理、结果、混淆变量的因果假设编码为图模型和变量角色
2. **效应识别**：在图结构上判断效应是否可识别，找出有效的调整变量集合
3. **效应估计**：用指定统计方法从数据中估计因果效应大小并给出置信区间
4. **反驳检验**：用加随机混淆、安慰剂处理、数据子集等方法检验效应估计的稳定性
5. **敏感性分析**：量化未观测混淆需要多强才能推翻已有结论（E-value）

## 第一次这样开始

帮我安装这个库：https://github.com/py-why/dowhy
安装前先告诉我需要什么环境，安装后帮我跑通一个最小示例。

## 结果出来后先看什么

- 因果图的箭头方向是否符合领域知识而非数据驱动
- 识别阶段输出的调整变量集合是否排除本身就是处理结果的变量
- 反驳检验中新效应值是否偏离原估计过大
- E-value是否足够大于最强已观测混淆

## 常见误区

- 跳过因果图建模直接用变量声明跑识别和估计
- 把单一反驳方法通过等同于因果结论成立
- 忽略econml等可选依赖未安装时的降级行为

## 能力拆解

### 1. GCM因果机制建模与归因

不依赖经典 backdoor/iv 识别-估计范式，而是通过显式建模每个变量的因果数据生成机制（structural causal model），支持效应估计以外的因果任务：异常归因、根因分析、分布变化分解、干预/反事实采样、因果影响力量化、特征相关性、模型证伪。

证据：source-reviewed

输入

- DAG：networkx DiGraph 描述变量因果关系。
- data：DataFrame。
- StructuralCausalModel（causalmodels.py）：基于 DAG 构建，每个节点对应一个因果机制。

输出

- 归因字典：{node: attributionscore}（attributeanomalies）。
- interventionalsamples / counterfactualsamples：DataFrame。
- averagecausaleffect：数值。

人工检查

- DAG 结构是否符合真实因果机制（核心人工门）。
- 自动选择的机制类型是否适合数据（连续用 ANM，分类用 ClassifierFCM）。
- evaluatecausalmodel 的评估结果是否可接受。

### 2. 反驳检验

对已估计的因果效应，用多种稳健性检验（加随机共同原因、加未观测共同原因、安慰剂处理、虚拟结果、数据子集、bootstrap）扰动估计过程，判断估计值是否对假设违反敏感，从而为非专家提供"这个因果结论是否站得住"的可检查信号。

证据：source-reviewed

输入

- estimand：IdentifiedEstimand。
- estimate：CausalEstimate（必须有 value，None 则 raise ValueError）。
- methodname：反驳方法名。

输出

- CausalRefutation 对象：
- 函数式入口返回 List[CausalRefutation]。

人工检查

- 反驳结果解读：neweffect 偏离多少算"敏感"，需领域判断。
- 是否跑了足够多种反驳（单一反驳通过不能下结论）。
- placebo/dummy 的效应是否真趋近 0（否则估计有结构性问题）。

### 3. 因果图建模与假设声明

把研究者的因果假设（哪些变量是处理、哪些是结果、哪些是共同原因、哪些是工具变量、哪些是效应修饰因子）和数据，显式编码成内部 CausalModel 和 CausalGraph，使后续识别、估计和反驳都能引用同一套假设。

证据：source-reviewed

输入

- data：pandas DataFrame，含 treatment、outcome 和其他变量列。
- treatment：处理变量名（字符串或列表）。
- outcome：结果变量名（字符串或列表）。

输出

- CausalModel 对象（含 CausalGraph、变量角色、estimator cache）。
- summary 文本。
- 未观测变量/未使用变量 warning（人工判断是否预期）。

人工检查

- 因果 DAG 是否反映真实因果机制（核心人工门，dowhy 无法替代）。
- 未观测变量 warning 是否符合预期（拼写错误 vs 真正未观测）。
- 未使用数据变量是否应纳入图。

### 4. 因果效应估计

在已识别的 estimand 基础上，用指定的统计方法（倾向性得分、回归、双重稳健、工具变量、回归断点、两阶段、GLM、econml CATE）从数据中估计因果效应的数值大小，并给出置信区间、显著性和条件效应。

证据：source-reviewed

输入

- identifiedestimand：来自 identifyeffect 的 IdentifiedEstimand。
- methodname：估计方法名，格式 <identifier.<estimator，如 backdoor.linearregression、backdoor.propensityscorematching、iv.instrumentalvariable、frontdoor.twostageregression、backdoor.econml.dml.DML。
- controlvalue、treatmentvalue：对照组/处理组取值。

输出

- CausalEstimate 对象：

人工检查

- 估计方法选择是否符合数据结构和研究问题（连续/二元 outcome、观察 vs 实验）。
- bootstrap 次数是否足够（399 为 5% 误差率下限）。
- 效应强度和条件效应的解释是否正确。

### 5. 因果效应识别

在已编码的因果图假设下，判断处理对结果的因果效应是否可识别（identifiable），若可识别则给出非参数 estimand 表达式和对应的调整集（backdoor/frontdoor/instrumental variables），若不可识别则明确告知原因。

证据：source-reviewed

输入

- CausalModel 的内部 CausalGraph（networkx DiGraph）。
- treatment、outcome 节点。
- observednodes：观测节点列表。

输出

- IdentifiedEstimand 对象：
- CausalModel.identifier 被赋值。

人工检查

- 因果图假设是否正确（核心人工门）。
- proceedwhenunidentifiable 是否应设为 True（接受潜在未观测混淆）。
- 调整集是否符合领域知识（排除本身就是处理结果的变量）。

### 6. 敏感性分析

量化未观测混淆需要多强才能推翻已估计的因果效应（E-value），并把假想的未观测混淆强度与已观测混淆的强度做基准对比（Observed Covariate E-value），从而给"因果结论对未观测混淆有多稳健"一个可比的数值边界，而不只是 pass/fail 式反驳。

证据：source-reviewed

输入

- estimate：CausalEstimate。
- estimand：IdentifiedEstimand。
- data：DataFrame。

输出

- E-value：未观测混淆的最小关联强度（risk ratio 尺度）。
- Observed Covariate E-value：各已观测混淆的基准 E-value。
- CI 边界变化曲线（matplotlib 可选绘图）。

人工检查

- outcome measure type 选择（ratio vs additive）。
- E-value 阈值解读：多强算"足够稳健"，需领域判断。
- 是否同时跑了反驳检验（敏感性分析应与反驳互补）。

## 处理机制

1. **核心处理链**：端到端因果推断库，用四步流水线统一因果建模、识别、估计与反驳

## 开始方式

1. 因果图建模：把处理、结果、混淆变量的因果假设编码为图模型和变量角色
2. 效应识别：在图结构上判断效应是否可识别，找出有效的调整变量集合
3. 效应估计：用指定统计方法从数据中估计因果效应大小并给出置信区间
4. 反驳检验：用加随机混淆、安慰剂处理、数据子集等方法检验效应估计的稳定性
5. 敏感性分析：量化未观测混淆需要多强才能推翻已有结论（E-value）

## 环境要求

- 准备包含处理、结果和协变量的表格数据
- 基于领域知识画出变量间的因果DAG（有向无环图）
- 选定estimand类型（ATE、CATE、ATE等）和估计方法

## 关键限制

- 跳过因果图建模直接用变量声明跑识别和估计
- 把单一反驳方法通过等同于因果结论成立
- 忽略econml等可选依赖未安装时的降级行为

## 已核对范围

- 静态读取 gcm/init.py 完整 API 导出清单
- 静态读取 gcm/whatif.py interventionalsamples/counterfactualsamples/averagecausaleffect
- 静态读取 gcm/anomaly.py attributeanomalies、gcm/distributionchange.py、gcm/influence.py、gcm/featurerelevance.py 模块位置
- 静态读取 gcm/fittingsampling.py fit/drawsamples、gcm/auto.py assigncausalmechanisms、gcm/modelevaluation.py evaluatecausalmodel、gcm/validation.py refutecausalstructure
- 静态读取 causalmodel.py refuteestimate、causalrefuter.py CausalRefuter 基类与 CausalRefutation 对象
- 静态读取 causalrefuters/ 目录反驳器模块清单与 refuteestimate.py ALLREFUTERS 列表
- 静态读取 testsignificance、choosevariables、SignificanceTestType 枚举
- 静态读取 randomseed、njobs、showprogressbar 参数

## 仍待核对

- 未运行 GCM 流程，未验证归因字典和采样输出
- 未验证 auto.assigncausalmechanisms 的机制选择规则
- 未验证 distributionchange 和 arrowstrength 的数值输出
- 未运行反驳检验，未验证 neweffect 和 pvalue 实际输出
- 未验证各反驳器对估计值的实际扰动幅度
- 未运行 CausalModel 构造，未验证图解析（DOT/GML/DiGraph）实际行为
- 未验证 missingnodesasconfounders 对结果的改变
- 未运行 estimateeffect，未验证任一估计器实际数值输出

## 许可边界

本页的能力分析、结构与说明由 bo老师整理，采用 CC BY 4.0。上游项目、代码、名称和商标继续遵循各自许可。这里不重新发布上游代码。
