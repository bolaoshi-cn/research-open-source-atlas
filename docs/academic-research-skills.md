# Academic Research Skills

在Claude Code中将研究、写作、审稿和修订串成可追溯的学术论文全流程

适合需要用AI辅助从研究选题到论文定稿全流程的研究者。它把深度研究、论文写作、多视角审稿和修订闭环拆成独立阶段，每个阶段都有人工确认点和质量门。

- 官网详情：https://bolaoshi.cn/research-tools/academic-research-skills
- 原始来源：https://github.com/Imbad0202/academic-research-skills
- 读取版本：e8ad78f43a57794408ae6aa6913acee12f523012
- 核对日期：2026-06-15
- GitHub Stars：37,312（2026-07-11）
- 上游许可：CC-BY-NC-4.0
- 验证程度：catalogued
- 编辑判断：worth-watching

## 什么时候使用

- 启动一个新研究项目，需要形成结构化的研究问题和方法蓝图
- 撰写学术论文时需要结构论证、引用合规检查和格式化输出
- 论文投稿前需要分视角（方法、领域、前瞻、数据分析）的多维度审稿
- 管理从文献检索到修订再投的完整科研流程，需要全程可追溯

## 先准备什么

- 安装并配置好该skill suite的Claude Code运行环境
- 明确的研究问题或想探索的主题领域
- 目标期刊、引用格式和输出格式偏好（LaTeX/DOCX/PDF）
- 可选：已有文献语料库以支持语料优先检索

## 它会怎样推进

1. **形成研究问题**：选择研究模式，生成研究问题简述和方法蓝图，确认后再进入下一阶段
2. **检索与分析文献**：从语料库或外部来源检索文献，筛选核验后产出跨文献综合报告
3. **撰写论文**：配置论文类型，生成大纲，构建论证链，撰写正文并处理引用合规
4. **多视角审稿**：执行分视角审稿（方法、领域、前瞻、数据分析），产出问题清单和编辑综合决策
5. **修订与验证**：应用修订补丁，验证作者回复，通过引用对齐和完整性核验门
6. **格式化输出**：生成中英文摘要和格式化投稿包（LaTeX/DOCX/PDF），完成合规检查

## 第一次这样开始

帮我安装这个库：https://github.com/Imbad0202/academic-research-skills
安装前先告诉我需要什么环境，安装后帮我跑通一个最小示例。

## 结果出来后先看什么

- 确认研究问题是可回答的，方法论与研究问题匹配，再进入下一阶段
- 检查引用文献真实存在，文中主张有锚点对应，证据链完整
- 审阅综合报告中是否有遗漏的关键文献或跨论文矛盾未处理
- 确认编辑综合决策的严重程度与修订工作量一致，再进入修订循环

## 常见误区

- 跳过提纲确认环节直接生成全文，导致后续需要大面积返工
- 依赖工具自动检索的文献而不手动核实关键引用来源的真实性
- 把LLM审稿评分等同于真实同行评审，忽略工具无法替代的学术判断
- 对依赖领域规范的论文，未提供外部参考标准就直接使用工具

## 能力拆解

### 1. claim-ref对齐审计与终端阻断

把论文中的 claim、引用标记、正文 anchor、文献 corpus、uncited sentence stream 和 Material Passport 策略，转成 claim-ref 对齐结果、未引用断言、claim drift、constraint violation、抽样摘要和终端阻断信号。

证据：source-reviewed

输入

- claimintentmanifests[]：写作阶段声明的 claim intent、planned citation、planned experiment id 和约束。
- literaturecorpus[]：带 citation key、metadata、locator 和可选 excerpt 的文献条目。
- 正文引用标记和 anchor：resolved citation markers、refslug、句子或段落 anchor。

输出

- claimauditresults[]
- uncitedassertions[]
- claimdrifts[]

人工检查

- 是否启用 ARSCLAIMAUDIT=1。
- 是否允许外部 judge model 或 resolver。
- strict terminal policy 是否启用。

### 2. pipeline完整性核验与失败恢复

把论文草稿、修订稿、引用列表、主张、图表、实验来源、Material Passport 和审稿阶段产物，转成完整性报告、阻断项、修复任务、重验结果、最终提交包 gate 和可追踪失败恢复记录。

证据：source-reviewed

输入

- Stage 2 论文草稿、引用列表、方法说明、图表说明、数据或实验来源。
- Stage 4 修订稿、Response to Reviewers、patch apply report、revision delta、新增引用。
- Material Passport：citation verification summary、experiment provenance、claim intent manifest、terminal policies、audit artifacts、version records。

输出

- Stage 2.5 Integrity Report。
- Stage 4.5 Final Integrity Report。
- Issue ledger 和 correction list。

人工检查

- 是否允许调用外部 resolver、plagiarism checker 或 cross-model。
- integrity fail 后选择修复、重验、暂停或终止。
- advisory 是否可接受进入下一阶段。

### 3. pipeline状态追踪与阶段路由

把用户当前研究任务、已有材料、Material Passport、阶段产物和人工确认，转成可恢复的 pipeline 状态、阶段路由、checkpoint、handoff 包和 reset boundary。

证据：source-reviewed

输入

- 用户任务：从零研究、继续写作、已有草稿审查、收到审稿意见、格式收口、恢复旧会话。
- 已有材料：研究问题、文献 corpus、paper draft、review report、revision roadmap、response letter、Material Passport。
- 状态对象：pipeline stage、stage status、loop count、pending decision、checkpoint type、material inventory、integrity history、review history。

输出

- 起始阶段判断和推荐 mode。
- 当前 stage、next stage、blocked reason、required material。
- Checkpoint dashboard 和人工决策项。

人工检查

