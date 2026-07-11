# Research-Paper-Writing-Skills

按章节拆分论文写作任务，用结构化模板和claim-evidence对齐检查提升稿件质量。

面向ML/CV/NLP方向的研究生和工程师。从摘要和引言入手，按任务-挑战-贡献主线组织每个章节，最后以审稿人视角做全稿自审。

- 官网详情：https://bolaoshi.cn/research-tools/research-paper-writing-skills
- 原始来源：https://github.com/Master-cai/Research-Paper-Writing-Skills
- 读取版本：9ee5eddc10068cc52590b3a68a827d3a387f5af9
- 核对日期：2026-06-16
- GitHub Stars：4,998（2026-07-11）
- 上游许可：MIT
- 验证程度：catalogued
- 编辑判断：worth-watching

## 什么时候使用

- 初稿写完每一章后需要结构化的写作自查
- Introduction写完后逻辑不清楚，需要按任务-挑战-方案重建主线
- 投稿前想模拟审稿人视角做全稿质量扫描
- 实验章节的表格和图注表述不清，需要可读性审查

## 先准备什么

- 论文各章节草稿（至少Abstract和Introduction）
- 你的技术贡献、核心洞见和方法pipeline的清晰描述
- 对比实验、消融实验和泛化测试的结果
- 目标投稿会议或期刊的格式和评审标准

## 它会怎样推进

1. **路由章节**：明确当前要处理的章节，Skill只加载该章节的写作指南，不混入无关规则
2. **回答前置问题**：写出该章节最核心的几个问题答案（任务是什么、贡献在哪里、为什么有效）
3. **选择模板**：根据贡献数量、任务熟悉度等选择匹配的写作模板和段落结构
4. **生成草稿**：按模板槽位填充内容，生成带段落角色的初稿和mini-outline
5. **质量检查**：检查段落首句是否说明作用、术语是否统一、major claim是否有实验支撑
6. **全稿自审**：以审稿人视角逐维打分，产出claim-evidence对齐表和修订清单

## 第一次这样开始

帮我安装这个库：https://github.com/Master-cai/Research-Paper-Writing-Skills
安装前先告诉我需要什么环境，安装后帮我跑通一个最小示例。

## 结果出来后先看什么

- 每个major claim是否都能对应到具体实验或引用支撑
- 段落首句是否清晰表达了该段的唯一作用
- Introduction的故事线是否从任务-挑战-洞见-贡献逐层推进
- 是否存在只有弱基线对比而做出强结论的实验论证

## 常见误区

- 不加区分地一次加载所有章节指南——应该渐进加载当前任务需要的参考
- 把模板当写作替代品——模板是结构框架，不能替代真实贡献和实验事实
- Introduction先写naive baseline再写改进——会让贡献显得直白，应反向组织
- 表格缺少指标方向标注（越高越好/越低越好）和数字精度控制，读者无法判断

## 能力拆解

### 1. 实验论证与表格质量审查

把 Experiments 写作和审查拆成三类证据问题：是否优于强基线、哪些模块带来增益、在更难设置下能否泛化，并同时检查表格和图形是否以清晰、低噪声、可读的方式呈现实验结论。

证据：source-reviewed

输入

- 论文主要 claim。
- 对比实验、消融实验、泛化或压力测试结果。
- baselines、metrics、datasets 和 evaluation protocol。

输出

- 实验段落或实验章节草稿。
- claim-evidence 对齐表。
- 消融缺口和 hard setting 缺口。

人工检查

- 作者确认 baselines 是否公平且足够强。
- 作者确认 metrics、datasets 和 splits 是否准确。
- 作者确认实验 claim 是否需要弱化或补实验。

### 2. 引言任务挑战流水线组织

把 Introduction 写作拆成任务和应用、已有方法失败、根本技术问题、本文方案、方案有效原因、贡献和实验结果几个层级，并根据任务熟悉度、挑战来源和贡献形态选择合适的引言模板。

证据：source-reviewed

输入

- 论文任务和应用背景。
- 目标指标或读者关心的评价维度。
- 已有方法类别和各自技术限制。

输出

- Introduction outline。
- 结构化引言段落草稿。
- 段落首句和术语稳定性检查。

人工检查

- 作者确认技术挑战是否正好是本文解决的问题。
- 作者确认已有方法描述是否准确。
- 作者确认所有强 claim 都能被实验或引用支撑。

### 3. 摘要挑战贡献模板改写

把论文摘要写作任务拆成任务、技术挑战、洞见或贡献、技术优势和实验结果几个位置，并在三种模板之间选择合适结构来生成或改写摘要。

证据：source-reviewed

输入

- 论文任务或问题定义。
- 当前摘要草稿或待写摘要目标。
- 技术挑战、技术贡献、洞见、优势和实验结果。

输出

