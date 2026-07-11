# Papermill

把 Jupyter Notebook 变成可参数化、可重复执行的计算单元。

通过参数注入、执行引擎和多存储后端，把交互式 Notebook 接入可复现的数据与实验流水线。

- 官网详情：https://bolaoshi.cn/research-tools/papermill
- 原始来源：https://github.com/nteract/papermill
- 读取版本：e4e4ddd362037309c53ab5230541759707779687
- 核对日期：2026-07-10
- 上游许可：BSD-3-Clause
- 验证程度：catalogued
- 编辑判断：recommended

## 适合这些任务

- 需要批量重复运行 Notebook 的研究者
- 构建参数扫描和报告流水线的团队

## 这些情况要谨慎

- Notebook 本身不可复现的项目
- 缺少内核和数据依赖管理的环境

## 能力拆解

### 1. notebook参数化执行

读取一个输入 notebook，应用参数（参数注入），用指定 Jupyter kernel 逐 cell 执行，在执行过程中实时落盘带运行元数据的输出 notebook，并在出错时保留 traceback 和错误标记后抛出结构化异常。这是 papermill 的核心能力，把 notebook 变成可编程、可追踪、可批量运行的计算单元。

证据：source-reviewed

输入

- inputpath：notebook 路径（本地/云/URL/NotebookNode/- stdin）。支持路径参数化（{param} 插值）。
- outputpath：输出路径（None 则不落盘；- 则写 stdout）。
- parameters（dict，可选）：传入参数，交由 papermill-能力-参数注入与cell标记 注入。

输出

- 执行后 notebook（含 cell outputs、executioncount、papermill 运行元数据）。
- stdout/stderr 文件（若指定）。
- 错误标记 cell（失败时）。

人工检查

- autosave 指数退避的真实触发条件需运行验证（长 cell + 慢盘）。
- cell 失败后 break 的行为意味着后续 cell 不执行，需人工确认部分结果是否可信。
- DeadKernel 退出码 138 的下游处理（调度系统）需人工确认。

### 2. 参数注入与cell标记

把一组外部传入的参数（dict/YAML/base64）注入到一个 Jupyter Notebook 中，覆盖该 notebook 用 parameters cell tag 声明的默认值，生成一个新的带 injected-parameters cell 的 notebook，使其在执行时使用传入值而非默认值。这是 papermill 把手工 notebook 变成可参数化计算单元的核心机制。

证据：source-reviewed

输入

- notebook（NotebookNode）：必须先用 nbformat 读取并 upgrade 到 v4，cell 需有 tags 属性。期望含一个 tag 为 parameters 的 code cell，声明默认参数（如 alpha = 0.6 学习率）。
- parameters（dict 或 str）：调用方传入的参数字典。若为 str，视为 YAML 文件路径（readyamlfile）。
- reportmode（bool）：注入 cell 是否标记 sourcehidden。

输出

- injected-parameters cell：含目标语言参数赋值代码的 code cell（产物）。
- 修改后的 NotebookNode（cells 插入/替换、metadata.papermill.parameters 写入）。
- 后续交由 papermill-能力-notebook参数化执行 执行。

人工检查

- 非 Python 语言的 translator 输出正确性需人工或测试复核（本轮只静态阅读）。
- 无 parameters cell 时注入到顶部可能改变 notebook 语义（变量定义顺序），需人工确认。
- pm 内置参数注入路径后，跨时区机器的 datetime 可能不一致，需人工确认时间口径。

### 3. 参数自省

在不执行 notebook 的情况下，静态解析它的 parameters cell，推断出该 notebook 接受哪些参数、每个参数的名称、推断类型、默认值和帮助文本，返回结构化 Parameter 列表。用于 CLI 帮助（--help-notebook）和程序化参数发现，让调用方在执行前知道 notebook 的参数契约。

证据：source-reviewed

输入

- notebookpath：notebook 路径（支持路径参数化，可传 parameters 解析路径占位符）。
- parameters（可选，仅用于路径插值）：opennotebook 调 addbuiltinparameters + parameterizepath 解析路径。
- notebook 内的 parameters cell 源码（需有 tag parameters）。

输出

- inspectnotebook 返回 {name: {name, inferredtypename, default, help}}。
- CLI 打印格式化帮助。

人工检查

- PythonTranslator.inspect 对复杂类型注解（list[int]、dict[str, float]）的解析正确性需测试复核。
- 多行 dict/list 定义的 flatten 行为（中间行注释丢弃）需人工确认是否符合预期。
- 非 Python notebook 的自省完全不可用（返回空），调用方需人工确认是否接受。

