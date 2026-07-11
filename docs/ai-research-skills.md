# AI Research SKILLs

面向AI研究Agent的大型技能库，覆盖从构思到论文的科研全流程

适合AI/ML研究者用Agent辅助完成从研究构思、实验执行到论文写作的全流程。通过自主科研调度器管理研究进程，按需调用98个领域技能执行具体任务，经引用核验后产出论文草稿。

- 官网详情：https://bolaoshi.cn/research-tools/ai-research-skills
- 原始来源：https://github.com/Orchestra-Research/AI-Research-SKILLs
- 读取版本：773a52944ba4747a18bd4ae9ade53fff041adcbc
- 核对日期：2026-06-16
- GitHub Stars：10,603（2026-07-11）
- 上游许可：MIT
- 验证程度：catalogued
- 编辑判断：worth-watching

## 什么时候使用

- 想系统性用AI Agent辅助完成AI/ML研究的完整生命周期
- 需要标准化的论文写作、引用核验和审阅响应工作流
- 需要为研究Agent搭建覆盖训练、评估、推理和写作的技能库

## 先准备什么

- 明确的研究问题或已有实验结果的代码仓库
- 目标会议或期刊信息，用于论文写作和格式装配
- Claude Code或兼容的AI Agent运行环境

## 它会怎样推进

1. **启动研究**：输入研究问题，自动初始化工作区、搜索文献并识别研究缺口与初始假设
2. **执行实验**：通过内层循环优先测试假设，调用领域技能执行并记录结果
3. **综合发现**：外层循环聚合所有实验结果，更新研究记忆，决定深入、拓宽或转向
4. **核验引用**：通过API获取可验证的BibTeX条目，无法确认的引用标注为占位符
5. **撰写论文**：生成带贡献叙事的完整初稿，通过会议checklist和引用门质量检查

## 第一次这样开始

帮我安装这个库：https://github.com/Orchestra-Research/AI-Research-SKILLs
安装前先告诉我需要什么环境，安装后帮我跑通一个最小示例。

## 结果出来后先看什么

- 确认研究日志是否完整记录了所有关键决策和实验结果
- 检查所有引用是否已核验，每个占位符是否有明确的补全计划
- 验证论文初稿满足目标会议的格式、页数和checklist要求

## 常见误区

- 未在研究启动前锁定评价指标，导致实验后选指标的确认偏差
- 让Agent凭记忆生成BibTeX引用，跳过引用外部核验环节
- 一次性安装全部98个技能而不按需选择性加载
- 用自治模式直接提交论文草稿，忽略人工对研究贡献表述的最终确认

## 能力拆解

### 1. ARA研究artifact编译与审阅

把论文、代码仓、实验日志、原始笔记或会话过程转成 Agent-Native Research Artifact，并在结构验证后做语义审阅，检查证据、主张、探索路径和方法严谨性。

证据：source-reviewed

输入

- PDF paper、arXiv 链接、GitHub repo、本地代码目录、notebook、实验日志、配置、训练输出、原始笔记、会议记录或对话。
- 可选参数：--output <dir、--rubric <path。
- 既有 ara/ 目录和会话历史。

输出

- 完整 ARA artifact 目录。
- coverage gap patch 记录。
- session provenance 记录。

人工检查

- 输入材料是否允许被完整读取。
- 是否允许写 ARA 目录。
- derived evidence 是否符合用户想要的复查粒度。

### 2. 技能发布同步与库存校验

在 skill 目录或 npm package 发生变化时，用静态库存检查、GitHub Action、zip 打包、API 上传和 npm trusted publishing 控制仓库内容、marketplace 和 npm 包之间的同步状态。

证据：source-reviewed

输入

- 文件系统中的 SKILL.md。
- 顶层 category 目录。
- README、CLAUDE、WELCOME、npm package README 中的库存数字。

输出

- 库存校验日志。
- GitHub Action pass 或 fail。
- marketplace zip payload 和 API response。

人工检查

- 是否允许上传 marketplace。
- 是否允许发布 npm。
- 是否接受 skip reason。

### 3. 研究生命周期技能目录封装

把 AI 研究生命周期中的模型架构、微调、后训练、分布式训练、评价、推理、RAG、论文写作和科研 artifact 等知识，封装成 agent 可发现、可触发、可渐进读取的 skill 目录。

证据：source-reviewed

输入

- 某个 AI 研究工具、框架、工作流或论文写作能力。
- 官方文档、GitHub issue、示例代码、模板、脚本和参考资料。
- 目录分类，例如 06-post-training、11-evaluation、20-ml-paper-writing。

输出

- 98 个 agent 可读 SKILL.md。
- 23 个顶层研究能力类别。
- README、CLAUDE、CONTRIBUTING 和 marketplace 中的目录导航。

人工检查

- 是否把它作为本地 skill 系统的参考来源。
- 是否只吸收少数结构机制，还是对单个 skill 做二次拆解。
- 是否进入第二阶段科研生命周期锚点碰撞。

### 4. 自主科研双循环调度

把一个研究问题或已有研究状态，组织成持续运行的 AI 研究项目，借助内层实验循环、外层综合循环和 domain skill 路由推进到进展报告或论文写作。

证据：source-reviewed

输入

- 用户给出的模糊研究方向、明确研究问题或已有计划。
- 已存在的 research-state.yaml、research-log.md 和 findings.md。
- 可调用的 domain skill 目录，例如 data processing、training、evaluation、interpretability、paper writing。

输出

