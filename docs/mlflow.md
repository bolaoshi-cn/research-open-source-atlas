# MLflow

AI工程平台，统一管理ML实验、LLM追踪、模型注册和部署

适合需要追踪大量ML实验或管理LLM调用链路的研究团队。MLflow用统一的对象生命周期治理骨架，把实验参数指标、模型版本、LLM推理轨迹和AI网关路由整合在同一套基础设施中。

- 官网详情：https://bolaoshi.cn/research-tools/mlflow
- 原始来源：https://github.com/mlflow/mlflow
- 读取版本：source-reviewed
- 核对日期：2026-07-10
- GitHub Stars：26,974（2026-07-11）
- 上游许可：Apache-2.0
- 验证程度：catalogued
- 编辑判断：recommended

## 什么时候使用

- 需要记录和对比数百次ML实验的参数、指标和产物
- 需要追踪LLM agent的多步推理轨迹和token消耗
- 需要给训练好的模型做版本管理、阶段标记和部署
- 需要统一管理多个LLM API的路由、限流和成本预算

## 先准备什么

- 安装MLflow并启动Tracking Server（本地或远程）
- 在训练代码中插入几行MLflow logging代码记录参数和指标
- 如需LLM追踪，配置OpenTelemetry或启用MLflow的自动埋点
- 如有多个LLM提供商，在AI Gateway中配置路由和密钥

## 它会怎样推进

1. **记录实验**：在代码中用mlflow.log记录参数、指标和模型产物
2. **对比Run**：在MLflow UI中筛选、排序和可视化对比多次实验
3. **注册模型**：将最佳模型注册到Registry并标记阶段（Staging/Production）
4. **追踪LLM**：自动捕获agent的span树、token用量和推理延迟
5. **部署服务**：将注册模型一键部署为REST endpoint或Docker容器

## 第一次这样开始

帮我安装这个库：https://github.com/mlflow/mlflow
安装前先告诉我需要什么环境，安装后帮我跑通一个最小示例。

## 结果出来后先看什么

- 检查注册模型的版本和阶段标记是否与预期一致
- 验证LLM追踪中的span树是否完整覆盖了agent执行路径
- 确认AI Gateway的预算限额和限流规则是否在真实负载下正确生效

## 常见误区

- 把实验记录当备份用而不同步管理训练数据和环境依赖
- 在LLM追踪中不做采样导致存储和成本远超预期
- 把Staging阶段的模型直接推Production而不做线上评估和灰度验证

## 能力拆解

### 1. AI网关与LLM路由治理

在多个 LLM provider（OpenAI/Anthropic/Gemini/Bedrock/...25 种）和 gateway（Databricks/LiteLLM/Vercel/OpenRouter/...）之上提供一层统一、OpenAI 兼容的路由网关，统一管理 credential、rate limit、fallback、成本预算（budget）、流量分流（A/B）与输入输出护栏（guardrail）。这是从"应用直连各家 API"到"统一可治理的 LLM 访问层"的转换。

证据：source-reviewed

输入

- OpenAI 兼容请求（chat/completions、completions、embeddings）打到 gateway。
- gateway config（route→provider/model 映射、rate limit、fallback、budget policy、guardrail、traffic split）。
- credential（provider API key，集中管理）。

输出

- OpenAI 兼容响应（对客户端透明）。
- 每请求 trace（含成本子 span）→ 与 mlflow tracing 打通。
- budget 超限 webhook（通知外部系统）。

人工检查

- budget 阈值与窗口由人设定。
- guardrail 规则由人设定（误判/漏判需调）。
- fallback 与 traffic split 策略由人定。

### 2. LLM与agent轨迹追踪

把一次 LLM/agent 执行（含多次模型调用、工具调用、检索、子链路）的结构化轨迹（Trace）捕获下来，形成可回放、可评估、可关联到 Run 的树状执行记录。这是 mlflow 在 LLM 时代的核心观测能力，独立于传统 Run 记录。

证据：source-reviewed

输入

- 被 @mlflow.trace 或 startspan 修饰的函数/代码块（含 LLM 调用、工具调用、检索等）。
- 或 autolog 框架自动埋点（openai/anthropic/langchain/crewai 等 60+）。
- 可选：samplingratio、trace destination、OTLP endpoint。

