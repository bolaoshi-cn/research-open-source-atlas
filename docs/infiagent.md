# infiAgent

用YAML配置多层agent完成跨小时/跨天的全流程科研任务

适合需要让AI agent自主完成文献综述、实验执行和论文撰写的长周期研究任务。研究者只需给出研究主题，infiAgent用config文件编排多层agent流水线，支持窗口重置记忆和断点续跑。

- 官网详情：https://bolaoshi.cn/research-tools/infiagent
- 原始来源：https://github.com/polyuiislab/infiAgent
- 读取版本：source-reviewed
- 核对日期：2026-06-19
- GitHub Stars：1,181（2026-07-11）
- 上游许可：GPL-3.0
- 验证程度：catalogued
- 编辑判断：worth-watching

## 什么时候使用

- 需要AI自主完成从文献检索到LaTeX论文输出的完整科研流水线
- 面临需要读50篇以上论文、跨多天运行的上下文密集型研究任务
- 想用YAML配置而非写代码的方式定义科研agent的行为和工具权限
- 需要崩溃后可续跑的长任务执行能力

## 先准备什么

- 准备一句明确的研究主题或综述题目
- 安装infiAgent并配置LLM API密钥
- 根据任务复杂度选择合适的agent模板（Researcher用于科研全流程）
- 评估GPL-3.0许可证是否兼容你的使用场景

## 它会怎样推进

1. **编写YAML配置**：定义agent层级、工具白名单、模型选择和提示词模板
2. **输入研究主题**：提交研究问题，系统自动启动多级agent编排
3. **文献采集分析**：agent搜索arxiv、Google Scholar等来源，维护统一引用库
4. **实验与作图**：agent规划并运行Python实验，生成数据图和框架图
5. **论文撰写输出**：agent逐章撰写LaTeX论文，经质控agent审查后产出终稿

## 第一次这样开始

帮我安装这个库：https://github.com/polyuiislab/infiAgent
安装前先告诉我需要什么环境，安装后帮我跑通一个最小示例。

## 结果出来后先看什么

- 检查论文中的引用是否真实存在、DOI是否正确
- 人工复核实验代码和数据分析逻辑是否合理
- 验证论文的论点和结论是否与所引用文献一致

## 常见误区

- 把agent生成的论文直接投稿而不做人工审校，论文质量需逐章验证
- 在GPL-3.0不兼容的闭源项目中复制infiAgent源码
- 默认thinking固化在超长任务中不会丢失关键信息而不做抽查

## 能力拆解

### 1. AI-Scientist科研全流程

把一句研究主题转成完整 LaTeX 论文项目（paper.tex + references.bib + figures/）或 10000+ 字 Markdown 研究报告，经 alphaagent.workflow 明文编排：datacollection（arxiv/scholar/wiki/websearch + answerfrompapers + reference.bib 维护）→ dataanalysis（出 idea/实验计划）→ coder（coderun 跑实验，限 Python）→ createfigures（matplotlib 数据图 + AI 框架图）→ materialtodocument（拼 main.tex + 逐章 subparteditor + proof 润色）→ judge（质控）→ finaloutput。

证据：source-reviewed

输入

- 一句研究主题（如"Write a survey paper on Transformers"）。

输出

- 完整 LaTeX 论文项目（paper.tex + references.bib + figures/）。
- 或 10000+ 字 Markdown 研究报告。

人工检查

- 核对结果是否符合当前研究问题和证据边界

### 2. anthropic-skills对接

把外部 SKILL.md 兼容的 skill（agentskills.io 标准）接入 agent 运行时，实现发现-注入-执行-卸载完整生命周期：扫描 skillslibrary//SKILL.md 的 YAML frontmatter → 生成 ~100 token/skill 的发现清单注入 system prompt → agent 调 loadskill → copytree 到 workspace + SKILL.md 全文注入 → agent 用 fileread/executecommand 执行 → offloadskill 仅摘除上下文不删盘。

证据：source-reviewed

输入

- skillslibrary/<name/SKILL.md（YAML frontmatter: name+description+license+compatibility）。

输出

- skill 部署到 workspace + 上下文注入。

人工检查

- 核对结果是否符合当前研究问题和证据边界

### 3. config驱动多层agent构建

把一组 YAML 配置文件（按 level 切分：level -1 judge / level 0 tools / level 1 executor / level 2 expert / level 3 orchestrator）转成可直接调度的 agent 实例树，每个节点带可用工具白名单、多模型配置和提示词。改 YAML 即改 agent 行为，无需改代码。

