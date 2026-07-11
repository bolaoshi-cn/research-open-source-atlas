# Auto-Empirical-Research-Skills

面向实证研究的agent skill发行仓，含完整pipeline、质量门和许可审计

面向经济学和社会科学实证研究者。提供四种技术栈的完整实证pipeline skill、AER投稿与复现skill栈，以及数值benchmark、行为eval和来源许可审计等质量基础设施。

- 官网详情：https://bolaoshi.cn/research-tools/auto-empirical-research-skills
- 原始来源：https://github.com/brycewang-stanford/Auto-Empirical-Research-Skills
- 读取版本：7f52b28bb58b78a7719f76ccbe63b1fadea9c228
- 核对日期：2026-06-16
- GitHub Stars：2,790（2026-07-11）
- 上游许可：CC-BY-SA-4.0
- 验证程度：catalogued
- 编辑判断：worth-watching

## 什么时候使用

- 需要从数据清洗到表图输出的完整实证分析pipeline时
- 准备向AER/AER:Insights/AEJ投稿并需要复现包和审稿回复指导时
- 需要在大规模skill目录中检索适用工具并检查来源许可时

## 先准备什么

- 准备研究问题、数据路径、变量含义和处理组/结果变量定义
- 选择技术栈：StatsPAI DSL、Python、Stata或R
- 明确研究设计类型（DiD/IV/RDD/SCM等）和输出目标（表格/图形/复现包）

## 它会怎样推进

1. **栈选择**：根据项目需求和合作者偏好选择StatsPAI/Python/Stata/R技术栈
2. **八步执行**：依次完成清洗、变量构造、描述统计、诊断、基准估计、稳健性、机制异质性和表图输出
3. **数值验证**：用小型benchmark检查pipeline能否发现常见因果推断陷阱
4. **行为评审**：用rubric检查agent输出的弱IV误判、TWFE误用、引用编造等高成本错误
5. **许可审计**：检查所使用skill的来源许可和商业使用兼容性

## 第一次这样开始

帮我安装这个库：https://github.com/brycewang-stanford/Auto-Empirical-Research-Skills
安装前先告诉我需要什么环境，安装后帮我跑通一个最小示例。

## 结果出来后先看什么

- 统计结果是否通过benchmark的数值重算验证
- agent输出是否触发行为eval的红线（弱IV误判、TWFE误用等）
- skill的来源许可证是否允许在当前场景使用
- 输出格式是否满足目标期刊要求

## 常见误区

- 把skill推荐的统计默认值直接用于真实研究而不按研究设计复核
- 把hygiene score（结构分数）等同于skill的行为正确性
- 忽略CC BY-SA许可的署名和相同方式共享义务

## 能力拆解

### 1. AER投稿与复现skill栈

把 AER、AER: Insights 和 AEJ 论文从选题、识别、稳健性、引言、表图、复现包、提交到 R&R 的流程拆成可路由的专门 skill。

证据：source-reviewed

输入

- 新研究项目、论文草稿、识别策略、主结果、表图、复现包、投稿材料或审稿意见。
- 目标期刊：AER、AER: Insights 或 AEJ。
- 作者已经拥有的材料状态，例如数据、代码、表图、README、cover letter 或 reviewer reports。

输出

- 下一步 skill、理由和输入清单。
- 识别审查、复现包结构、投稿前检查或审稿回复草案。

人工检查

- 目标期刊和投稿策略。
- 数据和代码是否允许公开。
- 识别假设是否可以承担顶刊审稿。

### 2. CI质量门与同步治理

用本地 Makefile 和 GitHub Actions 把 catalog 新鲜度、repo 结构、workflow 安全策略、Python 兼容、unit tests、eval harness、benchmark 和 upstream skill 同步放进一组持续质量门。

证据：source-reviewed

输入

- 本地文件树、generated artifacts、skill catalog、provenance、audit、eval docs。
- Python tooling、tests、eval scenarios、benchmark tasks。
- GitHub Actions event、branch、workflow permission 和 sync source。

输出

- 本地 make 退出码。
- CI job pass/fail。
- sync PR。

人工检查

- sync PR 是否合并。
- workflow policy 变更是否放行。
- failed eval 或 benchmark 是否修改 skill、scenario 还是 candidate。

### 3. skill目录生成与生态索引

把 skills/ 下的 vendored skill 文件收集成机器可读目录，再叠加来源、许可证、taxonomy、搜索页面和结构化 hygiene score，使大量 skill 能按研究阶段、方法、语言和主题检索。

证据：source-reviewed

输入

- skills//SKILL.md
- catalog/provenance.json
- skill frontmatter 中的 name、description 和路径。

输出

- catalog/skills.json
- catalog/skills-enriched.json
- docs/SKILLCATALOG.md

人工检查

- 新 skill 是否真的属于 empirical research。
- 缺 license 或 unknown commercialuse 是否允许收录。
- taxonomy 标签是否需要人工纠偏。

### 4. 完整实证pipeline技能分发

