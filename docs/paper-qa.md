# paper-qa

从本地论文和文档中检索证据并生成带精确引用的科研答案

适合需要在自有文献库中做高准确度问答的研究者，支持 PDF、Office 文档和 ClinicalTrials.gov 试验数据。先把论文目录建成可复用索引，再对每个问题依次检索候选文献、汇聚证据片段、生成带上下文编号引用的答案。

- 官网详情：https://bolaoshi.cn/research-tools/paper-qa
- 原始来源：https://github.com/Future-House/paper-qa
- 读取版本：d7675d7
- 核对日期：2026-06-12
- GitHub Stars：8,849（2026-07-11）
- 上游许可：Apache-2.0
- 验证程度：partially-verified
- 编辑判断：recommended

## 什么时候使用

- 需要从一批论文或文档中精准回答科研问题，且要求每个主张都有文献引用
- 需要本地索引文献库并在多轮问答中复用同一索引
- 需要搜索 ClinicalTrials.gov 注册试验并将其作为与论文并列的证据类型
- 需要一种能区分“不确定”与“可回答”的科研问答工具

## 先准备什么

- 一个包含论文 PDF、txt、Office 文档或 HTML 的目录
- LLM API 密钥（如 OpenAI）或本地模型配置
- 可选 manifest CSV 文件补充文献的 DOI、标题等元数据
- 运行 paper-qa 的 Python 3.11+ 环境

## 它会怎样推进

1. **建立本地文档索引**：扫描论文目录，解析全文为可检索片段，存入 Tantivy 索引供后续复用
2. **生成检索关键词**：用 LLM 将研究问题改写为多条宽窄混合的关键词查询
3. **汇聚可引用证据**：从命中文档中检索相关片段，用 LLM 围绕问题摘要并打分，形成 scored context
4. **生成带引用答案**：将 top 证据片段格式化为编号引用，让 LLM 生成每个主张都带 context ID 的答案
5. **格式化引用与状态标记**：将 context ID 替换为文献名并生成参考文献列表，标记答案是否确定

## 第一次这样开始

帮我安装这个库：https://github.com/Future-House/paper-qa
安装前先告诉我需要什么环境，安装后帮我跑通一个最小示例。

## 结果出来后先看什么

- 答案中的引用标记是否都能回溯到具体的 context 片段
- 答案状态是 Certain 还是 Unsure——Unsure 不应被手动改为确定
- evidence 片段的 relevance score 是否被用于过滤低质量上下文
- 是否使用了合适的 settings 配置（answer_max_sources、evidence_k 等控制成本和质量）

## 常见误区

- 不改默认配置就开始使用——answer_max_sources、evidence_k 等参数直接影响引用覆盖度和 API 成本
- 把 evidence summary 当原文——context 是 LLM 摘要后的证据片段，研究关键判断仍需回溯原始文本确认
- 没有 manifest 文件时让 LLM 从首段推断 citation 和 DOI——这可能导致文献身份错误，且无法检测撤稿风险
- 忘记索引需要显式 rebuild——源目录增删文件后不重建索引会导致新旧文献混用

## 能力拆解

### 1. 复用向量存储与缓存

把已经解析过的 Doc、Text、embedding 和搜索索引，以 Docs、textsindex、Numpy 向量存储、Qdrant collection 或目录索引的形式复用，减少重复解析、重复 embedding 和重复检索准备。

证据：source-reviewed

输入

- 已解析的 Doc 和 Text 对象。
- embedding 模型输出的向量。
- Docs 对象和可选 textsindex。

输出

- 可复用的 Docs 对象。
- 已填充的 textsindex。
- Numpy 内存向量索引或 Qdrant collection。

人工检查

- 是否允许持久化原文 chunk、embedding 和 metadata。
- 是否把 Qdrant collection 当作长期证据缓存，还是只用于本轮运行。
- 是否固定 index name、collection name 和 settings JSON 路径。

### 2. 建立本地文档索引

把本地论文、文本、Office 文档、HTML 或源码文件，转成可复用的 Tantivy 全文搜索索引和可回载的 Docs 对象，使后续 papersearch、search 和 ask 能从本地索引取候选文档。

证据：source-reviewed

输入

- 本地文件目录或单层文件目录。
- Settings.agent.index.paperdirectory。
- 可选 --index 或 Settings.agent.index.name。

输出

- SearchIndex 对象。
- index/<indexname/index 下的 Tantivy index。
- index/<indexname/docs 下按 body hash 保存的 Docs 对象。

人工检查

- 本轮是否允许调用 LLM 推断 citation 和结构化文献字段。
- 是否提供 manifest，减少标题、DOI 和作者推断不确定性。
- 是否固定 index name，避免配置 hash 变化后出现多个索引。

### 3. 搜索临床试验

