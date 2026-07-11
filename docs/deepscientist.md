# DeepScientist

本地优先的科研操作系统，用多Agent自动推进研究全流程

适合希望把整个研究项目从选题到论文纳入一个可恢复系统的研究者。它把研究目标固化为启动契约，用Git分支管理实验探索，通过多Agent交替执行和持久状态让长周期研究不丢失上下文。

- 官网详情：https://bolaoshi.cn/research-tools/deepscientist
- 原始来源：https://github.com/ResearAI/DeepScientist
- 读取版本：3aaab17244f981fc72522180717f2867fdc95b61
- 核对日期：2026-06-16
- GitHub Stars：3,179（2026-07-11）
- 上游许可：Apache-2.0
- 验证程度：catalogued
- 编辑判断：worth-watching

## 什么时候使用

- 研究周期长、需要多轮实验和写作且希望中间状态不丢失
- 想在同一个项目中切换不同Agent（Claude/Codex/Kimi）执行不同阶段
- 需要从持久状态重建研究全景图以决定下一步方向
- 希望把命令执行、记忆、实验证据和图谱分别持久化治理

## 先准备什么

- 填写研究目标、基线要求、执行策略和参考材料
- 选择希望使用的Runner（Claude/Codex/Kimi等）及模型
- 确认本机已安装相应Runner的命令行工具

## 它会怎样推进

1. **启动项目**：填写研究目标、基线和约束，生成启动契约并初始化Quest仓库
2. **阶段推进**：按scout→baseline→idea→experiment→write等阶段切换skill和上下文
3. **Agent执行**：每轮从持久状态重建prompt，由选定Runner执行一步研究操作
4. **工具写回**：Agent通过MCP工具写记忆、记录实验证据、执行命令并保存日志
5. **地图重建**：从Git分支和工件记录重建研究地图，查看各分支的进展和结论

## 第一次这样开始

帮我安装这个库：https://github.com/ResearAI/DeepScientist
安装前先告诉我需要什么环境，安装后帮我跑通一个最小示例。

## 结果出来后先看什么

- 启动契约中的研究目标和约束是否在后续执行中被遵循
- 命令执行日志是否记录了退出码、标准输出和失败原因
- 实验证据记录的数值是否与原始输出文件一致
- 研究地图中各分支的状态是否与实际Git分支和工件一致

## 常见误区

- 把启动契约创建成功视同研究已经完成
- 在Runner连续失败时仍自动重试而不检查失败指纹
- 把UI画布显示的状态当作真相而不是从持久数据重建的视图

## 能力拆解

### 1. BashExec持久命令执行与日志监控

让 agent 在 Quest 或工作区范围内启动可监控的命令 session，把命令、工作目录、进程状态、日志、退出码和压缩读取都保存在 runtime 目录中，支撑长实验和可恢复调试。

证据：source-reviewed

输入

- command。
- Quest root 或 current workspace root。
- workdir。

输出

- session meta。
- stdout 和 stderr log。
- compact read 摘要。

人工检查

- 是否允许安装依赖、下载数据或长时间训练。
- 是否允许访问 GPU 或网络。
- 命令失败后是否继续自动修复。

### 2. BenchStore基准目录发现与安装状态管理

从仓内 AISB/catalog//.yaml 自动发现 benchmark 条目，规范化论文、下载、资源、环境、风险和推荐字段，并把本地安装状态写到 runtime，使 agent 和 UI 能判断哪些 benchmark 可查看、可安装或适合当前设备。

证据：source-reviewed

输入

- AISB/catalog//.yaml。
- 可选 .zh.yaml locale replacement。
- hardware payload。

输出

- benchmark items。
- invalid entries。
- filter options。

人工检查

- 是否允许下载和安装 benchmark。
- 设备资源是否足以运行。
- risk flags 是否允许推荐。

### 3. Connector跨表面对话绑定与消息转发

把 web、TUI 和外部聊天平台绑定到同一个 Quest，对 inbound conversation 进行规范化，并在 agent 回复、进度控制和任务创建后，把可见消息转发到绑定的外部 target。

证据：source-reviewed

输入

- conversation id。
- connector config。
- bound conversations。

输出

- bound conversation。
- outbound connector payload。
- active communication surface prompt block。

人工检查

- 是否绑定外部 connector。
- 哪些消息允许外发。
- connector 失败时是否阻断 Quest 创建或继续本地执行。

### 4. MCP三命名空间Quest状态工具面

为 agent 暴露 Quest 内存、artifact 状态和持久 shell 三类 MCP 工具，让模型在不直接理解全仓实现细节的情况下，读写记忆、记录证据、检查全局状态、准备分支、提交 idea、读取 BenchStore 和执行命令。

证据：source-reviewed

输入

- Quest root。
- MCP namespace。
- agent 调用参数。