把完整实证研究流程封装成 agent 可调用的技能入口，让用户可以按 StatsPAI DSL、显式 Python、Stata 或 R 四种栈选择同一条实证 pipeline。

证据：source-reviewed

输入

- 用户研究问题、数据路径、变量含义、处理组、结果变量、时间或面板单位。
- 研究设计选择，例如 DID、IV、RDD、SCM、DML、matching、target trial、TMLE 或 CATE。
- 输出目标，例如 Table 1、Table 2、多列回归表、机制表、异质性表、事件研究图、稳健性表和复现包。

输出

- 研究 pipeline 计划。
- 表格、图形、复现包或论文段落。
- 可进入后续质量门的中间对象和未验证范围。

人工检查

- 当前项目该用 DSL 还是显式栈。
- 研究设计和识别假设是否成立。
- 输出是否满足目标期刊和合作者工具链。

### 5. 数值benchmark重算与反作弊评分

用小型、无第三方依赖、可重算真值的实证 benchmark 检查 agent pipeline 是否能发现常见因果推断陷阱，并拒绝把误导性数值当成主结果。

证据：source-reviewed

输入

- 任务 TOML：lalonde-recovery、card-iv-recovery、did-staggered-recovery、rdd-recovery、bad-control-recovery。
- 本地数据或确定性 DGP：LaLonde、Card、staggered DID、RDD、bad-control。
- candidate results.json。

输出

- benchmark scorecard。
- required failures。
- strict gate exit code。

人工检查

- 新 benchmark task 是否有真实或确定性 gold。
- 哪些 check 标成 required。
- 分数门槛是否足以阻断高风险 agent 输出。

### 6. 来源许可与provenance审计

为 vendored skill collection 建立来源、许可证、商业使用状态、source confidence 和同步方式的审计表，降低大规模 skill 收录时的来源不清和授权风险。

证据：source-reviewed

输入

- 顶层 skills/<collection/。
- collection 内的 LICENSE、README、SKILL.md 和 GitHub URL 候选。
- scripts/build-provenance.py 中的 override 表。

输出

- catalog/provenance.json
- docs/LICENSEAUDIT.md
- catalog 中 collection 的 license 和 commercialuse 字段。

人工检查

- 是否允许在目标场景复用 unknown 或 share-alike 内容。
- 是否需要联系 upstream 或排除某个 collection。
- 新增 collection 是否具备足够来源证据。

## 处理机制

1. **skill目录到许可审计链**：这个横向机制解释 AERS 如何把分散的 SKILL.md 文件整理成可检索 catalog，并把 catalog 与 provenance、license、commercialuse 和 sourceconfidence 绑定，形成大规模 skill 入库的治理链。
2. **信任表面与质量门总图**：这个横向机制解释 AERS 如何把 skill 说明、数值 benchmark、行为 eval、repo validator、workflow policy、unit tests 和 CI 组合成多层信任表面，避免大规模目录只停留在数量声明。

## 开始方式

1. 栈选择：根据项目需求和合作者偏好选择StatsPAI/Python/Stata/R技术栈
2. 八步执行：依次完成清洗、变量构造、描述统计、诊断、基准估计、稳健性、机制异质性和表图输出
3. 数值验证：用小型benchmark检查pipeline能否发现常见因果推断陷阱
4. 行为评审：用rubric检查agent输出的弱IV误判、TWFE误用、引用编造等高成本错误
5. 许可审计：检查所使用skill的来源许可和商业使用兼容性

## 环境要求

- 准备研究问题、数据路径、变量含义和处理组/结果变量定义
- 选择技术栈：StatsPAI DSL、Python、Stata或R
- 明确研究设计类型（DiD/IV/RDD/SCM等）和输出目标（表格/图形/复现包）

## 关键限制

- 把skill推荐的统计默认值直接用于真实研究而不按研究设计复核
- 把hygiene score（结构分数）等同于skill的行为正确性
- 忽略CC BY-SA许可的署名和相同方式共享义务

## 已核对范围

- 读取 README 的 AER-Skills 总览
- 读取 aer-workflow 的路由顺序和 handoff contract
- 读取 aer-identification 与 aer-replication 的判断和产物要求
- 读取 Makefile 的 catalog、validate、check、benchmark、eval 入口
- 读取 docs/QUALITYGATE.md 和 docs/TRUST.md 的质量门说明
- 读取 .github/workflows 中 validate、quality、sync、scorecard 和 link check workflow
- 运行上游 scripts/validate-repo.py
- 读取 catalog/skills.json summary

## 仍待核对

- 未检查全部 9 个 AER skill 的每个资源文件
- 未按真实 AER 稿件运行审稿或复现包流程
- 未验证 2026 年 AEA 政策实时页面
- 未运行完整 make check
- 未触发 GitHub Actions
- 未验证 sync workflow 会实际开 PR
- 未重新生成 catalog
- 未人工审查 1080 个 skill 的语义质量

## 许可边界

本页的能力分析、结构与说明由 bo老师整理，采用 CC BY 4.0。上游项目、代码、名称和商标继续遵循各自许可。这里不重新发布上游代码。