证据：source-reviewed

输入

- 一个 agentsystem 目录下的 YAML 集合（level -1 到 level 3）。

输出

- 可直接调度的 AgentExecutor 实例树。

人工检查

- 核对结果是否符合当前研究问题和证据边界

### 4. infinite-horizon执行引擎

让 agent 在跨小时/跨天的长任务中持续运行而不上下文溢出或不丢失关键信息，通过十步窗口+思考模块重置（每 N 步产 thinking 文本后清空 actionhistory，记忆转移进 thinking 而非压缩历史）、断点续跑（状态落盘崩溃可恢复）、fresh 热刷新（安全点重载 config）、文件即状态（子 agent 结果写文件不回传上下文）。这是本仓库最具原创性的能力。

证据：source-reviewed

输入

- 一条用户指令 + taskid（workspace 绝对路径）。

输出

- 跨小时/跨天的任务最终产物（文件）+ finaloutput。

人工检查

- 核对结果是否符合当前研究问题和证据边界

### 5. 串行树形多agent协作

父 agent 在主循环中经 toolcall 调用子 agent（toolcall 名 = 子 agent 名），递归 new AgentExecutor 并入栈/出栈，子 agent 的工具白名单由其 YAML 声明，父 agent 无法授予子 agent 自己没有的工具：这是调用图边界控制。子 agent 结果作为工具结果返回父 agent。

证据：source-reviewed

输入

- 父 agent 的 LLM 决策（toolcall 名 = 子 agent 名，参数含 taskinput）。

输出

- 子 agent finaloutput（dict 含 status/output/文件路径）。

人工检查

- 核对结果是否符合当前研究问题和证据边界

## 处理机制

1. **窗口重置记忆与config分层编排**：infiAgent 的多张能力卡共享两条机制链：(1) config-driven：YAML 分层定义 agent 树，工具白名单构成调用图边界，父 agent 调子 agent 经递归 AgentExecutor 入栈出栈；(2) infinite-horizon：主循环每轮落盘，每 N 步 thinking 固化进度后清空 actionhistory，崩溃可续跑，fresh 可热刷新。本机制卡说明这些对象如何跨能力传递。

## 开始方式

1. 编写YAML配置：定义agent层级、工具白名单、模型选择和提示词模板
2. 输入研究主题：提交研究问题，系统自动启动多级agent编排
3. 文献采集分析：agent搜索arxiv、Google Scholar等来源，维护统一引用库
4. 实验与作图：agent规划并运行Python实验，生成数据图和框架图
5. 论文撰写输出：agent逐章撰写LaTeX论文，经质控agent审查后产出终稿

## 环境要求

- 准备一句明确的研究主题或综述题目
- 安装infiAgent并配置LLM API密钥
- 根据任务复杂度选择合适的agent模板（Researcher用于科研全流程）
- 评估GPL-3.0许可证是否兼容你的使用场景

## 关键限制

- 把agent生成的论文直接投稿而不做人工审校，论文质量需逐章验证
- 在GPL-3.0不兼容的闭源项目中复制infiAgent源码
- 默认thinking固化在超长任务中不会丢失关键信息而不做抽查

## 已核对范围

- 静态确认 Researcher alphaagent.workflow 编排（datacollection→dataanalysis→coder→figures→document→judge）
- 静态确认 level 1-2 agent 分工和 reference.bib 维护
- 静态确认 SkillLoader 的 discover/buildxml/load/offload 生命周期
- 静态确认 loadskill copytree 到 workspace + SKILL.md 全文注入
- 静态确认 16 个内置 skill（pdf/pptx/docx/xlsx/mcp-builder 等）
- 静态确认 ConfigLoader 的 YAML 分层加载和 level 自动授权
- 静态确认 agent YAML 字段（type/level/availabletools/model/prompts/maxturns）
- 静态确认 thinking 窗口重置机制（每 N 步 thinking + 清空 actionhistory）

## 仍待核对

- 未真实运行 Researcher 端到端
- 未验证论文产出质量（README 宣称过 EI/IEEE 评审）
- 未真实运行 skill 加载和卸载
- 未真实运行 config 加载
- 未真实运行超长任务（1000 步）
- 未验证 thinking 重置在超长任务下不丢关键信息
- 未真实运行多 agent 协作

## 许可边界

本页的能力分析、结构与说明由 bo老师整理，采用 CC BY 4.0。上游项目、代码、名称和商标继续遵循各自许可。这里不重新发布上游代码。