- 起始阶段是否判断正确。
- 是否允许从已有材料中途进入。
- MANDATORY checkpoint 的继续、修正、暂停或终止选择。

### 4. 修订回复与复审闭环

把 Editorial Decision Package、Revision Roadmap、修订稿、Response to Reviewers 和可选 Material Passport 追踪字段，转成逐点回复、R&R Traceability Matrix、复审验证报告、残留问题和新的接受或修订判断。

证据：source-reviewed

输入

- Editorial Decision Package 和 Revision Roadmap。
- academic-paper revision mode 产出的 revised manuscript、patch apply report、Response to Reviewers。
- Schema 8 Response to Reviewers：revisionround、items、summary、wordcountdelta、newreferencesadded、summaryofchanges、newcontenthighlight。

输出

- Response to Reviewer Comments。
- Change Log。
- R&R Traceability Matrix。

人工检查

- 作者是否接受逐点回复格式。
- revisionlocation 是否足够精确。
- 未完成项是否允许标为 deliberate limitation。

### 5. 全流程学术pipeline调度

把一个研究者的论文任务定位到合适阶段，并调度 research、write、integrity、review、revise、re-review、finalize 和 process summary，保证每个阶段都有产物、状态、确认点和失败回路。

证据：source-reviewed

输入

- 用户自然语言任务，例如从零写论文、已有论文需审查、收到审稿意见需修订、只需格式转换。
- 已有材料状态：研究问题、文献、草稿、审稿意见、修订稿、最终稿。
- 可选环境变量：ARSPASSPORTRESET、resumefrompassport=<hash、ARSCLAIMAUDIT、ARSCROSSMODEL。

输出

- 当前阶段判定和推荐 mode。
- 被调度的 skill 和阶段产物。
- Checkpoint dashboard、mandatory decision、progress status。

人工检查

- 当前任务是否需要完整 pipeline，还是只用某个子 skill。
- integrity fail 是否修正、重验或暂停。
- 审稿结果是接受、修订、重构还是终止。

### 6. 分视角论文审查与issue生成

把已确认的 reviewer configuration、待审论文、reviewer contract 和审查模式，转成 EIC、methodology、domain、perspective 和 devil advocate 的独立审查报告、issue records、dimension scores、问题严重度和作者问题清单。

证据：source-reviewed

输入

- 已确认的 Field Analysis Report 和 Reviewer Configuration Cards。
- 待审论文完整正文、Paper Draft 对象或 pipeline handoff。
- reviewer sprint contract：reviewerfull 或 reviewermethodologyfocus。

输出

- EIC Review Report。
- Methodology Review Report。
- Domain Review Report。

人工检查

- reviewer configuration 是否已经确认。
- 目标期刊和 reviewer role 是否和论文真实语境一致。
- field-norm evidence 是否足以支撑 critical 或 major。

## 处理机制

1. **hook写入范围与仓库安全治理**：解释 academic-research-skills 如何把 Claude Code hook、单阶段 subagent 名称、phase 写入范围、受保护基础设施、仓库安全政策、非商业定位和贡献规则组合成运行时与仓库层安全边界。这个机制不负责生成学术产物，但会决定 agent 能写哪里、哪些能力属于安全报告范围、哪些贡献或使用方式会被拒绝。
2. **material-passport与共享合同总图**：解释 academic-research-skills 如何把跨阶段 handoff、Material Passport、passport 子对象、reset boundary、version records、rejection log、terminal policies 和 schema lint 组合成一套共享状态合同。这个机制本身不直接生成论文、综述或审稿报告，但它决定各阶段产物能否被后续 agent 接收、能否断点重启、哪些证据可以继续流转、哪些失败会阻断流程。
3. **命令入口与模式路由**：解释 academic-research-skills 如何把用户的 slash command、自然语言意图、Claude Code plugin 安装面和 mode registry 连接到四个顶层 skill。这个机制本身不产出研究、论文或审稿结果，但会决定用户入口进入哪张能力卡、用什么 mode、是否继承会话模型、是否触发状态写入命令，以及哪些安装路径会改变运行边界。

## 开始方式

1. 形成研究问题：选择研究模式，生成研究问题简述和方法蓝图，确认后再进入下一阶段
2. 检索与分析文献：从语料库或外部来源检索文献，筛选核验后产出跨文献综合报告
3. 撰写论文：配置论文类型，生成大纲，构建论证链，撰写正文并处理引用合规
4. 多视角审稿：执行分视角审稿（方法、领域、前瞻、数据分析），产出问题清单和编辑综合决策
5. 修订与验证：应用修订补丁，验证作者回复，通过引用对齐和完整性核验门
6. 格式化输出：生成中英文摘要和格式化投稿包（LaTeX/DOCX/PDF），完成合规检查

## 环境要求

- 安装并配置好该skill suite的Claude Code运行环境
- 明确的研究问题或想探索的主题领域
- 目标期刊、引用格式和输出格式偏好（LaTeX/DOCX/PDF）
- 可选：已有文献语料库以支持语料优先检索

## 关键限制

- 跳过提纲确认环节直接生成全文，导致后续需要大面积返工
- 依赖工具自动检索的文献而不手动核实关键引用来源的真实性
- 把LLM审稿评分等同于真实同行评审，忽略工具无法替代的学术判断
- 对依赖领域规范的论文，未提供外部参考标准就直接使用工具

## 已核对范围

- 已核对来源定位、公开入口和核心处理链

## 仍待核对

- 尚未完成真实环境运行和大规模输入验证

## 许可边界

本页的能力分析、结构与说明由 bo老师整理，采用 CC BY 4.0。上游项目、代码、名称和商标继续遵循各自许可。这里不重新发布上游代码。
