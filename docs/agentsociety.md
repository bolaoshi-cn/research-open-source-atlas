# AgentSociety

用LLM驱动的大规模社会模拟平台，支撑计算社会科学全流程研究

适合社科研究者用LLM驱动的人格化Agent验证社会机制假说。工作流程：从文献出发生成可执行假设，编排实验步骤，运行模拟后自动产出双语分析报告。

- 官网详情：https://bolaoshi.cn/research-tools/agentsociety
- 原始来源：https://github.com/tsinghua-fib-lab/AgentSociety
- 读取版本：source-reviewed
- 核对日期：2026-06-19
- GitHub Stars：1,118（2026-07-11）
- 上游许可：Apache-2.0
- 验证程度：catalogued
- 编辑判断：worth-watching

## 什么时候使用

- 想用LLM驱动的人格化Agent模拟信息茧房、群体极化等社会现象
- 需要设计对照实验来验证政策或干预措施在社会系统中的效果
- 研究需要从文献检索到模拟分析再到报告产出的全流程自动化

## 先准备什么

- 一个明确的社会科学问题或可验证的社会机制假说
- OpenAI或其他兼容LLM的API密钥
- 已安装Python和Ray分布式计算环境
- 实验所需的Agent人格描述和环境参数

## 它会怎样推进

1. **检索文献**：从研究主题出发，通过多源检索获取相关论文并建立结构化文献索引
2. **生成假设**：基于文献自动生成可验证研究假设，设计实验组与对照组的实验骨架
3. **编排实验**：将假设转化为可执行实验配方，配置Agent人格、环境模块和步骤序列
4. **运行模拟**：用Ray分布式调度并行驱动大量Agent在模拟环境中交互和决策
5. **分析结果**：对模拟数据进行探索性分析，生成图表和双语研究报告

## 第一次这样开始

帮我安装这个库：https://github.com/tsinghua-fib-lab/AgentSociety
安装前先告诉我需要什么环境，安装后帮我跑通一个最小示例。

## 结果出来后先看什么

- 检查模拟数据量是否足够支撑研究结论
- 确认LLM决策行为与Agent人格设定的一致性
- 验证分析报告中的统计推断是否基于多seed重复实验
- 核对双语报告中的图表是否准确反映模拟结果

## 常见误区

- 把LLM驱动的Agent当作传统ABM使用，忽视其内生决策随机性
- 未做多seed重复实验就直接下统计结论
- 依赖LLM Agent行为的表面真实性，放松对严格实验设计的要求
- 忽略环境模块的模拟保真度验证，直接采信模拟结果

## 能力拆解

### 1. Agent分布式调度执行

给定 agentspecs 集合和时钟，并行推进一个 tick：driver 不持有 agent 对象（record-based），每 tick 切批（BATCHSIZE 默认 256）→ 每批一个 Ray Task → task 内 asyncio.gather 并发跑该批所有 agent → 单 agent 异常隔离不中断整批。这是其相对传统 ABM 的架构卖点：内存与 N 解耦，并行来自"大量 Task"。

证据：source-reviewed

输入

- agentids + workspaceroot + ServiceProxy。

输出

- 每 tick 的 agent summary 列表 + tokenstats。

人工检查

- 核对结果是否符合当前研究问题和证据边界

### 2. LLM-native决策循环ReAct

给定 tick、当前时间、observation 列表和人格，用 LLM 做多轮 ReAct 决策（每轮 LLM 产 decisions → 执行工具 → 观察追加），直到 finish 或达 maxreactturns。这是 LLM-native ABM 的核心：用 LLM 驱动人格化 agent 决策，而非传统 ABM 的规则驱动。

证据：source-reviewed

输入

- tick、当前时间、observation 列表。
- 可选 question/skillhooks。

输出

- 最终结果字符串。
- 落盘的 AGENT.json + trace spans。

人工检查

- 核对结果是否符合当前研究问题和证据边界

### 3. 假设生成与实验组设计

把文献索引转换成可执行假设（HYPOTHESIS.md）和实验设计骨架（SIMSETTINGS.json，含 agentclasses、envmodules、实验组/对照组），经 Pydantic schema 校验和模块选择校验（至少一个 agent + 一个 env）。这是"文献→可执行假设+实验设计骨架"的转换，直接命中科研生命周期的实验设计环节。

证据：source-reviewed

输入

- 文献索引（literature index）。

输出

- hypothesis{id}/HYPOTHESIS.md。
- hypothesis{id}/SIMSETTINGS.json（含 agentclasses、envmodules、实验组）。

人工检查

- 核对结果是否符合当前研究问题和证据边界

### 4. 实验配置与步骤编排