输出

- memory card。
- artifact record。
- evidence graph node。

人工检查

- 哪些工具可在当前阶段暴露。
- 哪些 artifact 可对用户或 connector 可见。
- 是否允许 bash 执行高风险命令。

### 5. PromptBuilder阶段Skill注入与上下文组装

在每一轮 runner 执行前，把系统提示、共享交互规则、runtime context、connector contract、当前阶段 skill、Quest 文件、持久状态、优先 memory、最近对话和当前用户消息组装成一份可执行 prompt。

证据：source-reviewed

输入

- Quest snapshot。
- active anchor 和 requested skill。
- runtime config 和 connectors config。

输出

- 单轮 runner prompt。
- quest-local prompt mirror。
- prompt version backup。

人工检查

- 当前阶段 skill 是否正确。
- 外部 connector 可见内容是否合适。
- 是否需要切换 workspace mode 或 active anchor。

### 6. ResearchMap分支工件到Canvas重建

从 Quest 的 Git 分支、idea artifact、main run artifact、research state 和 analysis campaign 状态中重建研究地图，让 UI 或 agent 可以看到当前 workspace、research head、分支编号、最新 idea、实验结果、写作 readiness 和推荐激活节点。

证据：source-reviewed

输入

- research state。
- Git branch canvas payload。
- idea artifacts。

输出

- branch summary。
- runtime refs。
- research canvas payload。

人工检查

- 当前应继续 active workspace 还是 research head。
- analysis 是否完成。
- main result 是否足以进入写作。

## 处理机制

1. **Quest持久状态对象链**：DeepScientist 的多张能力卡共享一条状态对象链：用户启动契约先进入 Quest 仓库，之后 prompt、runner、MCP、bash、memory、artifact、research map 和 connector 都围绕这个 Quest 读写状态。本机制卡说明对象如何传递，以及哪些状态不能被混为一谈。
2. **三层控制面与工具边界**：DeepScientist 把科研运行拆成三层控制面：启动和阶段控制面决定任务边界，runner 和 prompt 控制面决定单轮 agent 行为，MCP 工具控制面决定能写哪些状态和执行哪些命令。本机制卡说明三层边界如何相互约束。
3. **失败治理栈与doctor诊断**：deepscientist 如何在 turn 之间与 runner 之外，治理失败分类、崩溃循环、僵尸状态、环境健康与运行可观测。这张卡补充 deepscientist-能力-多Runner执行调度与失败恢复 只覆盖单 turn 内 retry 的缺口。

## 开始方式

1. 启动项目：填写研究目标、基线和约束，生成启动契约并初始化Quest仓库
2. 阶段推进：按scout→baseline→idea→experiment→write等阶段切换skill和上下文
3. Agent执行：每轮从持久状态重建prompt，由选定Runner执行一步研究操作
4. 工具写回：Agent通过MCP工具写记忆、记录实验证据、执行命令并保存日志
5. 地图重建：从Git分支和工件记录重建研究地图，查看各分支的进展和结论

## 环境要求

- 填写研究目标、基线要求、执行策略和参考材料
- 选择希望使用的Runner（Claude/Codex/Kimi等）及模型
- 确认本机已安装相应Runner的命令行工具

## 关键限制

- 把启动契约创建成功视同研究已经完成
- 在Runner连续失败时仍自动重试而不检查失败指纹
- 把UI画布显示的状态当作真相而不是从持久数据重建的视图

## 已核对范围

- 读取 BashExecService 和 MCP bashexec 工具入口
- 静态确认 quest-local session storage、cwd 限制、meta、monitor process 和 startsession
- 读取 BenchStore YAML reference 和 BenchStoreService
- 静态确认 catalog 扫描、locale replacement、entry normalizer、compatibility、installstate 和 install root
- 读取 connector runtime、channel registry、builtins、daemon connector relay 和 prompt surface context
- 静态确认 conversationid 规范化、channel 注册、bound target 选择和 outbound payload 字段
- 读取 MCP server、memory service、artifact 工具和 docs memory guide
- 静态确认 memory、artifact、bashexec 三个内置命名空间及若干 Quest 状态工具

## 仍待核对

- 未启动真实 shell session
- 未验证长命令、交互命令和进程清理
- 未验证不同平台 shell 行为
- 未下载任何 benchmark
- 未运行 BenchStore UI 或安装流程
- 未验证硬件推荐排序效果
- 未连接任何真实 QQ、Weixin、Telegram、Discord、Slack、Feishu、WhatsApp 或 Lingzhu
- 未验证外部平台鉴权、回调和消息格式

## 许可边界

本页的能力分析、结构与说明由 bo老师整理，采用 CC BY 4.0。上游项目、代码、名称和商标继续遵循各自许可。这里不重新发布上游代码。
