# Scientific Agent Skills

用一组科研 Skill 覆盖论文查找、写作、评审、统计和实验设计。

面向多学科科研 Agent 的大型 Skill 集合，首批已拆解六类高频科研能力和安全扫描机制。

- 官网详情：https://bolaoshi.cn/research-tools/scientific-agent-skills
- 原始来源：https://github.com/K-Dense-AI/scientific-agent-skills
- 读取版本：2b4cce7d8763232864fa3dff701e311636bcc62f
- 核对日期：2026-06-15
- 上游许可：MIT
- 验证程度：partially-verified
- 编辑判断：worth-watching

## 适合这些任务

- 希望组合科研 Skill 的 Agent 开发者
- 需要论文查找、统计和实验设计入口的研究者

## 这些情况要谨慎

- 准备一次性安装全部 Skill 的用户
- 缺少安全扫描和人工验收的环境

## 能力拆解

### 1. 同行评审结构化审查

把论文、grant、preprint 或科学演示材料，按初评、逐节审查、方法统计、可复现、图表、伦理和写作质量生成结构化评审意见。

证据：source-reviewed

输入

- 待审 manuscript、grant、preprint 或 presentation。
- 目标 venue 或评审标准。
- 可选报告指南：CONSORT、STROBE、PRISMA、ARRIVE、MIAME、MINSEQE 等。

输出

- summary statement。
- major comments，按严重度和可修复性排序。
- minor comments。

人工检查

- 目标 venue 的审稿标准是否已知。
- 哪些问题足以构成 reject 或 major revision。
- 评论是否要求了超出论文范围的新实验。

### 2. 多数据库论文查找

把用户的论文查找、DOI 查询、开放获取全文、作者论文、引用图谱或预印本需求，路由到合适的学术数据库，并返回可追溯的原始 API 结果和查询记录。

证据：runtime-verified

输入

- 用户自然语言问题，例如找某主题论文、查 DOI、找开放获取 PDF、找作者论文、查引用关系。
- 可选论文标识符：DOI、PMID、PMCID、arXiv ID、OpenAlex ID、Semantic Scholar ID、ORCID、ISSN。
- 可选环境变量：NCBIAPIKEY、COREAPIKEY、S2APIKEY、OPENALEXAPIKEY。

输出

- Databases Queried 清单，包含数据库名、endpoint 和查询意图。
- 每个数据库的原始 JSON 或解析后的 XML 结果。
- 失败库、空结果和替代尝试说明。

人工检查

- 用户要的是“查到候选论文”，还是“证明某个主张有依据”。
- 多库结果是否需要去重和排序。
- 多库去重是否允许用标题弱匹配，是否需要人工复核。

### 3. 实验设计与随机化布局

把研究问题、处理条件、实验单位、干扰因素和运行约束，转换成可审计的设计结构、随机化 schedule、运行布局、质量门和后续 power 与 analysis 交接。

证据：runtime-verified

输入

- 研究问题、response、处理条件和控制条件。
- 实验单位、随机化单位、重复单位和分析单位。
- 样本或实验单元数量。

输出

- 研究设计对象。
- allocation schedule 或 DOE layout。
- arm balance、block balance、run order 和 layout status。

人工检查

- 实验单位和处理施加层级是否真实一致。
- 技术重复是否被误当作独立重复。
- 哪些 nuisance factor 必须 block，哪些只能记录为 covariate。

### 4. 科研写作结构化表达

把科研材料、文献线索、图表计划和目标期刊要求，转成符合 IMRAD、引用规范、报告指南和段落化表达要求的科学写作产物。

证据：source-reviewed

输入

- 研究问题、目标期刊、文章类型和目标读者。
- 已有数据结果、图表、方法说明、研究限制和参考文献。
- research-lookup 或其他文献检索能力产出的文献与要点。

输出

- 科学论文段落、摘要、Introduction、Methods、Results、Discussion、投稿稿。
- 研究报告或白皮书 LaTeX 结构。
- 图形计划、graphical abstract、schematic、figure caption。

人工检查

- 哪些主张必须有真实来源支持。
- 目标期刊是否允许非结构化摘要。
- 图像是否真的需要生成，还是只需要图表计划。

### 5. 统计分析与结果报告

把已有研究数据、变量类型、研究问题和报告要求，转换成检验选择、假设诊断、统计结果、效应量、解释边界和 APA 风格报告。

证据：runtime-verified

输入

- 研究问题和假设。
- 数据表，至少包含 outcome、predictor 或 group 列。
- 变量类型：continuous、binary、categorical、ordinal、count、time-to-event。

输出

- 数据状态对象。
- 检验选择路径和 selected test。
- 假设诊断结果。

人工检查

- 研究问题和变量类型是否被正确理解。
- 分组是否独立，是否存在配对、重复测量或聚类。
- 缺失机制和排除规则是否需要人工确认。

### 6. 统计功效与样本量计算

把研究设计、效应量、显著性水平、目标功效和样本量约束，转换成样本量、达到功效、最小可检测效应、功效曲线或模拟功效估计。

证据：runtime-verified

输入

- 研究设计和计划分析，例如 two independent means、paired mean、ANOVA、two proportions、correlation、chi-square、linear regression。
- 三个已知量：样本量、效应量、alpha、power 中的任意三个。
- test 特定参数，例如 kgroups、prop1、prop2、dof、dfnum、ktotal、ratio、alternative。

输出

- required sample size，例如 npergroup 或 total n。
- achieved power。
- minimum detectable effect。

人工检查

- effect size 是否来自 SESOI、收缩后的先验估计或领域阈值。
- 计划分析是否和 power 方法一致。
- 是否需要 one-sided test，以及是否有合理说明。

## 处理机制

1. **skill市场准入与安全扫描**：解释 scientific-agent-skills 作为大型 skill 集合，怎样通过目录结构、frontmatter、版本、安装路径、贡献规范和安全扫描，把单个 skill 变成可分发、可审查、可安装的 agent 能力单元。

## 开始方式

1. 先运行安全扫描
2. 只选择一个明确任务的 Skill
3. 用最小输入检查输出合同

## 环境要求

- 支持 Skill 的 Agent 环境
- Python
- 按具体 Skill 准备数据或 API

## 关键限制

- 当前只拆解大型集合中的六类能力
- 不同 Skill 的依赖和质量差异明显
- 外部模型和数据库需要独立核验

## 已核对范围

- 只读上游仓库 2b4cce7
- peer-review/SKILL.md
- commonissues.md
- reportingstandards.md
- generateschematic.py
- SECURITY.md 中 peer-review 的 critical 摘要线索
- 无外部 API 的评审判断合同 dry run
- 自造 manuscript 样本的 major、minor、question 和 final recommendation

## 仍待核对

- 未审阅真实论文或 grant 样本
- 未运行 presentation PDF 转图审阅流程
- 未调用示意图生成脚本
- 未验证评审意见和原文行号映射
- 未验证不同 venue 的 reviewer expectations
- 未验证原始数据、代码和伦理文件审查
- 未验证 API key 模式
- 未验证 rate limit 和 429 重试

## 许可边界

本页的能力分析、结构与说明由 bo老师整理，采用 CC BY 4.0。上游项目、代码、名称和商标继续遵循各自许可。这里不重新发布上游代码。