把假设（SIMSETTINGS.json）转换成可执行实验配方（initconfig.json + steps.yaml），经模块发现、agent spec 拆分（profile/config）、Pydantic 校验。步骤语言提供 4 类原语：run（N 步×tick 秒）、ask（提问）、intervene（干预/实验操纵）、questionnaire（问卷测量）。对照实验通过多份 profile + 多份 initconfig 实现。

证据：source-reviewed

输入

- SIMSETTINGS.json（来自假设生成）。

输出

- initconfig.json（envmodules + agents + codegenrouter）。
- steps.yaml（步骤序列）。
- configparams.py。

人工检查

- 核对结果是否符合当前研究问题和证据边界

### 5. 文献检索与索引化

把研究主题（中文或英文 TOPIC.md）转换成结构化文献库（papers/ 目录 + literatureindex.json），经中文→英文翻译、多 query 拆分、MCP gateway 多源检索（local/arxiv/crossref/openalex）、全文获取和格式化索引。这是社科模拟流水线的入口环节。

证据：source-reviewed

输入

- 研究主题 TOPIC.md（中文或英文）。

输出

- papers/ 目录 + papers/literatureindex.json。

人工检查

- 核对结果是否符合当前研究问题和证据边界

### 6. 模拟数据收集与持久化replay

把模拟运行轨迹（env step 状态 + agent profile + agent snapshot）收集成可查询数据集：分布式、lock-free 写入 sharded JSONL，用 schema.json catalog 注册 schema 和 capabilities，DuckDB 读侧跨分片查询。支持 entitysnapshot/geopoint/agentsnapshot 等 capabilities 元数据驱动查询。

证据：source-reviewed

输入

- env step 状态 + agent profile + agent snapshot。

输出

- run/replay/ 下 sharded JSONL + schema.json catalog。
- DuckDB 读侧可查询视图。

人工检查

- 核对结果是否符合当前研究问题和证据边界

## 处理机制

1. **LLM-native社科模拟实验流水线**：AgentSociety 的多张能力卡共享一条完整的研究流水线：文献→假设→实验配置→运行→分析→论文。对象从研究主题出发，经文献库、假设+实验组、initconfig+steps.yaml、agent workspace+replay，最终变成双语报告+经验记忆。本机制卡说明这些对象如何跨阶段传递，以及哪些状态会阻断或降级。

## 开始方式

1. 检索文献：从研究主题出发，通过多源检索获取相关论文并建立结构化文献索引
2. 生成假设：基于文献自动生成可验证研究假设，设计实验组与对照组的实验骨架
3. 编排实验：将假设转化为可执行实验配方，配置Agent人格、环境模块和步骤序列
4. 运行模拟：用Ray分布式调度并行驱动大量Agent在模拟环境中交互和决策
5. 分析结果：对模拟数据进行探索性分析，生成图表和双语研究报告

## 环境要求

- 一个明确的社会科学问题或可验证的社会机制假说
- OpenAI或其他兼容LLM的API密钥
- 已安装Python和Ray分布式计算环境
- 实验所需的Agent人格描述和环境参数

## 关键限制

- 把LLM驱动的Agent当作传统ABM使用，忽视其内生决策随机性
- 未做多seed重复实验就直接下统计结论
- 依赖LLM Agent行为的表面真实性，放松对严格实验设计的要求
- 忽略环境模块的模拟保真度验证，直接采信模拟结果

## 已核对范围

- 静态确认 record-based 设计：driver 不持有 agent 对象
- 静态确认 stepagentbatch @ray.remote 切批 + asyncio.gather 并发 + AdaptiveSemaphore 限流
- 静态确认 per-agent 异常隔离（不中断整批）
- 静态确认 runreactloop 的 observe → 多轮 ReAct → finish/maxturns 循环
- 静态确认 6 种路由模式（CodeGen/ReAct/Plan-Execute/Two-Tier-ReAct/Two-Tier-Plan-Execute/Search-Tool）
- 静态确认 ReactDecision 和 ReactToolResult 对象
- 静态确认 skills/hypothesis 的 manager/models 和 validatehypothesiswithmodules
- 静态确认 HypothesisDataModel 和 ExperimentGroupModel（对照/实验组）schema

## 仍待核对

- 未真实运行大规模 agent 模拟
- 未验证 record-based 设计在大 N 下的内存/磁盘表现
- 未真实运行 agent 决策循环
- 未验证不同路由模式在社科模拟中的效果差异
- 未真实运行假设生成
- 未验证生成假设的科研质量
- 未真实运行实验编排
- 未验证对照实验多 config 编排在真实场景的效果

## 许可边界

本页的能力分析、结构与说明由 bo老师整理，采用 CC BY 4.0。上游项目、代码、名称和商标继续遵循各自许可。这里不重新发布上游代码。
