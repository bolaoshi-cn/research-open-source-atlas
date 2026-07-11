# PapersGPT for Zotero

在 Zotero 内直接用 LLM 对论文 PDF 提问并点击答案回到原文位置

适合日常使用 Zotero 管理文献并希望边读边与论文对话的研究者。安装为 Zotero 插件后，当前 PDF 的段落自动转为可检索上下文，选中文本或输入问题即可让在线或本地模型生成回答，点击引用片段可直接定位回原文。

- 官网详情：https://bolaoshi.cn/research-tools/papersgpt-for-zotero
- 原始来源：https://github.com/papersgpt/papersgpt-for-zotero
- 读取版本：531a4eb51051676426ff5023999c0a55779e1bf6
- 核对日期：2026-06-16
- GitHub Stars：2,504（2026-07-11）
- 上游许可：AGPL-3.0
- 验证程度：catalogued
- 编辑判断：use-with-caution

## 什么时候使用

- 在 Zotero 中阅读 PDF 时需要快速向论文提问并获得带原文锚点的回答
- 想在文献库中跨 PDF 检索与某个问题相关的段落
- 用 Zotero 笔记做论文阅读记录，希望将 AI 回答一键写回笔记
- 本地有可用的 LLM 服务，希望在不离开 Zotero 的情况下使用

## 先准备什么

- 已安装的 Zotero 客户端
- OpenAI、Gemini、Claude 或 DeepSeek API 密钥（或本地 LLM 服务地址）
- 需要在 Zotero 中打开的论文 PDF

## 它会怎样推进

1. **安装并配置插件**：将插件安装到 Zotero，配置模型供应商和 API 密钥
2. **打开 PDF 并自动解析**：在 Zotero Reader 中打开论文，插件自动解析 PDF 段落并建立坐标锚点
3. **提问或选中文本**：用内置 prompt 标签或自由文本提问，可选中 PDF 文本作为额外上下文
4. **获取流式回答与锚点**：模型返回 Markdown 流式回答，每条引用可点击跳转回 PDF 原文位置
5. **写回笔记或复制**：将问答结果保存到 Zotero 笔记，或复制到外部文档继续处理

## 第一次这样开始

帮我安装这个库：https://github.com/papersgpt/papersgpt-for-zotero
安装前先告诉我需要什么环境，安装后帮我跑通一个最小示例。
请先说明许可证和安装风险，等我确认后再安装。

## 结果出来后先看什么

- 回答引用的段落是否确实在 PDF 原文中存在——点击锚点确认定位是否准确
- 扫描版 PDF 或双栏论文的段落解析是否完整——缺失区域可能导致检索盲区
- AGPL 许可证是否与本地工具链的使用方式兼容

## 常见误区

- 把模型回答当作论文事实——LLM 回答是对检索片段的生成式解释，不能跳过阅读原文
- 同时使用多个模型供应商但不记录哪个答案来自哪个模型，导致答案溯源混乱
- 将 AI 回答直接写入正式笔记而不标注“AI 生成”——这在学术规范中应显式区分

## 能力拆解

### 1. PDF长文解析与段落锚点构建

把 Zotero PDF Reader 中当前 PDF 的页面文本抽出、合并成段落候选，并给每个段落保留 page 和 box 坐标，使后续问答引用的候选内容能被点击定位回 PDF 原文位置。

证据：source-reviewed

输入

- Zotero PDF Reader 当前 tab。
- PDFViewerApplication 中已加载的页面对象。
- 用户输入的检索问题。

输出

- 带 [1]、[2] 编号的上下文文本。
- Document[] 候选集合。
- 输出区中的 PDF 定位辅助按钮。

人工检查

- 用户需要回到 PDF 锚点复核回答是否真的由该段支持。
- 后续进入科研生命周期比较前，需要用样本 PDF 检查候选覆盖率。

### 2. 回答定位复制与笔记写回

把模型回答渲染成 Markdown/HTML 输出后，支持复制纯文本、点击候选锚点定位回 Zotero item 或 PDF 段落，并在特定上下文下写入 Better Notes 主笔记或 Zotero note。

证据：source-reviewed

输入

- 模型生成的 Markdown 文本。
- Document[] 候选对象及其 id 或 box metadata。
- 当前 Zotero tab、PDF reader、note editor 或 Better Notes workspace 状态。

输出

- 剪贴板纯文本。
- Better Notes 或 Zotero note 中的 HTML 回答。
- PDF 或 item 定位动作。

人工检查

- 写入笔记前应确认回答已回源核验。
- Better Notes 自动写入能力需要用户确认是否启用。
- 复制与写回后的内容需要人工检查格式。

### 3. 提示标签到LLM流式回答

把用户输入、内置 prompt tag、自定义 tag 和 Zotero 上下文动态拼接成模型请求，再用在线或本地模型流式生成 Markdown 回答，并把回答保存在会话历史中。

证据：source-reviewed

输入

