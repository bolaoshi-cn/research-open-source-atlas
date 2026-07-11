# Agent Laboratory

用多Agent自动完成文献检索、实验代码生成和论文写作的研究流水线

适合希望用AI Agent自动化计算研究全流程的研究者。用YAML配置研究主题和模型后，多个Agent依次完成文献检查、方案规划、代码实验、论文撰写和评审回环，支持人工确认节点。

- 官网详情：https://bolaoshi.cn/research-tools/agentlaboratory
- 原始来源：https://github.com/SamuelSchmidgall/AgentLaboratory
- 读取版本：d9017d90e329112d2a80b7712f37ee9094d2cd27
- 核对日期：2026-06-16
- GitHub Stars：5,741（2026-07-11）
- 上游许可：MIT
- 验证程度：catalogued
- 编辑判断：worth-watching

## 什么时候使用

- 需要自动化文献检索、实验代码生成和论文初稿撰写的完整科研流程
- 想快速探索一个研究方向，用多Agent协作生成实验代码并评分迭代
- 需要将实验结果自动编排为结构化的LaTeX论文，含评审和改进回环
- 希望原型化研究想法，通过可选的人工审核门控制自动化程度

## 先准备什么

- 一个YAML配置文件，指定研究主题、LLM模型和工作流参数
- OpenAI或DeepSeek API密钥，确保有足够额度支撑多阶段LLM调用
- Python环境，包含PyTorch、HuggingFace datasets和sentence-transformers
- 可选：LaTeX发行版（pdflatex）以生成编译后的PDF论文

## 它会怎样推进

1. **配置与启动**：在YAML中定义研究主题、模型和预算，启动流水线脚本
2. **文献检索**：PhD Agent搜索arXiv、下载PDF并筛选论文，直至达到目标数量
3. **方案规划**：Postdoc和PhD Agent通过对话协作生成研究方案，提交人工确认
4. **运行实验**：MLESolver迭代生成、执行和评分实验代码，保留最优版本
5. **撰写论文**：PaperSolver逐章填充LaTeX框架，编译并通过LLM评分
6. **评审与回环**：三位评审Agent打分；根据分数决定结束或回到规划阶段重新开始

## 第一次这样开始

帮我安装这个库：https://github.com/SamuelSchmidgall/AgentLaboratory
安装前先告诉我需要什么环境，安装后帮我跑通一个最小示例。

## 结果出来后先看什么

- 检查文献综述是否覆盖了相关论文，而非仅仅搜索arXiv首页结果
- 验证实验代码能正常运行，输出的结果与论文中的主张一致
- 逐一核对论文引用是否真实存在且与正文内容匹配
- 确认评审回环未导致实验代码或论文质量退化

## 常见误区

- 不设置API调用预算上限就运行流水线，多Agent循环可能导致高额费用
- 信任LLM生成的实验代码而不做人工代码审查就直接采用结果
- 未核实自动生成的引用文献是否存在，直接提交包含幻觉引用的论文
- 让工具在修订回环中无限制循环，未设定最大步骤数

## 能力拆解

### 1. AgentRxiv本地论文共享检索

在本地启动一个 AgentRxiv Flask 服务，把上传或自动生成的 PDF 抽取成文本并存入 SQLite，再用 sentence transformer embedding 做相似度搜索，让后续 agent 在文献综述阶段检索其他 agent 产出的论文全文和摘要。

证据：source-reviewed

输入

- agentRxiv 配置开关。
- uploads/ 目录中的 PDF。
- 写作阶段生成的 <labdir/tex/temp.pdf。

输出

- uploads/<title.pdf
- SQLite Paper 记录。
- AgentRxiv 搜索候选文本。

人工检查

- 是否允许生成 PDF 进入共享库。
- 是否要过滤低质量或未完成论文。
- 是否允许后续 agent 引用 agent 生成论文。

### 2. HuggingFace数据准备与提交

让 ML engineer 和 software engineer 围绕研究计划协作，必要时搜索 HuggingFace 数据集，逐步执行候选数据准备代码，直到 software engineer 提交通过执行检查的最终 loaddata.py。

证据：source-reviewed

输入

- 研究计划 self.plan。
- 文献综述 litreviewsum。
- 数据准备 notes。

输出

- <labdir/src/loaddata.py
- self.datasetcode
- 后续实验代码生成阶段的固定前置代码。

人工检查

- 数据集是否适配研究问题。
- 数据许可证和使用限制。
- 是否允许执行数据准备代码。

### 3. arXiv文献检索与综述归档

让 PhD agent 围绕研究主题反复发出 SUMMARY、FULLTEXT 和 ADDPAPER 命令，从 arXiv 搜索摘要、读取论文全文、选择相关论文，并把达到数量阈值的论文摘要整理成后续计划阶段可用的文献综述。

证据：source-reviewed

输入

- 研究主题 researchtopic。
- PhD agent 当前历史、phase notes 和已有 litreview。
- arXiv 搜索 query。

输出

- self.phd.litreview
- self.referencepapers
- litreviewsum

人工检查

- 论文是否确实相关。
- 摘要是否准确反映全文。
- 检索 query 是否覆盖研究主题关键同义词。