- 摘要 outline。
- 改写后的摘要段落。
- 摘要 claim-evidence map。

人工检查

- 作者确认贡献是否真实。
- 作者确认实验结果是否足以支撑摘要 claim。
- 作者确认摘要是否符合目标会议或期刊限制。

### 4. 方法模块三要素写作

把 Method 写作任务转化为模块清单、pipeline 图草稿、每个模块的 motivation、design 和 technical advantage，并按 overview 到子模块的顺序生成可读方法章节。

证据：source-reviewed

输入

- 论文方法 pipeline 或模块清单。
- 每个模块的运行流程、存在原因和有效原因。
- 可用的 pipeline figure sketch 或草图说明。

输出

- Method section outline。
- Overview 和模块 subsection 草稿。
- 模块三要素表。

人工检查

- 作者确认模块边界和运行顺序。
- 作者确认技术优势是否真实并有实验支撑。
- 作者确认关键参数、结构和实现细节是否准确。

### 5. 段落流畅度反向大纲检查

当用户问段落是否 flow 或清楚时，先以外部读者视角检查单段消息、首句、术语可读性和句间关系，再用 reverse outlining 检查段落主题句、证据点和整节 thesis 的映射。

证据：source-reviewed

输入

- 用户提供的论文段落或章节草稿。
- 章节 thesis 或主 claim。
- 段落 topic sentence、证据或解释点。

输出

- 段落 flow 诊断。
- reverse outline。
- 改写建议或改写段落。

人工检查

- 作者确认 thesis 和 topic sentence 是否真实表达论文意图。
- 作者确认改写是否保持原意。
- 作者确认证据是否足以支撑段落消息。

### 6. 相关工作与结论章节定位

把 Related Work 写成按技术主题组织的差异定位，把 Conclusion 写成问题、核心技术、最强证据、影响或 insight、限制和未来方向的收口段落。

证据：source-reviewed

输入

- 直接竞争论文和近期 baseline。
- 技术主题分组。
- 本文与每组工作的机制、假设和失败模式差异。

输出

- Related Work 主题 outline 和段落草稿。
- Conclusion 草稿。
- baseline coverage 和 limitation type 检查清单。

人工检查

- 作者确认 strongest baselines 是否全部覆盖。
- 作者确认本文区别是否真实。
- 作者确认限制是否诚实且不会过度削弱贡献。

## 处理机制

1. **skill入口与参考材料合同**：这个横向机制解释该仓如何用一个 SKILL.md 入口、一个 metadata 文件和一组按章节拆分的 references/examples，把论文写作请求转成可审查产物。

## 开始方式

1. 路由章节：明确当前要处理的章节，Skill只加载该章节的写作指南，不混入无关规则
2. 回答前置问题：写出该章节最核心的几个问题答案（任务是什么、贡献在哪里、为什么有效）
3. 选择模板：根据贡献数量、任务熟悉度等选择匹配的写作模板和段落结构
4. 生成草稿：按模板槽位填充内容，生成带段落角色的初稿和mini-outline
5. 质量检查：检查段落首句是否说明作用、术语是否统一、major claim是否有实验支撑
6. 全稿自审：以审稿人视角逐维打分，产出claim-evidence对齐表和修订清单

## 环境要求

- 论文各章节草稿（至少Abstract和Introduction）
- 你的技术贡献、核心洞见和方法pipeline的清晰描述
- 对比实验、消融实验和泛化测试的结果
- 目标投稿会议或期刊的格式和评审标准

## 关键限制

- 不加区分地一次加载所有章节指南——应该渐进加载当前任务需要的参考
- 把模板当写作替代品——模板是结构框架，不能替代真实贡献和实验事实
- Introduction先写naive baseline再写改进——会让贡献显得直白，应反向组织
- 表格缺少指标方向标注（越高越好/越低越好）和数字精度控制，读者无法判断

## 已核对范围

- 读取 references/experiments.md
- 读取 SKILL.md 中视觉质量和 claim-evidence 规则
- 读取 references/introduction.md
- 读取 introduction examples index 和 introduction 子目录示例文件
- 读取 references/abstract.md
- 读取 abstract example index 和三个 abstract template 文件
- 读取 references/method.md
- 读取 method examples index 和 method 子目录示例文件

## 仍待核对

- 未读取任何真实实验表格样本
- 未验证 LaTeX 表格能否编译
- 未做统计显著性或数据真实性检查
- 未用真实 Introduction 样本运行
- 未验证模板对非 ML/CV/NLP 论文的适配性
- 未验证 claim-evidence 对齐结果
- 未用真实论文摘要样本运行
- 未验证输出是否能自动保持实验支持边界

## 许可边界

本页的能力分析、结构与说明由 bo老师整理，采用 CC BY 4.0。上游项目、代码、名称和商标继续遵循各自许可。这里不重新发布上游代码。
