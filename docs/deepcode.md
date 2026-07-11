# DeepCode

从论文或需求文本自动生成可复现代码的智能编程工具

面向需要复现论文算法或从文本需求生成代码的研究者和开发者。它把论文、文档或自然语言描述转成实现计划、搜索参考仓库生成代码索引，再逐文件写入代码目录，适合从零搭建论文代码的场景。

- 官网详情：https://bolaoshi.cn/research-tools/deepcode
- 原始来源：https://github.com/HKUDS/DeepCode
- 读取版本：5fbfa27f206b4d310a57007e0ad62d2f911171fb
- 核对日期：2026-06-16
- GitHub Stars：16,025（2026-07-11）
- 上游许可：MIT
- 验证程度：partially-verified
- 编辑判断：worth-watching

## 什么时候使用

- 拿到一篇论文需要从头写出可运行的复现代码
- 论文太长需要先分段提取算法和实验描述再生成实现计划
- 想在写代码前发现相关的开源参考仓库作为实现参考
- 需要人工审核实现计划后再自动生成代码

## 先准备什么

- 准备论文PDF或Markdown文件，或直接用文本描述需求
- 可选提供目标参考仓库的线索以辅助代码生成
- 运行环境需能访问LLM API以驱动代码生成循环

## 它会怎样推进

1. **输入归一化**：把论文URL、本地PDF或文本需求统一转为任务工作区中的Markdown材料
2. **分段规划**：长论文自动分段，按代码实现相关性从中选取关键片段作为规划上下文
3. **生成实现计划**：从论文内容生成包含文件结构、组件和验证策略的YAML实现计划
4. **人工审核**：展示实现计划，等待研究者确认、修改或取消后再进入代码生成
5. **逐文件写代码**：按计划逐步调用工具写入代码文件，用记忆压缩保持长循环上下文稳定

## 第一次这样开始

帮我安装这个库：https://github.com/HKUDS/DeepCode
安装前先告诉我需要什么环境，安装后帮我跑通一个最小示例。

## 结果出来后先看什么

- 实现计划是否覆盖了论文的核心算法、数据处理和评估逻辑
- 生成的代码目录结构是否与计划一致，未实现文件是否被明确列出
- 代码是否至少能运行到不报语法错误，关键函数是否对应论文公式
- 参考仓库索引建立的关系是否真实可查，而非LLM幻觉

## 常见误区

- 跳过人工审核环节，让低质量实现计划直接进入代码生成
- 把生成代码能跑等同于论文结果可复现而不验证数值一致性
- 将参考仓库的代码片段直接当作论文的官方实现使用

## 能力拆解

### 1. MCP驱动代码生成与记忆压缩迭代

把通过 review 的 initialplan.txt 交给代码实现循环，由 LLM 调用 MCP 工具写入目标文件，并在长循环中用 concise memory、文件摘要和未完成文件追踪压缩上下文，最后区分完整完成、提前停止、警告和错误。

证据：source-reviewed

输入

- initialplan.txt
- task 目录和 generatecode 目标目录。
- 可选 indexes 参考代码索引。

输出

- MCP驱动代码生成与记忆压缩迭代的结构化结果

人工检查

- 核对结果是否符合当前研究问题和证据边界

### 2. 参考仓库发现与代码索引检索

在启用 indexing 的完整模式下，先从论文参考文献或相关工作中发现可能有帮助的 GitHub 仓库，再下载到 task 目录下的 codebase，随后为每个仓库生成结构化 JSON index。代码实现阶段可按目标文件和关键词检索相关参考代码、关系和实现建议。

证据：source-reviewed

输入

- reference.txt 或 reference analysis result。
- initialplan.txt 中的目标文件树。
- codebase 目录中的一个或多个参考仓库。

输出

- 参考仓库发现与代码索引检索的结构化结果

人工检查

- 核对结果是否符合当前研究问题和证据边界

### 3. 多入口任务创建与会话状态治理

把 Web、CLI、文件、URL 和 chat 需求统一成可追踪任务，并把任务挂到 session。任务运行过程中持续更新状态、进度、待用户确认、取消、完成、错误和中断状态，使前端、CLI 和本地 JSONL 存储看到同一条任务轨迹。

证据：source-reviewed

输入

- PaperToCodeRequest：inputsource、inputtype、enableindexing、enableuserinteraction、sessionid。
- ChatPlanningRequest：requirements、enableindexing、enableuserinteraction、sessionid。
- CLI 参数：--file、--url、--chat、--requirement、--session、--session-title。

输出

- 多入口任务创建与会话状态治理的结构化结果