### 4. 实验代码生成修复与评分

把数据准备代码、研究计划和文献洞察交给 MLESolver，让模型用 REPLACE 或 EDIT 生成实验代码，先运行代码，再用 LLM reward model 给代码与输出打分，保留最高分代码并写入实验目录。

证据：source-reviewed

输入

- datasetcode
- plan
- litreviewsum

输出

- <labdir/src/runexperiments.py
- <labdir/src/experimentoutput.log
- self.resultscode

人工检查

- 是否允许执行模型生成代码。
- reward 分数是否可信。
- 输出是否真实覆盖计划中的实验。

### 5. 研究计划协作生成

把文献综述、研究主题和用户 notes 放进 postdoc 与 PhD student 的往返对话，让 postdoc 在有限步数内提交 PLAN，形成后续数据准备、实验执行和论文写作共享的研究计划。

证据：source-reviewed

输入

- litreviewsum
- researchtopic
- task-notes 中的 plan-formulation 条目。

输出

- self.plan
- 所有 agent 的 plan 属性。
- 数据准备、实验、结果解释和论文写作的共享上下文。

人工检查

- 计划是否符合研究主题。
- 数据、模型、指标、baseline 和预算是否完整。
- 是否允许自由文本计划进入自动代码生成。

### 6. 论文写作与LaTeX产物生成

把研究计划、实验代码、实验输出、结果解释和文献综述交给 PaperSolver，先生成 LaTeX scaffold，再按章节填入正文、相关论文和图表引用，经过编译检查和 LLM 评审评分后，输出报告文本、readme 和可选 PDF。

证据：source-reviewed

输入

- self.phd.plan
- self.phd.resultscode
- self.phd.expresults

输出

- <labdir/tex/temp.tex
- <labdir/tex/temp.pdf
- <labdir/report.txt

人工检查

- 论文是否准确反映实验。
- 引用是否支持正文主张。
- PDF 是否可传播。

## 处理机制

1. **运行文件合同与外部依赖边界**：这个机制解释 AgentLaboratory 的运行文件、目录、外部 API、代码执行、LaTeX 编译和 AgentRxiv 服务如何共同构成复现边界。它跨越配置调度、数据准备、实验代码、论文写作和 AgentRxiv 本地共享能力。
2. **阶段命令协议与状态记忆**：这个机制解释 AgentLaboratory 如何用单命令协议、agent history、phase status、过期上下文和 pickle checkpoint 串联多阶段研究流程。它支撑配置调度、文献综述、计划、数据准备、实验、论文写作和评审回环多张能力卡。

## 开始方式

1. 配置与启动：在YAML中定义研究主题、模型和预算，启动流水线脚本
2. 文献检索：PhD Agent搜索arXiv、下载PDF并筛选论文，直至达到目标数量
3. 方案规划：Postdoc和PhD Agent通过对话协作生成研究方案，提交人工确认
4. 运行实验：MLESolver迭代生成、执行和评分实验代码，保留最优版本
5. 撰写论文：PaperSolver逐章填充LaTeX框架，编译并通过LLM评分
6. 评审与回环：三位评审Agent打分；根据分数决定结束或回到规划阶段重新开始

## 环境要求

- 一个YAML配置文件，指定研究主题、LLM模型和工作流参数
- OpenAI或DeepSeek API密钥，确保有足够额度支撑多阶段LLM调用
- Python环境，包含PyTorch、HuggingFace datasets和sentence-transformers
- 可选：LaTeX发行版（pdflatex）以生成编译后的PDF论文

## 关键限制

- 不设置API调用预算上限就运行流水线，多Agent循环可能导致高额费用
- 信任LLM生成的实验代码而不做人工代码审查就直接采用结果
- 未核实自动生成的引用文献是否存在，直接提交包含幻觉引用的论文
- 让工具在修订回环中无限制循环，未设定最大步骤数

## 已核对范围

- 读取 AgentRxiv 类的本地 Flask server 启动、搜索和 PDF 缓存逻辑
- 读取 app.py 的 PDF 上传、SQLite 入库、embedding 搜索和 API 返回
- 读取 literaturereview 和 reportwriting 中的 AgentRxiv 接入点
- 读取 HFDataSearch 的数据集检索、过滤和评分逻辑
- 读取 SWEngineerAgent 与 MLEngineerAgent 的 data preparation 命令协议
- 读取 LaboratoryWorkflow.datapreparation 的代码执行、SEARCHHF 和 SUBMITCODE 分支
- 读取 PhDStudentAgent 的文献命令协议
- 读取 LaboratoryWorkflow.literaturereview 的 SUMMARY、FULLTEXT 和 ADDPAPER 分支

## 仍待核对

- 未启动 Flask server
- 未上传 PDF
- 未下载 sentence transformer 模型
- 未验证并行 lab 的 AgentRxiv 路径
- 未下载 HuggingFace dataset 索引
- 未运行模型生成的数据准备代码
- 未验证代码安全隔离
- 未验证真实 MATH 数据加载

## 许可边界

本页的能力分析、结构与说明由 bo老师整理，采用 CC BY 4.0。上游项目、代码、名称和商标继续遵循各自许可。这里不重新发布上游代码。