- 用户输入框文本或长文本编辑区。
- 内置 prompt tag 或用户保存的 tag。
- prompt 内嵌 JavaScript code block 或 ${...} 表达式。

输出

- Markdown 回答。
- views.messages 历史。
- progress window 中的字符数、回答状态和错误提示。

人工检查

- 自定义 tag 和来自模型的代码必须人工检查后再执行。
- 模型回答必须回到候选段落或 Zotero item 核验。

### 4. 条目与PDF上下文候选检索

把 Zotero 主界面选中的条目或 PDF Reader 当前论文转成候选 Document 集合，再按用户问题检索最相关片段，输出给 prompt 并保留条目或 PDF 定位锚点。

证据：source-reviewed

输入

- Zotero 主界面选中的一个或多个 item。
- Zotero PDF Reader 当前 PDF。
- 用户查询文本。

输出

- prompt 中的 [n] 编号上下文。
- 输出区中的条目或 PDF 定位辅助按钮。
- LocalStorage 缓存或 localhost:9080 本地文档索引。

人工检查

- 用户需要点击定位回源核验候选是否真支持回答。
- 本地复用前需要确认 embedding provider、数据保留路径和隐私边界。

### 5. 模型供应商与本地LLM治理

在 Zotero 插件内维护在线模型供应商、本地 LLM、API key、API URL、模型选择、模型下载进度和本地服务启动关闭状态，使同一问答 UI 能切换不同模型后端。

证据：source-reviewed

输入

- 用户选择的 publisher 和 model。
- 用户输入的 API key、custom API URL 和 custom model name。
- 用户 email 和 token。

输出

- 当前 provider/model/API key/API URL prefs。
- publisher select 和 model select UI。
- Local LLM 下载进度条和 model ready 状态。

人工检查

- 本地服务下载、启动和运行前需要用户确认安全边界。
- API key 存储和发送范围需要用户确认。
- 选择本地模型前需要确认模型下载体积、来源和许可。

## 处理机制

1. **Zotero插件到模型服务状态链**：这个横向机制解释 PapersGPT for Zotero 在 Zotero 插件 UI、Zotero 材料对象、上下文检索、模型供应商、本地 ChatPDFLocal 服务、输出定位和笔记写回之间怎样传递状态。

## 开始方式

1. 安装并配置插件：将插件安装到 Zotero，配置模型供应商和 API 密钥
2. 打开 PDF 并自动解析：在 Zotero Reader 中打开论文，插件自动解析 PDF 段落并建立坐标锚点
3. 提问或选中文本：用内置 prompt 标签或自由文本提问，可选中 PDF 文本作为额外上下文
4. 获取流式回答与锚点：模型返回 Markdown 流式回答，每条引用可点击跳转回 PDF 原文位置
5. 写回笔记或复制：将问答结果保存到 Zotero 笔记，或复制到外部文档继续处理

## 环境要求

- 已安装的 Zotero 客户端
- OpenAI、Gemini、Claude 或 DeepSeek API 密钥（或本地 LLM 服务地址）
- 需要在 Zotero 中打开的论文 PDF

## 关键限制

- 把模型回答当作论文事实——LLM 回答是对检索片段的生成式解释，不能跳过阅读原文
- 同时使用多个模型供应商但不记录哪个答案来自哪个模型，导致答案溯源混乱
- 将 AI 回答直接写入正式笔记而不标注“AI 生成”——这在学术规范中应显式区分

## 已核对范围

- 读取 src/modules/Meet/Zotero.ts 的 pdf2documents、mergeSameLine、paragraph clustering 和 box metadata 逻辑
- 读取 src/modules/views.ts 的 insertAuxiliary PDF box 定位逻辑
- 读取 src/modules/views.ts 的 setText、output dblclick、insertAuxiliary 和 Better Notes 触发
- 读取 src/modules/Meet/BetterNotes.ts 的 getEditorText、insertEditorText、replaceEditorText、follow
- 读取 src/modules/Meet/Zotero.ts 的 getPDFAnnotations 和 getPDFSelection
- 读取 src/modules/base.ts 的 defaultChatPrompt、defaultBuiltInTags 和 parseTag
- 读取 src/modules/views.ts 的 execTag、execText、命令处理和 prompt JS 插值逻辑
- 读取 src/modules/Meet/integratellms.ts 的 OpenAI、Gemini、Claude、本地 LLM 流式回答分支

## 仍待核对

- 未在真实 Zotero Reader 中运行
- 未验证扫描版 PDF、双栏论文、长参考文献区和异常坐标
- 未在真实 Better Notes 中写入
- 未验证普通 Zotero note pane 写入
- 未验证复制和写回的 HTML 格式
- 未调用真实模型
- 未验证 prompt tag 中 JS 执行安全
- 未验证模型输出代码执行分支

## 许可边界

本页的能力分析、结构与说明由 bo老师整理，采用 CC BY 4.0。上游项目、代码、名称和商标继续遵循各自许可。这里不重新发布上游代码。
