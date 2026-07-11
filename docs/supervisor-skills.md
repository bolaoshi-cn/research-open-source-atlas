# Supervisor-Skills

把导师的经验方法拆成7个AI可调用的技能，覆盖从idea评估到投稿审查全流程。

面向需要结构化科研指导的博士生和青年研究者。从idea质量评估开始，到论文逻辑骨架规划、写作、科研图审计，最后做投稿前的五维审查。

- 官网详情：https://bolaoshi.cn/research-tools/supervisor-skills
- 原始来源：https://github.com/HKUSTDial/Supervisor-Skills
- 读取版本：e36828dde1e59e3537afc4d62bb572ae815845d1
- 核对日期：2026-06-16
- GitHub Stars：3,840（2026-07-11）
- 上游许可：CC-BY-NC-SA-4.0
- 验证程度：catalogued
- 编辑判断：worth-watching

## 什么时候使用

- 有初步科研idea但不确定是否值得投入，需要系统性评估
- 写完论文后想在投稿前做一轮导师视角的全面审查
- 正文写完但逻辑骨架松散，需要重建技术论文的论证主线
- 论文中的图表设计不清楚，需要审查信息传达和视觉质量

## 先准备什么

- 初步idea的描述、可用资源、时间线和领域背景
- 论文完整稿或至少引言、方法和实验章节
- 目标会议或期刊的投稿标准和Research Contribution评估习惯
- 论文中使用或计划绘制的图表草稿

## 它会怎样推进

1. **Idea评估**：输入idea描述，输出五维评分和致命缺陷，判断是否值得立项
2. **逻辑骨架规划**：从问题、局限、目标、挑战出发，做四项自洽检查，搭建论证骨架
3. **章节生成**：生成六段式Introduction大纲或Benchmark论文五支柱骨架
4. **图表审计**：从信息传达、视觉一致性和数据诚实三个维度审查图表设计
5. **投稿前审查**：从贡献、清晰度、实验、完整性、方法五个维度做严重性分级审查

## 第一次这样开始

帮我安装这个库：https://github.com/HKUSTDial/Supervisor-Skills
安装前先告诉我需要什么环境，安装后帮我跑通一个最小示例。

## 结果出来后先看什么

- idea评估的五维分数和致命缺陷是否明确标注
- 论文骨架的自洽检查是否通过——问题-挑战-方案是否形成闭环
- 投稿前审查的问题清单是否标明了严重性分级和修复建议
- 图表审计是否指出了信息冗余和视觉误导问题

## 常见误区

- 把idea评估的分数当客观标准——评估框架是参考工具，不能替代领域专家判断
- 投稿前审查只做笼统肯定而不指出具体问题和位置——审查必须带严重性分级
- 用Introduction模板生成的内容不核验引用真实性——模板是结构不是文献检索
- 混淆技术缺陷（可修复）和范围限制（研究边界）——两者在结论中的写法完全不同

## 能力拆解

### 1. AI辅助科研会话编排

把用户当前科研任务阶段转换成 AI 协作计划、工具选择、行为守则和完整性检查点，让机械任务交给 AI，学术判断保留给用户。

证据：source-reviewed

输入

- 当前任务阶段：coding、figure、writing 或 mixed。
- 用户可用工具：Cursor、Claude Code、Codex、Figma、Gemini、Overleaf、Grammarly 等。
- 任务时间块、交付目标、人工检查点和合规要求。

输出

- 当前 phase 和 secondary phases。
- 行为规则确认。
- time block 工作计划。

人工检查

- 用户确认任务阶段和工具可用性。
- 用户确认 AI 只做机械加速。
- 用户确认引用、数据、结论和 disclosure。

### 2. Benchmark论文五支柱编排

把 Benchmark 或 Evaluation 论文构想转换成 Research Gap、Construction Pipeline、Evaluation Framework、Empirical Findings 和可选 Companion Method 的五支柱审计与论文结构。

证据：source-reviewed

输入

- 研究领域、benchmark 名称、evaluation blind spot。
- 数据构造方法、来源、规模、质量控制和 split 策略。
- evaluation framework、difficulty tiers、error taxonomy、metrics。

输出

- five-pillar completeness table。
- benchmark introduction six-part logic chain。
- Section 2 到 7 outline。

人工检查

- 研究者确认 benchmark gap。
- 研究者确认数据来源和授权。
- 研究者确认标注与质量控制。

### 3. Introduction六段式大纲生成

把研究动机、局限、目标、挑战、方案和贡献转换成六段式 Introduction 大纲，并检查 running example、challenge module mapping 和 contribution section 是否闭合。

证据：source-reviewed

输入

- research area、limitations、hard constraints、key idea、challenges、solution overview。
- running example 或候选 running examples。
- paper type：Technique Paper 或 New Problem/Setting Paper。

输出

- paper type positioning。
- 六段式 outline。
- paragraph gaps 和 severity。