把 ClinicalTrials.gov 查询结果转成可进入 Docs 的临床试验文档和搜索元数据，使 agent 能把注册试验作为论文之外的证据类型继续检索、汇聚和回答。

证据：runtime-verified

输入

- agent 生成或用户传入的 ClinicalTrials.gov query。
- 当前 EnvironmentState.docs。
- Settings.agent.searchcount。

输出

- 当前 Docs 中新增的临床试验 Text。
- 临床试验 DocDetails，docname 和 dockey 通常为 NCT ID。
- 一条 search metadata text，记录 query 和 total result count。

人工检查

- 本轮研究问题是否允许把 clinical trial registry 当作 evidence type。
- 是否只搜临床试验，还是允许论文和临床试验混合进入 Docs。
- 是否使用 human readable 文本，还是保留原始 JSON。

### 4. 搜索候选文献

把用户问题或 agent 改写后的查询词，转成一组候选论文或本地文档，并加入当前问答状态。

证据：runtime-verified

输入

- 用户问题。
- agent 生成的一个或多个检索 query。
- 可选年份范围。

输出

- 加入 state.docs 的候选文档。
- 当前检索状态文本。
- 可选的 retrieved paper metadata。

人工检查

- 检索 query 是否保留了原研究问题的核心限制。
- 检索范围是否只包含本轮允许读取的目录。
- 搜索结果是否需要人工排除低相关文档。

### 5. 汇聚可引用证据

把候选文档中的 chunk 文本，转成围绕当前问题的 scored context，并保留 citation 线索。

证据：runtime-verified

输入

- 当前问题或更具体的证据问题。
- 已加入 Docs 的文档和文本 chunk。
- embedding 模型。

输出

- PQASession.contexts。
- 针对当前问题的 best evidence 文本。
- 状态文本中的 evidence count 和 relevant paper count。

人工检查

- context 是否真的回答当前问题。
- context 的 citation 是否能回到原文位置。
- 证据摘要是否压缩掉关键限定。

### 6. 生成带引用答案

把已汇聚的 evidence contexts 组织成带 in-text citations 的答案，并把答案、原始答案和引用格式化结果保存在 session 中。

证据：runtime-verified

输入

- 用户问题。
- PQASession.contexts。
- answer LLM。

输出

- session.answer。
- session.rawanswer。
- session.formattedanswer。

人工检查

- 答案主张是否只覆盖 citation 能支撑的范围。
- formatted answer 是否保留了关键来源。
- used contexts 是否和最终答案主张对应。

## 处理机制

1. **核心处理链**：从本地论文和文档中检索证据并生成带精确引用的科研答案

## 开始方式

1. 建立本地文档索引：扫描论文目录，解析全文为可检索片段，存入 Tantivy 索引供后续复用
2. 生成检索关键词：用 LLM 将研究问题改写为多条宽窄混合的关键词查询
3. 汇聚可引用证据：从命中文档中检索相关片段，用 LLM 围绕问题摘要并打分，形成 scored context
4. 生成带引用答案：将 top 证据片段格式化为编号引用，让 LLM 生成每个主张都带 context ID 的答案
5. 格式化引用与状态标记：将 context ID 替换为文献名并生成参考文献列表，标记答案是否确定

## 环境要求

- 一个包含论文 PDF、txt、Office 文档或 HTML 的目录
- LLM API 密钥（如 OpenAI）或本地模型配置
- 可选 manifest CSV 文件补充文献的 DOI、标题等元数据
- 运行 paper-qa 的 Python 3.11+ 环境

## 关键限制

- 不改默认配置就开始使用——answer_max_sources、evidence_k 等参数直接影响引用覆盖度和 API 成本
- 把 evidence summary 当原文——context 是 LLM 摘要后的证据片段，研究关键判断仍需回溯原始文本确认
- 没有 manifest 文件时让 LLM 从首段推断 citation 和 DOI——这可能导致文献身份错误，且无法检测撤稿风险
- 忘记索引需要显式 rebuild——源目录增删文件后不重建索引会导致新旧文献混用

## 已核对范围

- 真实 paper-qa d7675d7 包
- 本地临时 txt 文档和目录索引
- 本地确定性 stub embedding
- fake agent search query 到 PaperSearch.papersearch
- SearchIndex.query 命中后写入 state.docs
- 空索引停止线
- 本地临时 txt 文档
- 本地 stub summary LLM

## 仍待核对

- 真实外部 LLM 生成检索词的稳定性
- 真实 embedding 语义排序
- 大规模 PDF 索引
- 年份过滤字段行为
- 多文献同名或同 DOI 冲突
- 中文问题检索表现
- 真实 summary LLM 的证据摘要质量
- 真实 embedding 下的 MMR 排序

## 许可边界

本页的能力分析、结构与说明由 bo老师整理，采用 CC BY 4.0。上游项目、代码、名称和商标继续遵循各自许可。这里不重新发布上游代码。