人工检查

- 核对结果是否符合当前研究问题和证据边界

### 4. 大文档分段与规划上下文压缩

当论文或文档过长时，先判断是否需要分段，再调用 document-segmentation 工具生成文档结构和 segment index。规划阶段按 codeplanning 相关性挑选少量片段，把超长全文压缩成面向代码复现计划的上下文。

证据：source-reviewed

输入

- paper.md 或转换后的 Markdown。
- deepcodeconfig.json 中的 documentSegmentation.enabled 和 sizeThresholdChars。
- documentsegments/documentindex.json。

输出

- 大文档分段与规划上下文压缩的结构化结果

人工检查

- 核对结果是否符合当前研究问题和证据边界

### 5. 输入归一化与任务工作区准备

把用户输入的 URL、本地文件路径、file:// 路径、Markdown 文档或 chat 生成文本整理成一个任务工作区，并在工作区内派生后续阶段共用的路径、MCP 文件系统访问范围、日志上下文和任务目录。

证据：source-reviewed

输入

- rawinput：URL、本地路径、file:// 路径或相对路径。
- enableindexing：后续是否需要参考分析和索引。
- taskkind：paper2code 或 chat2code。

输出

- 输入归一化与任务工作区准备的结构化结果

人工检查

- 核对结果是否符合当前研究问题和证据边界

### 6. 需求澄清到YAML实现计划

把论文全文、分段上下文或用户自然语言需求转成实现计划，并在进入代码生成前用必需章节校验、重试、计划版本记录和人工 review 门控来降低低质计划直接执行的风险。

证据：source-reviewed

输入

- paper.md 或分段上下文。
- chat 模式用户需求文本。
- requirement analysis 的用户回答或 skip。

输出

- 需求澄清到YAML实现计划的结构化结果

人工检查

- 核对结果是否符合当前研究问题和证据边界

## 处理机制

1. **科研代码生成对象与状态链**：DeepCode 的多张能力卡共享一条对象链：外部材料先进入 task 和 session，再变成受控工作区、分段文档、实现计划、参考仓库索引、生成代码和实现报告。本机制卡说明这些对象如何跨阶段传递，以及哪些状态会阻断或降级。

## 开始方式

1. 输入归一化：把论文URL、本地PDF或文本需求统一转为任务工作区中的Markdown材料
2. 分段规划：长论文自动分段，按代码实现相关性从中选取关键片段作为规划上下文
3. 生成实现计划：从论文内容生成包含文件结构、组件和验证策略的YAML实现计划
4. 人工审核：展示实现计划，等待研究者确认、修改或取消后再进入代码生成
5. 逐文件写代码：按计划逐步调用工具写入代码文件，用记忆压缩保持长循环上下文稳定

## 环境要求

- 准备论文PDF或Markdown文件，或直接用文本描述需求
- 可选提供目标参考仓库的线索以辅助代码生成
- 运行环境需能访问LLM API以驱动代码生成循环

## 关键限制

- 跳过人工审核环节，让低质量实现计划直接进入代码生成
- 把生成代码能跑等同于论文结果可复现而不验证数值一致性
- 将参考仓库的代码片段直接当作论文的官方实现使用

## 已核对范围

- 读取 indexed code implementation workflow、CodeImplementationAgent、concise memory agent、tool definitions 和 implementation MCP server
- 静态确认 writefile、searchcodereferences、workspace path validation、memory summary 和 incomplete 状态
- 读取 reference analyzer、GitHub download、codebase index workflow、CodeIndexer 和 code-reference-indexer MCP
- 静态确认 reference analysis、repo download、index JSON、relationship 和 searchcodereferences 路径
- 读取 UI API、WorkflowService、WebSocket、CLI session 命令和本地 JSONL session store
- 静态确认 taskid、sessionid、pendinginteraction、status、progress、result、error 的读写路径
- 读取 documentSegmentation 配置、segmentation agent、MCP server 和 planner 读取分段上下文的逻辑
- 静态确认大文档阈值、segment index、relevance score 和 fallback 分支

## 仍待核对

- 未运行代码生成循环
- 未调用真实 LLM 或 MCP 工具
- 未验证生成代码可执行性
- 未调用 GitHub 下载 MCP
- 未运行 LLM 文件分析和关系识别
- 未验证 reference index 对代码生成的帮助
- 未启动 FastAPI 和 WebSocket
- 未运行 CLI 交互

## 许可边界

本页的能力分析、结构与说明由 bo老师整理，采用 CC BY 4.0。上游项目、代码、名称和商标继续遵循各自许可。这里不重新发布上游代码。