- 结构化研究工作区。
- 实验协议、实验结果、raw data 和分析记录。
- findings.md 中的综合叙事。

人工检查

- 是否允许持续运行和多轮自动实验。
- 是否允许调用外部文献 API、GPU、云资源或模型服务。
- 是否允许 agent 自行 commit protocol 和实验结果。

### 5. 论文写作与引用核验

从一个已有研究仓库、实验结果或论文写作任务出发，组织贡献叙事、会议格式、引用核验和草稿迭代，产出面向顶级 ML/AI 会议的论文草稿。

证据：source-reviewed

输入

- 研究仓库、README、results、outputs、experiments、configs、draft 文档。
- 目标会议，例如 NeurIPS、ICML、ICLR、ACL、AAAI、COLM。
- 已有 citation、.bib 文件、DOI、arXiv、Semantic Scholar 或 CrossRef 线索。

输出

- 论文大纲、Figure 1 计划、完整 first draft。
- 已核验 BibTeX。
- placeholder citation 列表。

人工检查

- 主贡献 framing。
- 目标会议。
- 关键结果选择。

### 6. 跨agent技能安装与更新

根据用户选择的范围和本机已安装 agent，把上游 skill 从 GitHub 浅克隆下载到统一目录，再为 Claude Code、Codex、Gemini CLI、Cursor、OpenCode、OpenClaw 等 agent 建立全局 symlink 或项目内 copy。

证据：source-reviewed

输入

- 用户选择：everything、quickstart、categories、individual、list、update、uninstall。
- 可选参数：--all、--category、--skill、--local。
- 本机 agent 配置目录，例如 ~/.claude、~/.codex、~/.gemini、~/.cursor、~/.config/opencode。

输出

- 全局 canonical skill 目录。
- 每个 agent 的 skills symlink。
- 项目级 agent skills copy。

人工检查

- 是否允许写用户 home 下 agent 配置目录。
- 是否允许安装全部 98 个 skill。
- 是否允许建立 symlink 或删除已有路径。

## 处理机制

1. **安装状态与跨agent路径治理**：解释上游 npm installer 怎样在一个 canonical 目录、多个 agent skills 目录、项目本地 copy 和 lock file 之间维持安装状态，并指出它和本机 skill 单一真实源规则的迁移边界。
2. **库存漂移与发布门**：解释上游怎样用库存脚本、GitHub Action、zip 文件数保护和 npm version gate 控制技能库演进中的文档漂移、marketplace 同步和 npm 发布。
3. **技能目录与渐进披露结构**：解释该仓库怎样把 98 个 AI 研究 skill 组织成 agent 可发现、可路由、可按需深读的知识结构，并把总控层、领域执行层、论文写作层和 ARA 层放到同一目录体系中。

## 开始方式

1. 启动研究：输入研究问题，自动初始化工作区、搜索文献并识别研究缺口与初始假设
2. 执行实验：通过内层循环优先测试假设，调用领域技能执行并记录结果
3. 综合发现：外层循环聚合所有实验结果，更新研究记忆，决定深入、拓宽或转向
4. 核验引用：通过API获取可验证的BibTeX条目，无法确认的引用标注为占位符
5. 撰写论文：生成带贡献叙事的完整初稿，通过会议checklist和引用门质量检查

## 环境要求

- 明确的研究问题或已有实验结果的代码仓库
- 目标会议或期刊信息，用于论文写作和格式装配
- Claude Code或兼容的AI Agent运行环境

## 关键限制

- 未在研究启动前锁定评价指标，导致实验后选指标的确认偏差
- 让Agent凭记忆生成BibTeX引用，跳过引用外部核验环节
- 一次性安装全部98个技能而不按需选择性加载
- 用自治模式直接提交论文草稿，忽略人工对研究贡献表述的最终确认

## 已核对范围

- 已静态核验 ARA compiler、research-manager、rigor-reviewer 三个 SKILL.md 的输入、文件结构、验证和审阅机制。
- 已静态核验 scripts/check-inventory.sh、check-inventory.yml、sync-skills.yml 和 publish-npm.yml。
- 已静态核验 98 个 SKILL.md、23 个顶层类别、frontmatter 基本字段、README/CLAUDE/CONTRIBUTING 的目录与质量口径。
- 已静态核验 0-autoresearch-skill/SKILL.md、references/skill-routing.md、references/agent-continuity.md 和模板目录中的机制描述。
- 已静态核验 20-ml-paper-writing/ml-paper-writing/SKILL.md 的 repo 到 paper、citation gate、venue workflow 和 human feedback 边界。
- 已静态核验 npm package、CLI 入口、agent detection、global install、local install、update、uninstall 和 lock file 逻辑。

## 仍待核对

- 未编译真实 ARA，未运行 Level 1 校验，未生成 level2report.json。
- 未触发 GitHub Actions、未上传 Orchestra marketplace、未发布 npm。
- 未逐一评估 98 个 skill 的内容正确性、外部依赖有效性、官方文档新旧程度或真实 agent 激活效果。
- 未真实运行 /loop、OpenClaw heartbeat、实验任务、progress presentation 生成或论文收口。
- 未调用 Semantic Scholar、arXiv、CrossRef、Exa MCP，未生成 LaTeX 论文，未验证模板编译。
- 未运行 npx 安装，未写入 ~/.orchestra，未创建任何 agent skill symlink，未验证 Windows copy fallback。

## 许可边界

本页的能力分析、结构与说明由 bo老师整理，采用 CC BY 4.0。上游项目、代码、名称和商标继续遵循各自许可。这里不重新发布上游代码。