输出

- 持久化 Trace（含 span 树、状态、输入输出）。
- UI 的 trace 视图（树/瀑布图）。
- 可 link 到 Run（linktracestorun），形成"运行 + 轨迹"联合观测。

人工检查

- 采样比例由人设定，影响观测代表性。
- trace 与 run 是否 link 由调用方决定（影响追溯完整性）。
- 生产 trace 保留周期与归档策略需人工治理（archival 模块）。

### 3. 实验跟踪与run状态管理

把一次模型训练/推理执行的"参数、指标、产物、标签、状态"沉淀成可检索、可对比、可复现的持久记录（Run），并按 Experiment 分组。这是 mlflow 最底层也最核心的能力：所有 genai/tracing/registry/evaluation 都建立在 Run 这条时间线之上。

证据：source-reviewed

输入

- 用户代码中对 mlflow.logparam/logmetric/logartifact 或 MlflowClient.log 的调用。
- 可选：trackinguri（file:/sql+sqlite/postgres/mysql/mlflow server URL）、registryuri、experimentid/experimentname。
- 可选：autolog 钩子触发的框架原生对象（OpenAI/OpenAI client、PyTorch model、sklearn estimator 等）。

输出

- 持久化 Run/Experiment/Param/Metric/Tag（结构化，存 tracking store）。
- 二进制 artifact（模型文件、图表、文本）存 artifact store，与 runid 关联。
- MLflow UI 按 experiment/run 维度可视化对比。

人工检查

- experiment 命名规范、tag 体系由团队自定义（无内建约束）。
- 删除 run/experiment 是软删除（lifecyclestage），需人工确认是否 purge。
- 生产 server 需人工配置 auth、DB、artifact 后端与备份。

### 4. 模型打包与flavor体系

把一个训练好的模型（任意框架：sklearn/pytorch/tensorflow/langchain pipeline/custom）打包成一个自描述、可多 flavor 加载、可跨环境复用的标准产物（MLflow Model），并支持签名校验与统一加载。这是模型从"内存对象"变成"可注册、可部署、可评估"产物的转换。

证据：source-reviewed

输入

- 一个训练好的模型对象 + 对应 flavor 模块（mlflow.sklearn 等）。
- 可选：ModelSignature（infersignature 从示例推断）、inputexample、registeredmodelname、artifactpath、conda/pip 依赖、code 路径。

输出

- 自描述 Model 目录（MLmodel + 权重 + 依赖描述）。
- run artifact 中的 model uri（供 registry 注册、deployment 部署、evaluation 评估）。
- PyFunc 可预测对象（load 后）。

人工检查

- 依赖（condaenv/pip）由人或自动推断，决定跨环境可加载性。
- signature 是否强制校验由部署侧决定。
- codepaths 携带哪些源码需人工判断。

### 5. 模型注册与版本生命周期

把分散在 run artifact 中的模型版本，沉淀进一个带名称、版本号、阶段（stage）/别名（alias）、标签、血缘（源 run）的中心注册表，并支撑从开发→预发布→生产→归档的协作生命周期治理。这是模型从"一次性产物"变成"可治理资产"的转换。

证据：source-reviewed

输入

- 来自 run 的 model uri（runs:/<runid/<artifactpath）或已注册模型。
- 注册请求：registeredmodelname、source（model uri）、tags、awaitregistrationfor。
- 流转请求：name、version、stage/alias、archiveexistingversions。

输出

- 版本化的 RegisteredModel/ModelVersion（中心资产目录）。
- 可寻址 model uri（models:/<name/<stage|alias|version）供 deployment 加载、evaluation 引用。
- transition 事件触发 webhooks（CI/CD、通知）。

人工检查

- 是否进 Production、是否归档旧版本需人工裁决（archiveexistingversions 的默认值影响行为）。
- alias 维护（champion/challenger 指向）需人工纪律。
- 审批流（开源弱、托管强）需人工或外部系统补齐。

### 6. 模型评估与eval-run

对一个模型（ML）或一组 trace/prediction（LLM/agent）按一套指标（数值指标、LLM-as-judge、自定义 scorer、基线对比）做系统性评估，产出可检索、可对比、可回溯的评估记录（Evaluation），并把指标写回 run 形成回归基线。这是从"模型/agent 能跑"到"模型/agent 质量可量化、可回归"的转换。

