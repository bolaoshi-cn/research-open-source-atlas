# ToolUniverse

把1000+科学工具统一封装成LLM可调用的标准协议，用一次对话发现并执行它们

适用于需要频繁调用不同科学数据库、API和分析工具的研究者。先用自然语言描述研究问题，工具自动从1000+科学工具中发现最相关的几个，调用并返回结构化结果，省去手动查阅每个工具文档的步骤。

- 官网详情：https://bolaoshi.cn/research-tools/tooluniverse
- 原始来源：https://github.com/mims-harvard/ToolUniverse
- 读取版本：source-reviewed
- 核对日期：2026-06-19
- GitHub Stars：1,561（2026-07-11）
- 上游许可：Apache-2.0
- 验证程度：catalogued
- 编辑判断：worth-watching

## 什么时候使用

- 需要从多个科学数据库（文献、基因、蛋白质、化合物、临床数据等）中交叉查询信息时
- 在LLM驱动的科研agent中需要统一工具调用协议，不想为每个API单独编写调用代码时
- 工具太多超出LLM上下文窗口，需要一个中间层自动发现和选择合适工具时
- 想通过MCP协议把科学工具暴露给Claude Desktop等AI客户端使用时

## 先准备什么

- 准备各科学数据库的API密钥（如NCBI、UniProt、OpenAI等），填入api_keys_catalog
- 确定研究问题所属领域，以便配置工具分类过滤和检索范围
- 安装Python环境并通过pip安装tooluniverse包及其依赖
- 如需LLM辅助工具发现，准备OpenAI/Azure OpenAI/Gemini等兼容API密钥

## 它会怎样推进

1. **注册工具**：工具启动时自动扫描预定义工具清单和用户自定义工具，构建统一注册表
2. **描述研究问题**：用自然语言描述你想完成的研究任务，如'查找与阿尔茨海默病相关的基因变异'
3. **发现候选工具**：LLM从注册表中检索最相关的工具，返回工具名、相关度评分和推荐理由
4. **查看工具参数**：查看候选工具的完整参数schema，确认需要提供哪些输入
5. **执行工具调用**：传入参数执行工具，结果经缓存检查后调用外部API，支持批量并行执行
6. **核对返回结果**：检查返回的结构化数据是否完整、参数是否正确映射

## 第一次这样开始

帮我安装这个库：https://github.com/mims-harvard/ToolUniverse
安装前先告诉我需要什么环境，安装后帮我跑通一个最小示例。

## 结果出来后先看什么

- 检查工具发现的候选列表是否与你的研究问题相关（核对relevance_score和reasoning）
- 检查工具返回的结构化数据是否完整，字段是否按schema正确强转类型
- 对于批量执行场景，检查是否有单独失败的任务（工具会隔离失败，不中断其他任务）
- 检查缓存是否命中了之前的相同查询（避免重复调用产生API费用）

## 常见误区

- 认为所有注册的工具都实际可用（部分工具加载时已失败并记入_TOOL_ERRORS，需运行tu doctor核查）
- 把compact mode的5个元工具当作全部能力（它们只是发现入口，背后才有1000+真实工具）
- 忽略LLM工具发现的选择质量差异（LLM finder与embedding finder在不同领域上召回率不同）
- 不检查工具返回结果的真实性就直接采信（工具只是中间件，结果质量取决于上游数据库）

## 能力拆解

### 1. MCP服务桥接

把 ToolUniverse 全部工具（或 compact mode 的 5 个元工具）通过标准 MCP 协议暴露给任意 MCP 客户端（Claude Desktop、Cursor、其他 agent），并把外部 MCP server 的工具反向拉进 ToolUniverse 注册表，实现双向桥接。

证据：source-reviewed

输入

- MCP 客户端的 tools/list、tools/call 请求（正向桥接）。
- 外部 MCP server 配置（反向 auto-loader）。

输出

- MCP 客户端可调用的标准 MCP 工具（正向）。
- 注册表内新增的外部 MCP 工具（反向）。

人工检查

- 核对结果是否符合当前研究问题和证据边界

### 2. compact-mode元工具发现与执行

把 1000+ 工具压缩成 5 个元工具（listtools、greptools、gettoolinfo、findtools、executetool），让 LLM 先用元工具发现真实工具，再查看参数，最后执行，从而节省约 99% 上下文窗口。

证据：source-reviewed

输入

- LLM agent 的工具调用（agent 只看到 5 个元工具的 schema）。

输出

- 动态发现并执行的 1000+ 真实工具结果。

人工检查

- 核对结果是否符合当前研究问题和证据边界

### 3. function-call解析与工具调用执行

把 LLM 产出的 function call（dict 或 llama/openai 格式字符串）解析成工具名和参数，校验 schema，实例化工具，执行并回填结果。支持单步和批量并行执行，单个失败不中断其他。

证据：source-reviewed

输入

- function call JSON {name, arguments} 或 llama/openai 格式字符串。
- 同一回合的多个 function call 列表（批量）。

输出

- 工具 run 返回值（结构化 dict 或 JSON 串）。
- 可选 OpenAI 风格 {role:tool, toolcalls, content} 消息。

人工检查

- 无显式人工确认。

### 4. 两级结果缓存

