# ARIS (Auto-Claude-Code-Research-in-Sleep)

用Markdown skill编排从选题到论文写作的科研全流程自动化

面向希望用AI agent串联科研全流程的研究者。把选题发现、实验执行、跨模型审稿、论文写作和审计拆成可组合的skill，支持夜间无人值守运行。

- 官网详情：https://bolaoshi.cn/research-tools/auto-claude-code-research-in-sleep
- 原始来源：https://github.com/wanshuiyin/Auto-claude-code-research-in-sleep
- 读取版本：885a237b47d9bd70739efdc8b82fb27eeda8ec3f
- 核对日期：2026-06-16
- GitHub Stars：13,266（2026-07-11）
- 上游许可：MIT
- 验证程度：catalogued
- 编辑判断：worth-watching

## 什么时候使用

- 希望让AI agent自动完成选题→实验→审稿→写作的完整科研循环时
- 需要跨模型审稿（用不同模型家族做独立评审）时
- 需要夜间自动运行长链路科研任务并监控授权状态时

## 先准备什么

- 准备研究方向描述或已有论文改进目标
- 配置Claude Code或Codex CLI环境及API密钥
- 配置至少一个外部reviewer模型（Gemini/OpenAI兼容接口/手动中转）
- 如有GPU实验需准备GPU环境、SSH访问和训练脚本

## 它会怎样推进

1. **选题发现**：从研究方向出发，串联文献检索、创意生成、新颖性检查和研究评审
2. **实验桥接**：将实验计划转化为可运行代码，先跑sanity检查再部署全量实验
3. **自动审稿**：外部reviewer对代码、结果和论文草案做多轮严厉评审和修复
4. **叙事交接**：汇总选题报告、实验结果和审稿记录，生成叙事报告
5. **论文写作**：将叙事报告转为论文结构、图表、LaTeX正文和审计报告

## 第一次这样开始

帮我安装这个库：https://github.com/wanshuiyin/Auto-claude-code-research-in-sleep
安装前先告诉我需要什么环境，安装后帮我跑通一个最小示例。

## 结果出来后先看什么

- 审稿循环是否由不同于执行模型的外部reviewer独立完成
- 阶段状态文件中done和accepted是否被正确分离
- 实验结果的JSON/CSV是否可解析并包含必要指标
- 论文审计产物（proof/claim/citation）是否通过verifier检查

## 常见误区

- 让执行模型自评结果并直接放行（必须由外部reviewer或人工确认）
- 未经人工确认就启动GPU全量实验和论文写作（会消耗大量成本）
- 把审稿循环的almost verdict等同于期刊录用标准

## 能力拆解

### 1. ARIS-Monitor授权状态监控

在本地以只读方式扫描 Claude Code 会话状态，准确识别哪个会话正在等待用户授权，并用置顶小窗提示，用户点击时只把对应终端窗口带到前台。

证据：source-reviewed

输入

- ~/.claude/sessions/<pid.json live session registry。
- 可选 transcript path，由 sessionId 与 cwd slug 推导。
- 当前时间、session pid、cwd、updatedAt、waitingFor 和 status。

输出

- 浮窗 UI。
- scanner.py CLI 输出的分类列表。
- header summary counts。

人工检查

- 点击窗口行触发 focus。
- 是否关闭小窗。

### 2. Research Wiki持久科研记忆

把论文、idea、experiment、claim 和它们之间的关系保存为项目内 research-wiki/，再生成压缩 querypack.md，让后续选题、文献、实验和主张判断能复用旧证据、失败经验和领域缺口。

证据：source-reviewed

输入

- research-wiki/ 项目目录或空项目。
- arXiv ID、title、authors、year、venue、thesis 和 tags。
- idea、experiment、claim 记录和关系边。

输出

- research-wiki/index.md
- research-wiki/log.md
- research-wiki/gapmap.md

人工检查

- 手动 paper metadata 是否可信。
- 关系边 evidence 是否足够。
- 失败 idea 是否属于真实研究失败，而非工具环境失败。

### 3. skill包安装与多宿主分发

把仓库内 skills/<name/SKILL.md 分发到 Claude Code、Codex 等宿主项目中，使用平铺 symlink、manifest、lock 和 doc managed block 管理安装、更新、卸载和 helper 解析。

证据：source-reviewed

输入

- ARIS repo 路径。
- 目标 project path。
- 目标宿主：Claude Code 的 .claude/skills 或 Codex 的 .agents/skills。

输出

- .claude/skills/<skill 或 .agents/skills/<skill。
- .aris/installed-skills.txt 或 .aris/installed-skills-codex.txt。
- .aris/tools symlink。

人工检查

- 是否替换 foreign symlink。
- legacy copy install 如何迁移。
- Codex overlay 选择哪一个。

### 4. 多模型Reviewer MCP桥接

为 ARIS 的跨模型评审提供多个 reviewer bridge，包括 Codex 默认路径之外的 manual review、Claude review、Gemini review 和 OpenAI-compatible LLM chat，使执行宿主可以把评审 prompt 交给不同模型家族或人工中转，并保留线程或异步 job 状态。

