# PaperQA

从本地论文中检索证据并生成可以回到来源的带引用答案。

连接文档索引、候选检索、证据汇聚、重排和带引用回答，部分主链已经完成本地小样本验证。

- 官网详情：https://bolaoshi.cn/research-tools/paper-qa
- 原始来源：https://github.com/Future-House/paper-qa
- 读取版本：d7675d7
- 核对日期：2026-06-12
- 上游许可：Apache-2.0
- 验证程度：verified
- 编辑判断：recommended

## 适合这些任务

- 需要基于本地论文问答的研究者
- 重视引用和证据回查的团队

## 这些情况要谨慎

- 希望模型直接替代文献判断的用户
- 无法提供模型和 embedding 的环境

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

1. **核心处理链**：从本地论文中检索证据并生成可以回到来源的带引用答案。

## 开始方式

1. 准备少量可合法使用的本地论文
2. 使用测试模型先验证索引和引用链
3. 再接入真实模型评估语义排序

## 环境要求

- Python
- LLM 和 embedding
- 本地论文文件

## 关键限制

- 真实外部模型表现仍需核验
- 中文引用表现未完成验证
- 引用格式不能代替证据质量判断

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
