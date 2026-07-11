# AI Scientist v2

从开放研究主题到论文产出的端到端自动化科研流水线

面向机器学习研究者，适合从零开始探索开放式研究方向。系统用LLM生成结构化研究想法，通过多阶段Agentic Tree Search自动实验探索、解析指标、聚合图表，最后撰写论文并做图文一致性复核。

- 官网详情：https://bolaoshi.cn/research-tools/ai-scientist-v2
- 原始来源：https://github.com/SakanaAI/AI-Scientist-v2
- 读取版本：96bd51617cfdbb494a9fc283af00fe090edfae48
- 核对日期：2026-06-16
- GitHub Stars：6,800（2026-07-11）
- 上游许可：AI-Scientist-Source-Code-License-1.0
- 验证程度：catalogued
- 编辑判断：use-with-caution

## 什么时候使用

- 有一个高层研究主题，希望自动生成候选研究方向并评估可行性
- 需要自动化执行多组实验探索并自动记录结果与指标
- 希望将实验数据自动汇总为论文草稿和图表
- 需要对生成论文做自动评审和图文一致性复核

## 先准备什么

- 准备一个Markdown格式的高层研究主题描述文件
- 配置LLM和VLM模型参数，确保API密钥可用
- 可选：准备好实验代码和Hugging Face数据集引用文件
- 确保运行环境满足GPU、磁盘空间和网络访问要求

## 它会怎样推进

1. **生成研究想法**：LLM基于主题描述生成多个结构化研究idea，并搜索相关文献辅助判断新颖性
2. **装配实验配置**：选定一个idea，自动创建实验目录、注入代码和数据集引用，生成专属配置
3. **探索实验树**：多阶段Agent自动生成实验计划、执行代码、解析指标并选择最优节点继续扩展
4. **解析与反馈**：节点级执行捕获stdout、解析metric、生成图像并用VLM提供图文反馈
5. **聚合图表**：LLM根据阶段摘要和数据文件生成聚合脚本，产出论文所需的最终图像
6. **撰写与评审**：收集引用生成LaTeX论文，编译PDF，LLM文本评审加VLM图文一致性复核

## 第一次这样开始

帮我安装这个库：https://github.com/SakanaAI/AI-Scientist-v2
安装前先告诉我需要什么环境，安装后帮我跑通一个最小示例。
请先说明许可证和安装风险，等我确认后再安装。

## 结果出来后先看什么

- 实验树是否按预期生成并完成所有阶段，检查checkpoint和阶段摘要完整性
- 最终图表是否来自真实实验数据，变量名和样本数可追溯
- 论文引用是否真实存在且与正文断言匹配
- 生成论文是否显著披露了机器生成和The AI Scientist使用信息

## 常见误区

- 直接信任LLM生成的研究想法质量而不做人工复核，低质量想法浪费实验资源
- 忽略许可证要求，传播生成论文时未显著披露机器生成身份
- 依赖单一metric排序最优节点，忽略无法被数值化的研究质量维度
- 让图表聚合脚本猜测数据文件结构而不提供schema，导致生成看似合理但错误的图表

## 能力拆解

### 1. idea到实验目录与BFTS配置装配

把一个 pregenerated idea JSON 中的单条 idea 装配成独立实验目录，补写 idea.md、idea.json 和运行专属 bftsconfig.yaml，再把配置交给 BFTS 实验执行入口。

证据：source-reviewed

输入

- --loadideas 指向的 idea JSON。
- --ideaidx 指定要运行的 idea。
- 可选 --loadcode，读取和 idea JSON 同名的 .py 文件。

输出

- experiments/<timestamp<ideaattempt<id/idea.md
- experiments/<timestamp<ideaattempt<id/idea.json
- experiments/<timestamp<ideaattempt<id/bftsconfig.yaml

人工检查

- 是否允许加载和执行同名 .py。
- 是否允许加入 HF dataset reference。
- BFTS 配置中的模型和成本是否可接受。

### 2. 主题自由研究想法生成与文献检索

把一个高层研究主题 Markdown 变成一组结构化研究 idea，并要求 LLM 在定稿前至少做一次 Semantic Scholar 文献搜索。输出 idea JSON 后，主实验流水线再按 --loadideas 读取其中一个 idea。

证据：source-reviewed

输入

- 研究主题 Markdown，例如 aiscientist/ideas/icantbelieveitsnotbetter.md。
- 模型名，必须在 AVAILABLELLMS 内。
- max-num-generations：最多生成 proposal 数。

输出

- aiscientist/ideas/<topic.json
- 每条 idea 的结构化字段。
- 后续 launchscientistbfts.py --loadideas 的输入。

人工检查

- 主题范围是否足够明确。
- Related Work 是否真正来自搜索结果。
- idea 是否适合自动执行实验。

### 3. 多阶段agentic tree search实验探索

把单个研究 idea 转成一个由多阶段 agent 管理的实验树。系统不再依赖人工模板中的固定实验脚本主轴，而是通过 AgentManager 拆出四个主阶段，让 ParallelAgent 在每个阶段生成 plan、执行代码、解析指标、挑选节点并继续扩展。

证据：source-reviewed

输入

- idea JSON，至少包含研究题目、描述、实验目标和约束。
- bftsconfig.yaml 中的 agent、search、exec、llm、workspace 配置。
- codedir、datadir、basedir 和实验工作目录。

输出

- stage summary JSON。
- tree export JSON。
- journal checkpoint。

人工检查

- 四个阶段是否符合当前研究任务。
- metric 是否真的代表研究目标。
- best node 是否需要人工复核后再进入下一阶段。

### 4. 实验摘要到最终图表聚合