证据：source-reviewed

输入

- reviewer prompt、config、model reasoning effort、threadId 或 jobId。
- 环境变量中的 API key、model、base URL、timeout 和 state dir。
- 人工 review 的 browser 或 file mode 输入。

输出

- reviewer response。
- threadId 或 jobId。
- pending review state。

人工检查

- manual review 时由用户选择外部模型。
- 是否接受 file mode 里粘贴的 response。
- 是否把 bridge 安装为长期 MCP server。

### 5. 实验计划到GPU执行桥接

把 EXPERIMENTPLAN.md 中的里程碑、预算、指标和方法描述，转化为可运行实验代码、部署命令、sanity 检查、GPU 运行和初始结果文件，为后续自动审稿循环提供可读输入。

证据：source-reviewed

输入

- refine-logs/EXPERIMENTPLAN.md
- refine-logs/EXPERIMENTTRACKER.md
- refine-logs/FINALPROPOSAL.md

输出

- refine-logs/EXPERIMENTRESULTS.md
- refine-logs/EXPERIMENTTRACKER.md
- 可选 EXPERIMENTLOG.md

人工检查

- 是否允许部署全量实验。
- 是否接受 reviewer 指出的 critical issue 已修复。
- 是否继续运行大规模多 seed sweep。

### 6. 科研全流程阶段调度

把一个研究方向或已有论文改进目标，按选题、实验、自动审稿、总结交接和可选论文写作拆成阶段，并用状态文件和外部评审门控制长链路继续、恢复和停止。

证据：source-reviewed

输入

- 一句话研究方向，或 RESEARCHBRIEF.md。
- 可选 ref paper、base repo、venue、human checkpoint、reviewer difficulty、auto write 等参数。
- 项目目录内的 idea-stage、refine-logs、review-stage 和 paper 产物。

输出

- idea-stage/IDEAREPORT.md
- refine-logs/EXPERIMENTRESULTS.md
- review-stage/AUTOREVIEW.md

人工检查

- 是否允许自动选择 idea。
- 是否允许部署实验和消耗 GPU。
- 是否进入 paper writing。

## 处理机制

1. **Markdown skill与artifact合同总图**：解释 ARIS 怎样把纯 Markdown skill、shared references、helper scripts、MCP reviewer、project artifacts、run state 和安装 manifest 组合成一套可跨宿主迁移的科研自动化骨架。

## 开始方式

1. 选题发现：从研究方向出发，串联文献检索、创意生成、新颖性检查和研究评审
2. 实验桥接：将实验计划转化为可运行代码，先跑sanity检查再部署全量实验
3. 自动审稿：外部reviewer对代码、结果和论文草案做多轮严厉评审和修复
4. 叙事交接：汇总选题报告、实验结果和审稿记录，生成叙事报告
5. 论文写作：将叙事报告转为论文结构、图表、LaTeX正文和审计报告

## 环境要求

- 准备研究方向描述或已有论文改进目标
- 配置Claude Code或Codex CLI环境及API密钥
- 配置至少一个外部reviewer模型（Gemini/OpenAI兼容接口/手动中转）
- 如有GPU实验需准备GPU环境、SSH访问和训练脚本

## 关键限制

- 让执行模型自评结果并直接放行（必须由外部reviewer或人工确认）
- 未经人工确认就启动GPU全量实验和论文写作（会消耗大量成本）
- 把审稿循环的almost verdict等同于期刊录用标准

## 已核对范围

- 读取 aris-monitor/README.md 的 read-only 监控、needs approval 信号和 focus 边界
- 读取 aris-monitor/scanner.py 的 status=="waiting"、session scan、summary 和 transcript tail 逻辑
- 读取 aris-monitor/widget.py 的浮窗渲染、红色提示、折叠和 focus 调用
- 读取 README 中 ARIS-Monitor 与 Claude Fleet 的定位
- 读取 skills/research-wiki/SKILL.md 的 entity、edge、目录结构、helper 解析和 subcommand
- 读取 tools/researchwiki.py 的 init、addedge、rebuildquerypack、ingestpaper、sync 和 stats 函数
- 读取 AGENTGUIDE 中 Research Wiki optional 和 Artifact Contracts
- 读取 README 中 Research Wiki 的 lifecycle memory 说明

## 仍待核对

- 未启动 ARIS-Monitor GUI
- 未读取真实 ~/.claude/sessions
- 未调用 focus-tty.sh 或 osascript
- 未运行 researchwiki.py
- 未调用 arXiv Atom API
- 未验证 querypack 对真实 idea generation 的影响
- 未在临时项目运行安装脚本
- 未验证 Windows PowerShell installer

## 许可边界

本页的能力分析、结构与说明由 bo老师整理，采用 CC BY 4.0。上游项目、代码、名称和商标继续遵循各自许可。这里不重新发布上游代码。
