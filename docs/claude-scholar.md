# claude-scholar

半自动化科研助手，用证据契约机制贯穿选题、实验分析、写作到审稿响应

专为计算机科学和AI/ML研究者设计的半自动化助手，人不做决策替代、助手加速周围工作。它的核心是一套证据契约机制，确保论文中每条结论都能追溯到实验证据或文献支撑。

- 官网详情：https://bolaoshi.cn/research-tools/claude-scholar
- 原始来源：https://github.com/Galaxy-Dawn/claude-scholar
- 读取版本：source-reviewed
- 核对日期：2026-06-19
- GitHub Stars：4,573（2026-07-11）
- 上游许可：MIT
- 验证程度：catalogued
- 编辑判断：worth-watching

## 什么时候使用

- 需要把模糊研究想法变成有证据支撑的选题方案
- 想在实验结果分析前先锁定度量标准和比较框架
- 写ML论文时要求每句结论都能追溯到实验证据
- 收到审稿意见后需要逐条对应论文证据来组织回复

## 先准备什么

- 准备好研究问题和现有的实验数据或文献笔记
- 安装并配置Zotero及Zotero MCP用于文献管理
- 明确目标会议或期刊的模板和格式要求

## 它会怎样推进

1. **选题构思**：从模糊兴趣出发，做5W1H头脑风暴和gap分析，形成带SMART问题的研究卡
2. **文献检索入Zotero**：多源搜索论文，按DOI/arXiv优先级过滤，导入Zotero并附加PDF全文
3. **实验分析**：先锁定主问题、度量、单位和比较族，再跑严格统计和生成科研图表
4. **论文写作**：每句结论追溯Claim Candidate或Evidence Record，不支持的结论显式标记
5. **审稿响应**：拆审稿意见为子问题，分类后逐条给出带论文位置/表格/图的证据锚点

## 第一次这样开始

帮我安装这个库：https://github.com/Galaxy-Dawn/claude-scholar
安装前先告诉我需要什么环境，安装后帮我跑通一个最小示例。

## 结果出来后先看什么

- 论文中每个claim是否都有对应的实验证据或文献支撑
- 统计结果是否报告了95%置信区间、效应量和多重比较校正
- 引用是否经过DOI/arXiv/CrossRef逐条验证
- 审稿回复中每条response是否有具体的论文位置锚点

## 常见误区

- 把claim strength从speculative静默升级为supported而不补证据
- 用abstract-only来源支撑论文正文的耐用结论
- 在证据不足时强行推进到proposal阶段而非停在问题卡

## 能力拆解

### 1. ML论文写作与引文校验

把研究仓库证据/草稿转成会议级论文草稿（NeurIPS/ICML/ICLR/ACL/AAAI/COLM），每句正文经 Claim Ledger gate 追溯到 Claim Candidate 或 Evidence Record，保留允许/禁止措辞，不支持的 claim 显式标记而非润色。配套引文校验：写作时即时验证，权威源优先（DOI/arXiv/CrossRef/Semantic Scholar），AI 生成引文约 40% 错误率必须逐条验证。

证据：source-reviewed

输入

- 研究仓库 / 草稿 / 实验证据 / 文献笔记。

输出

- 面向 NeurIPS/ICML/ICLR/ACL/AAAI/COLM 的论文草稿。
- 引文校验报告。

人工检查

- 核对结果是否符合当前研究问题和证据边界

### 2. 发表级图表生产

把科学沟通目标 + 数据形状转成发表级 pubfig 图和 pubtab 表，经环境探针（自动安装 pubfig/pubtab）、任务分类（figure/table/mixed × draft/final/revision）、画图前锁证据契约（primary claim/unit/metric/误差棒基础/winner 主张许可）、选表示族、工具链映射、最小可运行代码生成。失败时降级为 spec/pseudocode。

证据：source-reviewed

输入

- 科学沟通目标 + 数据形状 + 可选现有资产。

输出

- pubfig 图 + pubtab 表（含文件名/格式/导出/QA/修订计划）。

人工检查

- 核对结果是否符合当前研究问题和证据边界

### 3. 实验结果分析BlockerFirst

把原始实验产物（CSV/JSON/TSV/logs/checkpoints/seeds）转成带统计严谨性的分析 bundle（analysis-report + stats-appendix + figure-catalog + figures），经 Blocker-First Gate 锁定 primary question/metric/unit/seeds/provenance/comparison，quarantine 自相矛盾的统计文件，严格统计（描述统计 + 95%CI + 显著性检验 + 效应量 + 多重比较校正），产出真实科研图和 claim candidates。这是该仓库的核心差异化能力。

证据：source-reviewed

输入

- 实验目录（CSV/JSON/TSV/logs/checkpoints/seeds）。

输出

- 分析 bundle：analysis-report.md + stats-appendix.md + figure-catalog.md + figures/。

人工检查

- read-only audit mode 是显式人工确认点：证据缺失时只出 blockers 不强行产出。

### 4. 审稿响应与逐点rebuttal