人工检查

- 用户选择 running example。
- 用户确认 prior work limitations。
- 用户确认 challenge module mapping。

### 4. 技术论文逻辑骨架规划

把技术论文的背景、局限、关键想法、挑战、方法模块和贡献转换成一张 thinking template，并用四项自洽检查找出逻辑断点。

证据：source-reviewed

输入

- 论文类型候选：Technique Paper 或 New Problem/Setting Paper。
- 研究背景、已有方法局限、关键想法或目标。
- 最多 3 个技术挑战。

输出

- paper-type positioning。
- thinking template。
- self-consistency report。

人工检查

- 用户确认论文类型。
- 用户确认 limitations 是否真实对应 prior work。
- 用户确认 method modules 是否可实现。

### 5. 投稿前论文五维审查

把完整论文或关键章节转换成宏观逻辑、写作细节、英语语法、LaTeX 格式、图表质量五个维度的严重性分级审查报告。

证据：source-reviewed

输入

- 完整 paper 或关键 sections。
- 可选 LaTeX 源码、figure/table、section names、target venue。
- 投稿截止时间和用户希望关注的范围。

输出

- CRITICAL、MAJOR、MINOR 计数。
- 五维 finding 表。
- banned vocabulary scan。

人工检查

- 用户确认被审材料完整。
- 用户确认引用和实验结果真实。
- 用户确认修复建议是否采纳。

### 6. 科研idea评估与立项闸门

把一个初步科研 idea 转换成带致命缺陷审计、五维打分、生命周期匹配、范式跃迁探测和最终立项 verdict 的审稿式判断。

证据：source-reviewed

输入

- 用户给出的研究 idea、问题背景、预期贡献和目标 venue。
- 用户可用资源，包括每周有效时间、数据访问、硬件、工程栈和理论或应用偏好。
- 需要对比的候选 idea 或范围边界。

输出

- 立项 verdict。
- fatal flaw 表。
- lifecycle 和 capability fit。

人工检查

- 用户确认 idea 描述是否准确。
- 用户确认资源和时间窗口。
- 用户确认 prior work 列表。

## 处理机制

1. **handbook到skill渐进披露结构**：这个横向机制解释 Supervisor-Skills 怎样把同一套科研方法论拆成面向读者的 handbook、面向上手者的 skills README、面向 agent 的 SKILL.md、面向细节的 references 和面向维护者的 lint 规则。

## 开始方式

1. Idea评估：输入idea描述，输出五维评分和致命缺陷，判断是否值得立项
2. 逻辑骨架规划：从问题、局限、目标、挑战出发，做四项自洽检查，搭建论证骨架
3. 章节生成：生成六段式Introduction大纲或Benchmark论文五支柱骨架
4. 图表审计：从信息传达、视觉一致性和数据诚实三个维度审查图表设计
5. 投稿前审查：从贡献、清晰度、实验、完整性、方法五个维度做严重性分级审查

## 环境要求

- 初步idea的描述、可用资源、时间线和领域背景
- 论文完整稿或至少引言、方法和实验章节
- 目标会议或期刊的投稿标准和Research Contribution评估习惯
- 论文中使用或计划绘制的图表草稿

## 关键限制

- 把idea评估的分数当客观标准——评估框架是参考工具，不能替代领域专家判断
- 投稿前审查只做笼统肯定而不指出具体问题和位置——审查必须带严重性分级
- 用Introduction模板生成的内容不核验引用真实性——模板是结构不是文献检索
- 混淆技术缺陷（可修复）和范围限制（研究边界）——两者在结论中的写法完全不同

## 已核对范围

- 只读上游仓库 e36828dde1e59e3537afc4d62bb572ae815845d1
- plugins/phd-research/skills/vibe-research-workflow/SKILL.md
- plugins/phd-research/skills/vibe-research-workflow/references/behavior-guidelines.md
- plugins/phd-research/skills/vibe-research-workflow/references/tool-selection.md
- plugins/phd-research/skills/vibe-research-workflow/references/vibe-coding.md
- plugins/phd-research/skills/vibe-research-workflow/references/vibe-figure.md
- plugins/phd-research/skills/vibe-research-workflow/references/vibe-writing.md
- plugins/phd-research/skills/benchmark-paper-template/SKILL.md

## 仍待核对

- 未用真实科研项目跑多日 AI 协作计划
- 未核验具体 venue AI disclosure 政策
- 未验证工具推荐在本机账户和网络环境下可用
- 未用真实 benchmark 项目跑五支柱审计
- 未验证数据构造质量控制输出
- 未验证伴随方法实验设计
- 未用真实 paper skeleton 生成 introduction outline
- 未验证 interactive mode

## 许可边界

本页的能力分析、结构与说明由 bo老师整理，采用 CC BY 4.0。上游项目、代码、名称和商标继续遵循各自许可。这里不重新发布上游代码。