把多阶段 tree search 产生的 stage summaries 和实验数据合并成论文写作可用的最终图表。这个能力是实验探索和论文写作之间的窄口，核心在于它用 LLM 根据已有摘要和数据文件写聚合脚本，并尝试把散落的结果变成最终图像。

证据：source-reviewed

输入

- 各阶段 stage summary。
- 实验输出目录中的数据文件和图像文件。
- experimentdata.npy 或等价实验数据。

输出

- 聚合脚本。
- stdout 和 stderr。
- 最终图像文件。

人工检查

- 最终图表是否来自真实数据。
- 图表变量、轴名和样本数是否可追溯。
- 聚合脚本是否读取了正确文件。

### 5. 引用收集与论文写作编译

把实验摘要、最终图表和研究 idea 组织成论文草稿，并通过引用搜索、BibTeX 缓存、LaTeX 生成、编译和反思修订形成 PDF。这个能力把实验产物转成可传播文本，所以同时承担引用真实性、结果忠实度和许可证披露边界。

证据：source-reviewed

输入

- idea JSON。
- BFTS 实验摘要、stage summaries 和最终图表。
- 论文模板、目标会议或格式约束。

输出

- LaTeX 源文件。
- BibTeX 引用文件。
- PDF 草稿和反思后 PDF。

人工检查

- 引用是否真实存在且与正文断言匹配。
- 论文是否显著披露机器生成和 The AI Scientist 使用。
- 图表是否来自真实实验数据。

### 6. 节点执行指标解析与图像反馈

让搜索树中的单个节点从“计划和代码”变成“可比较的实验状态”。它负责执行节点代码、捕获 stdout 和 stderr、解析 metric、生成图像、用 VLM 读取图像反馈，再把这些结果写回 Node，供 tree search 继续选择和修复。

证据：source-reviewed

输入

- 当前节点的计划、代码、工作目录和前序节点上下文。
- Interpreter 执行配置、超时时间、环境变量和目录。
- metric 名称、metric 解析 prompt 或解析规则。

输出

- 节点执行日志。
- metric JSON 或等价字段。
- plot 脚本和图像文件。

人工检查

- metric 是否足以代表实验目标。
- 图像是否忠实呈现运行数据。
- 失败节点是否保留足够审计证据。

## 处理机制

1. **tree search状态对象与目录链路**：这个机制解释 AI-Scientist-v2 如何把 idea、BFTS 配置、workspace、stage、agent、journal、node、summary 和 final artifacts 串成一条可审计的状态链。它支撑装配、tree search、节点执行和图表聚合四张能力卡。
2. **论文产物质量门与反思链**：这个机制解释 AI-Scientist-v2 如何把实验摘要、最终图表、引用、LaTeX、PDF、LLM review 和 VLM review 串成论文产物质量门。它支撑图表聚合、写作编译和论文评审三张能力卡。
3. **运行安全与外部服务边界**：这个机制解释 AI-Scientist-v2 的运行风险来自哪里：LLM 写代码、子进程执行、多 worker、GPU、进程清理、外部模型 API、Semantic Scholar、VLM、LaTeX、PDF 工具和自定义许可证。它支撑本来源全部能力卡的运行边界判断。

## 开始方式

1. 生成研究想法：LLM基于主题描述生成多个结构化研究idea，并搜索相关文献辅助判断新颖性
2. 装配实验配置：选定一个idea，自动创建实验目录、注入代码和数据集引用，生成专属配置
3. 探索实验树：多阶段Agent自动生成实验计划、执行代码、解析指标并选择最优节点继续扩展
4. 解析与反馈：节点级执行捕获stdout、解析metric、生成图像并用VLM提供图文反馈
5. 聚合图表：LLM根据阶段摘要和数据文件生成聚合脚本，产出论文所需的最终图像
6. 撰写与评审：收集引用生成LaTeX论文，编译PDF，LLM文本评审加VLM图文一致性复核

## 环境要求

- 准备一个Markdown格式的高层研究主题描述文件
- 配置LLM和VLM模型参数，确保API密钥可用
- 可选：准备好实验代码和Hugging Face数据集引用文件
- 确保运行环境满足GPU、磁盘空间和网络访问要求

## 关键限制

- 直接信任LLM生成的研究想法质量而不做人工复核，低质量想法浪费实验资源
- 忽略许可证要求，传播生成论文时未显著披露机器生成身份
- 依赖单一metric排序最优节点，忽略无法被数值化的研究质量维度
- 让图表聚合脚本猜测数据文件结构而不提供schema，导致生成看似合理但错误的图表

## 已核对范围

- 读取 launchscientistbfts.py 的 CLI 参数、idea 读取、目录创建、code 和 dataset reference 注入
- 读取 bftsutils.py 的 idea Markdown 转换和 BFTS 配置重写
- 读取 bftsconfig.yaml 的 workspace、log、agent、search 和模型参数
- 读取 README 的 ideation 入口、参数和输出说明
- 读取 performideationtempfree.py 的工具调用、反思、JSON 解析和归档逻辑
- 读取 semanticscholar.py 的请求、排序和失败返回逻辑
- 读取 BFTS 主入口、AgentManager 阶段管理和 ParallelAgent 节点扩展代码
- 读取 journal 节点状态、summary 生成和 stage transition 逻辑

## 仍待核对

- 未实际创建 experiments 目录
- 未运行 BFTS
- 未验证 loadcode 与 adddatasetref 的组合效果
- 未调用真实 LLM
- 未调用 Semantic Scholar API
- 未验证 idea 质量、新颖性判断或 JSON 稳定性
- 未运行 tree search
- 未调用 LLM 生成实验计划

## 许可边界

本页的能力分析、结构与说明由 bo老师整理，采用 CC BY 4.0。上游项目、代码、名称和商标继续遵循各自许可。这里不重新发布上游代码。