证据：source-reviewed

输入

- 被评估对象：model uri（ML 模型）、dataset（带 label）、predictions、或 trace 集合（LLM）。
- 指标：evaluators（shenzhen/...）、extrametrics、custommetric/scorer、evaluatorconfig、baseline 模型。
- LLM 评估：metric 定义（含 LLM-as-judge 的 prompt、judge model）、数据集（input/expected output）。

输出

- Evaluation 实体 + eval metrics（写回 run）+ eval artifacts（confusion matrix、slice metrics）。
- 回归基线：searchruns 可按 eval metrics 对比跨 run 表现。
- LLM 评估的 per-trace/per-span 评分（与 trace 绑定）。

人工检查

- metric 选择（哪些指标进回归基线）由人决定。
- LLM judge prompt 与 model 由人设定，影响评分有效性。
- 阈值（pass/fail 回归门）由人定。

## 处理机制

1. **store抽象与对象生命周期治理**：解释 tracking/registry/artifact/tracing 四类能力的共同结构：mlflow 把每类能力都拆成"实体 + abstractstore + 多后端实现 + 异步刷新 + search 检索 + 软删除生命周期"的同构骨架。这个横切结构决定了为什么 Run/Trace/RegisteredModel/ModelVersion 的治理方式高度一致，以及为什么同一套概念能同时支撑 ML 与 LLM 两条产品线。

## 开始方式

1. 记录实验：在代码中用mlflow.log记录参数、指标和模型产物
2. 对比Run：在MLflow UI中筛选、排序和可视化对比多次实验
3. 注册模型：将最佳模型注册到Registry并标记阶段（Staging/Production）
4. 追踪LLM：自动捕获agent的span树、token用量和推理延迟
5. 部署服务：将注册模型一键部署为REST endpoint或Docker容器

## 环境要求

- 安装MLflow并启动Tracking Server（本地或远程）
- 在训练代码中插入几行MLflow logging代码记录参数和指标
- 如需LLM追踪，配置OpenTelemetry或启用MLflow的自动埋点
- 如有多个LLM提供商，在AI Gateway中配置路由和密钥

## 关键限制

- 把实验记录当备份用而不同步管理训练数据和环境依赖
- 在LLM追踪中不做采样导致存储和成本远超预期
- 把Staging阶段的模型直接推Production而不做线上评估和灰度验证

## 已核对范围

- 静态确认 gateway FastAPI app 的 OpenAI 兼容路由（/v1/chat/completions、/v1/completions、/v1/embeddings）
- 静态确认 25 个 provider 适配层（gateway/providers）
- 静态确认 budgettracker/budget 模块的成本聚合与 budgetexceeded webhook + trace 阻断
- 静态确认 guardrails（输入/输出护栏）与 rate limit/fallback/traffic-split 配置
- 静态确认 Trace/Span/TraceInfo/LiveSpan 实体与 SpanType(LLM/CHAIN/AGENT/TOOL/CHATMODEL/RETRIEVER/EMBEDDING/EVALUATOR)、TraceState(OK/ERROR/INPROGRESS) 枚举
- 静态确认 fluent trace 装饰器与 startspan 上下文、span 层级 parentid 链、span 权重 sampling 机制
- 静态确认 OpenTelemetry 导出路径（mlflow/tracing/otel）与 processor/destination 分层
- 静态确认 60+ 框架 autolog 式自动埋点（openai/anthropic/langchain/crewai/autogen/dspy 等）

## 仍待核对

- 未真实运行 gateway server
- 未验证 budget 在高并发下的准确性
- 未验证各 provider 的请求/响应兼容性与降级
- 未真实运行 tracing 捕获
- 未验证 sampling 比例在低 ratio 下的代表性偏差
- 未验证 trace 与 run 的 link 行为在并发 trace 下的边界
- 未真实运行 tracking server 与 client 的端到端记录
- 未验证异步日志（synchronous=False）在并发场景下的顺序与丢失边界

## 许可边界

本页的能力分析、结构与说明由 bo老师整理，采用 CC BY 4.0。上游项目、代码、名称和商标继续遵循各自许可。这里不重新发布上游代码。