### 4. 多引擎io读写

让 notebook 的读取和写入能透明地跨本地文件系统、S3、Azure DataLake/Blob、GCS、HTTP/HTTPS、HDFS、Github 和 stdin/stdout，使执行核心不需要关心存储后端差异。通过路径 scheme 前缀路由到对应 handler，并支持第三方包经 entry point 注册新 scheme。

证据：source-reviewed

输入

- path：notebook 路径字符串，scheme 前缀决定 handler。
- buf（写时）：notebook 序列化后的字符串（nbformat.writes）。
- extensions（可选）：read/write 时检查文件扩展名（默认 ['.ipynb','.json']），不匹配警告。

输出

- notebook 文件内容（读时，str/bytes→decode）。
- 落盘的 notebook（写时，到各后端）。
- listdir 返回的路径列表（listnotebookfiles 过滤 .ipynb）。

人工检查

- 各云 handler 的真实读写行为需凭证环境验证（本轮未配置）。
- GCS 重试的实际触发条件（429/None code）需运行确认。
- Github handler 的 path 解析（splits[3]/[4]/[6]）对非常规 URL 的鲁棒性需人工复核。

### 5. 引擎与io扩展注册

让 papermill 的三个核心可扩展面：执行引擎（engine）、I/O handler 和参数 translator：能被第三方包通过 Python entry point 自动发现并注册，或被程序化 register 调用注册，从而在不改 papermill 主代码的前提下增加新的执行后端、存储后端或语言支持。这是 papermill 把"读 notebook → 执行 → 写 notebook"三步都做成可扩展插件的关键治理结构。

证据：source-reviewed

输入

- 第三方包的 entry point 声明：在 pyproject.toml 的 [project.entry-points."papermill.engine"]、[project.entry-points."papermill.io"] 声明 name→class 映射。
- 编程式 register 调用：直接调 papermillengines.register(name, EngineClass)、papermillio.register(scheme, HandlerClass)、papermilltranslators.register(language, TranslatorClass)。
- engine 需继承 papermill.engines.Engine 并实现 executemanagednotebook。

输出

- 注册到单例的新 engine/handler/translator，可被 --engine/路径前缀/kernel 名调用。
- 扩展后的可用执行后端/存储后端/语言集合。

人工检查

- entry point 加载失败（目标包损坏）时 papermill 是否整体崩溃，需运行确认。
- 多个第三方包注册同名 engine/scheme 时的冲突解决（I/O 是 LIFO 后注册优先，engine 是 dict 覆盖），需人工确认。
- translator 不支持 entry point 是设计缺口还是有意，需结合 changelog 复核。

## 处理机制

1. **核心处理链**：把 Jupyter Notebook 变成可参数化、可重复执行的计算单元。

## 开始方式

1. 先整理一个可从头运行的 Notebook
2. 标记 parameters cell
3. 用本地输入输出跑通一次

## 环境要求

- Jupyter kernel
- Python
- 明确的数据和环境依赖

## 关键限制

- 本轮未实际运行 Notebook
- 参数化不能修复隐藏状态
- 云存储需要额外凭证和依赖

## 已核对范围

- 只读上游快照 本地只读快照，commit e4e4ddd362037309c53ab5230541759707779687
- papermill/execute.py（executenotebook、preparenotebookmetadata、removeerrormarkers、raiseforexecutionerrors）
- papermill/engines.py（Engine、NBClientEngine、NotebookExecutionManager、PapermillEngines）
- papermill/clientwrap.py（PapermillNotebookClient、papermillexecutecells）
- papermill/cli.py（CLI 参数与退出码）
- papermill/iorw.py（loadnotebooknode、writeipynb）
- README.md、docs/usage-execute.rst
- README.md（Parameterizing a Notebook 段）

## 仍待核对

- 未运行真实 notebook 执行
- 未验证逐 cell 落盘（requestsaveoncellexecute）的实际写入时机
- 未验证 autosave 指数退避的实际触发
- 未验证 starttimeout/executiontimeout 的超时行为
- 未验证 DeadKernel 退出码 138 的实际触发
- 未运行真实 notebook 验证注入 cell 的实际位置与覆盖行为
- 未验证非 Python 语言（R/Scala/Julia/Matlab/C/F/Powershell/Bash）translator 的 codify 输出正确性
- 未验证 Black 格式化（PythonTranslator.codify 的 black 调用）在无 black 时的降级

## 许可边界

本页的能力分析、结构与说明由 bo老师整理，采用 CC BY 4.0。上游项目、代码、名称和商标继续遵循各自许可。这里不重新发布上游代码。