对工具调用结果做两级缓存（内存 LRU + SQLite 持久化），并用 singleflight 防止并发重复调用同一工具同一参数，加速重复查询并降低外部 API 成本。

证据：source-reviewed

输入

- 工具调用 + 参数。

输出

- 缓存命中结果或工具执行结果。

人工检查

- 核对结果是否符合当前研究问题和证据边界

### 5. 工具注册与清单生成

把分散在 592 个 tools.json 文件、Python @registertool 装饰器、MCP auto-loader 配置和用户自定义文件中的工具定义，扫描、去重、懒加载注册成一个可被 list/grep/find 查询的统一工具字典 alltooldict。

证据：source-reviewed

输入

- data/tools.json：每个文件是一个工具定义数组。
- @registertool 装饰器声明的 Python 工具类。
- MCP auto-loader 配置（txagent、expertfeedback、usptodownloader、boltz、esm 等）。

输出

- self.alltooldict：可被 list/grep/find 查询的统一工具字典。

人工检查

- 无显式人工确认。工具可用性靠运行时 TOOLERRORS 暴露。

### 6. 自然语言工具发现LLM检索

把用户的自然语言问题转成候选工具列表，让 LLM 从 1000+ 工具中选出最相关的若干个，并附上 relevancescore 和 reasoning，解决"工具太多、LLM 上下文装不下"的发现难题。

证据：source-reviewed

输入

- 自然语言 query（用户问题）。
- limit：返回工具数上限。
- 可选 categories：工具分类过滤。

输出

- {selectedtools, toolobjects, toolprompts, totalselected}（JSON 或 prompt 串）。

人工检查

- 核对结果是否符合当前研究问题和证据边界

## 处理机制

1. **异常分类与HTTP重试健康诊断**：tooluniverse 如何统一治理工具调用的错误分类、HTTP 重试退避、工具健康诊断与失效登记、长任务生命周期与可观测。这张卡补充 tooluniverse-机制-统一工具调用编排与发现 只覆盖编排骨架的缺口。
2. **统一工具调用编排与发现**：ToolUniverse 的多张能力卡共享一条对象链：异构工具定义先进入注册表，再通过 compact mode 压缩成元工具，LLM 经发现（list/grep/find）→ 查看（gettoolinfo）→ 执行（executetool）两阶段完成调用，结果经缓存回填。本机制卡说明这些对象如何跨能力传递，以及哪些状态会阻断或降级。

## 开始方式

1. 注册工具：工具启动时自动扫描预定义工具清单和用户自定义工具，构建统一注册表
2. 描述研究问题：用自然语言描述你想完成的研究任务，如'查找与阿尔茨海默病相关的基因变异'
3. 发现候选工具：LLM从注册表中检索最相关的工具，返回工具名、相关度评分和推荐理由
4. 查看工具参数：查看候选工具的完整参数schema，确认需要提供哪些输入
5. 执行工具调用：传入参数执行工具，结果经缓存检查后调用外部API，支持批量并行执行
6. 核对返回结果：检查返回的结构化数据是否完整、参数是否正确映射

## 环境要求

- 准备各科学数据库的API密钥（如NCBI、UniProt、OpenAI等），填入api_keys_catalog
- 确定研究问题所属领域，以便配置工具分类过滤和检索范围
- 安装Python环境并通过pip安装tooluniverse包及其依赖
- 如需LLM辅助工具发现，准备OpenAI/Azure OpenAI/Gemini等兼容API密钥

## 关键限制

- 认为所有注册的工具都实际可用（部分工具加载时已失败并记入_TOOL_ERRORS，需运行tu doctor核查）
- 把compact mode的5个元工具当作全部能力（它们只是发现入口，背后才有1000+真实工具）
- 忽略LLM工具发现的选择质量差异（LLM finder与embedding finder在不同领域上召回率不同）
- 不检查工具返回结果的真实性就直接采信（工具只是中间件，结果质量取决于上游数据库）

## 已核对范围

- 静态确认 SMCP 继承 FastMCP，支持 stdio/HTTP/SSE 多传输
- 静态确认 pyproject.toml 多个 entry point 暴露 MCP/HTTP/stdio server
- 静态确认 MCP auto-loader 可把外部 MCP server 工具拉进注册表
- 静态确认 data/compactmodetools.json 定义 5 个元工具
- 静态确认 list/grep/find/info/execute 两阶段发现执行模式
- 静态确认 runonefunction 解析 fcall → 实例化工具 → schema 校验 → 调 run → 回填结果
- 静态确认批量执行 executefunctioncalllist 含去重合并和 per-tool 并发限流
- 静态确认 lenient 类型强转 coerceargumentstoschema

## 仍待核对

- 未实测 MCP server 与 Claude Desktop/Cursor 的端到端对接
- 未验证 profile 加载和 middleware 行为
- 未实测 compact mode 在真实 LLM agent 中的上下文节省效果
- 未对比 compact mode 与全量工具暴露的调用质量
- 未真实执行任何工具调用
- 未验证 async 工具在 sync 上下文的 asyncio.run 行为
- 未实测缓存命中率
- 未验证 SQLite 持久化缓存的失效和清理策略

## 许可边界

本页的能力分析、结构与说明由 bo老师整理，采用 CC BY 4.0。上游项目、代码、名称和商标继续遵循各自许可。这里不重新发布上游代码。