把审稿意见转成证据锚定的逐点 rebuttal：拆 atomic objections → 分类（Major/Minor/Typo/Misunderstanding）→ 选策略（Accept/Defend/Clarify/Experiment）→ 每条 response 必须含 evidence anchor（paper location/result table/figure/citation/ER ID）→ 包就绪度分级（ready/draft/placeholders/blocked）。红线：不忽略任何审稿意见、不改写 reviewer 原话、不编造实验/引文/行号。

证据：source-reviewed

输入

- 审稿意见文件/文本 / editor decision letter / reviewer comments / 修改记录。

输出

- rebuttal.md + 可选 review-analysis.md + experiment-plan.md。
- Nature 系：Response strategy + Comment-response tracker + 逐点响应信 + Manuscript change checklist + Missing info/risk flags。

人工检查

- 包就绪度分级是显式人工确认点。

### 5. 文献检索与Zotero集成

把研究主题转成结构化 Zotero 集合（Core/Methods/Applications/Baselines/To-Read 五子集合）+ 带证据标签的结构化综述，经 WebSearch 多源检索、源质量过滤（DOI/arXiv/PDF 优先，abstract-only 降级）、Zotero 导入、PDF 级联附加（landing-page→direct PDF→Unpaywall）、全文读取、主题分类、两步去重、gap 分析。

证据：source-reviewed

输入

- 研究主题（+ 可选 scope/outputtype）。

输出

- Zotero 集合树（五子集合）。
- research-question-card.md。
- 可选 literature-review.md / research-proposal.md / references.bib。

人工检查

- 核对结果是否符合当前研究问题和证据边界

### 6. 选题构思与证据契约

把一个模糊研究主题/兴趣转成带证据约束的研究问题卡（Research Question Card），经 5W1H 头脑风暴、文献检索、gap 分析（文献/方法/应用/跨学科/时间 5 类）、SMART 问题构造、方法选择。核心门控：若证据不足，Proposal Readiness Gate 禁止生成 proposal，只产出 question-card + gap note。

证据：source-reviewed

输入

- 一个模糊研究主题/兴趣。

输出

- research-question-card.md（始终生成）。
- literature-review.md（仅当证据充分）。
- research-proposal.md（仅当通过 Proposal Readiness Gate）。

人工检查

- Proposal Readiness Gate 是显式人工确认点：决定是否进入 proposal。

## 处理机制

1. **证据契约与claim强度门控**：claude-scholar 的多张能力卡共享一条证据约束链：从模糊课题经 Proposal Readiness Gate 进入研究，文献经源质量过滤和 Evidence Record 编号，实验结果经 Blocker-First Gate 和 Claim Candidate 构造，claim 经 Claim Promotion Gate 进入写作/综述/rebuttal，四级 claim strength 禁止静默升级。本机制卡说明这些对象如何跨能力传递，以及哪些状态会阻断或降级。

## 开始方式

1. 选题构思：从模糊兴趣出发，做5W1H头脑风暴和gap分析，形成带SMART问题的研究卡
2. 文献检索入Zotero：多源搜索论文，按DOI/arXiv优先级过滤，导入Zotero并附加PDF全文
3. 实验分析：先锁定主问题、度量、单位和比较族，再跑严格统计和生成科研图表
4. 论文写作：每句结论追溯Claim Candidate或Evidence Record，不支持的结论显式标记
5. 审稿响应：拆审稿意见为子问题，分类后逐条给出带论文位置/表格/图的证据锚点

## 环境要求

- 准备好研究问题和现有的实验数据或文献笔记
- 安装并配置Zotero及Zotero MCP用于文献管理
- 明确目标会议或期刊的模板和格式要求

## 关键限制

- 把claim strength从speculative静默升级为supported而不补证据
- 用abstract-only来源支撑论文正文的耐用结论
- 在证据不足时强行推进到proposal阶段而非停在问题卡

## 已核对范围

- 静态确认 ml-paper-writing 的 Claim Ledger gate 和六套会议模板
- 静态确认 citation-verification 的权威源优先序和 API 客户端
- 静态确认 nature-writing 的论证先行架构
- 静态确认 publication-chart-skill 的环境探针和证据契约锁定
- 静态确认表示族分类（comparison/ablation/distribution/relationship/trend/diagnostic/composition/table）
- 静态确认依赖 pubfig/pubtab 自动安装和降级路径
- 静态确认 results-analysis 的 Blocker-First Gate 和 read-only audit mode
- 静态确认 Claim Candidate 四级 claim strength 和 quarantine 机制

## 仍待核对

- 未真实运行论文写作
- 未验证引文 API 客户端实际可调通性
- 未真实运行图表生成
- 未验证 pubfig/pubtab 在无网络环境的降级质量
- 未真实运行 results-analysis
- 未验证真实科研图的生成质量
- 未真实运行审稿响应
- 未确认 nature-response tests/rubric.md 是否被 CI 执行

## 许可边界

本页的能力分析、结构与说明由 bo老师整理，采用 CC BY 4.0。上游项目、代码、名称和商标继续遵循各自许可。这里不重新发布上游代码。
