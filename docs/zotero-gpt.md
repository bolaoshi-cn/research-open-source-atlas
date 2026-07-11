# Zotero GPT

在Zotero内用GPT对论文全文进行问答、翻译、润色，并把结果写回笔记

适用于在Zotero中阅读论文时需要快速提问、翻译段落或生成摘要的研究者。安装插件后，选中论文条目或打开PDF，输入命令标签如AskPDF，GPT会自动检索最相关段落作为上下文生成回答，并可将结果一键写入关联笔记或给条目添加标签。

- 官网详情：https://bolaoshi.cn/research-tools/zotero-gpt
- 原始来源：https://github.com/MuiseDestiny/zotero-gpt
- 读取版本：c3d13201245a8b9dda0f468cff093e57804bb23c
- 核对日期：2026-07-10
- GitHub Stars：7,251（2026-07-11）
- 上游许可：AGPL-3.0
- 验证程度：catalogued
- 编辑判断：recommended

## 什么时候使用

- 阅读PDF论文时想快速提问（如'这篇论文的创新点是什么'）而不用跳出Zotero界面时
- 需要将论文段落翻译成中文或用学术风格润色英文写作时
- 希望将AI对话结果自动记录到Zotero笔记中，保持阅读与笔记在同一工具内闭环时
- 想一键给论文条目打上由AI根据摘要推荐的标签时

## 先准备什么

- 安装Zotero（7或6版本）和zotero-gpt插件（xpi文件）
- 配置OpenAI兼容API的密钥、接口地址和模型名称（或使用内置免费API回退）
- 如需自动写入功能，安装zotero-better-notes插件
- 根据需要自定义命令标签的prompt模板（如翻译指令、摘要格式、润色风格要求）

## 它会怎样推进

1. **选中论文**：在Zotero主面板选中条目，或打开PDF阅读器定位到目标论文页面
2. **输入问题**：在浮窗中输入问题，或点击AskPDF等命令标签使用预设的prompt模板
3. **检索相关段落**：工具用PDF全文做向量检索，找出与问题最相关的段落作为回答上下文
4. **GPT生成回答**：将问题加检索段落拼接成prompt，通过OpenAI兼容API流式生成Markdown回答
5. **写回笔记**：回答完成后双击复制，或自动写入Better Notes主笔记/条目笔记，完成阅读闭环

## 第一次这样开始

帮我安装这个库：https://github.com/MuiseDestiny/zotero-gpt
安装前先告诉我需要什么环境，安装后帮我跑通一个最小示例。

## 结果出来后先看什么

- 检查GPT回答是否引用了论文中的具体内容而非编造不存在的信息（核对引用段落编号）
- 检查向量检索返回的段落是否与问题真正语义相关（而非仅关键词表面匹配）
- 对于翻译任务，核对专业术语翻译是否准确，公式和特殊符号是否在翻译中丢失
- 检查写入笔记的格式和位置是否符合预期（笔记目标、光标位置、HTML渲染效果）

## 常见误区

- 把GPT基于少数检索段落的回答当作对全文的完整理解（上下文窗口有限，只看相似度最高的top段）
- 使用内置免费API回退时未评估隐私风险（论文全文会发送到第三方免费API，无隐私保障）
- 安装社区标签模板后未经审核直接使用（标签中的eval代码可执行任意JavaScript，无沙箱隔离）
- 忽略AGPL-3.0许可限制（将插件代码集成到闭源工具或商业化服务中会触发传染条款）

## 能力拆解

### 1. OpenAI流式对话

把组装好的 prompt 文本通过 OpenAI 兼容 API（或免费第三方 API 回退）以 SSE 流式方式发送，实时增量渲染回复到浮窗，维护会话消息历史，处理错误显示。这是所有问答/翻译/润色标签的统一回答后端。

证据：source-reviewed

输入

- requestText：组装好的 prompt 文本（命令标签 execTag / 普通输入 execText 传入）
- secretKey（pref，默认 ""）：OpenAI API key，未配置则回退免费 API
- api（pref，默认 https://api.openai.com）：API base，规范化去尾部 /v1

输出

- 流式 Markdown 回复（实时渲染到 output-container，markdown-it + mathjax3）。
- 会话消息记录（views.messages，user/assistant 角色对）。
- 错误信息（HTTP error code/type/message，ProgressWindow + 浮窗显示）。

人工检查

- secretKey 等配置由用户在 /secretKey /api /model 命令或设置面板填写，安全性自负。
- 免费 API 回退的数据隐私需用户自行评估。

### 2. PDF相关段落检索问答

从当前 PDF 全文或选中条目集合中，按用户问题用向量相似度检索最相关段落，以 [number] 编号拼接为上下文，供后续 GPT 问答引用。这是 AskPDF / SearchItems / Items 等标签的上下文采集核心。

证据：source-reviewed

输入

- queryText：用户问题字符串
- PDF 全文（通过 PDF.js 的 PDFViewerApplication.pdfViewer.pages 读取 textContent）
- 或选中条目集合（ZoteroPane.getSelectedItems()）

输出

- 编号拼接的相关段落文本：[1]pageContent\n\n[2]pageContent...，注入命令标签 prompt。
- 辅助定位按钮：点击可跳转 PDF 页面/坐标或选中 Zotero 条目。
- Document 列表（带 box/id metadata）供 insertAuxiliary。

人工检查

- 检索相关段落是否真正回答问题需人工判断（系统只提供候选上下文）。
- PDF 段落聚类边界对学术 PDF 以外版式的适用性需人工抽检。

### 3. Zotero文献写回

把 GPT 回复写回 Zotero：自动写入 Better Notes 主笔记（笔记内空格触发跟随模式）、双击复制并插入笔记、或让 GPT 生成 item.addTag 代码执行给条目添加标签。这是 AI 阅读辅助的闭环写回环节。

证据：source-reviewed

输入

- GPT 回复文本/HTML（output-container 的 markdown-body innerHTML 或 pureText）
- Better Notes 主笔记编辑器实例（Zotero.BetterNotes.api.editor + data.workspace.mainId）
- 或当前打开的条目笔记 editor（Zotero.Notes.editorInstances 按 parentID 匹配）

输出

- Better Notes 主笔记/条目笔记的 HTML 内容（光标处插入）。
- 剪贴板纯文本（双击复制）。
- Zotero 条目标签（addTag + saveTx）。

人工检查

- 写回笔记的 HTML 位置/内容需人工确认（自动写回主笔记直接 insert）。
- GPT 生成的 addTag 标签需人工审核（是否准确反映文献主题）。
- 双击插入的笔记目标（主笔记 vs 条目笔记）由 tab 状态决定，需人工确认。

### 4. 命令标签prompt模板执行

把可复用的 prompt 模板封装为"命令标签"，标签文本内可嵌入 ${JS代码} 或 js 代码 片段，执行时先 eval 这些代码（获取 PDF 选中文本、相关段落、条目字段等动态上下文）替换进模板，再交给 GPT 回答，实现"一键"加速研究问答。

证据：source-reviewed

输入

- 标签文本（含 名字[attr] 头部 + prompt 正文 + 嵌入 ${code}/ js ）
- 用户输入（Meet.Global.input，浮窗输入框值）
- trigger 关键词或正则（匹配输入后自动执行标签）

输出

- 组装好的 prompt 文本（注入 getGPTResponse）。
- 渲染后的 Markdown 回复（浮窗 output-container，markdown-it + mathjax3）。
- 会话消息记录（views.messages，user/assistant 角色对）。

人工检查

- 标签 prompt 模板的质量（是否给出有效引用编号指令、语言指令）需人工设计。
- GPT 回复 eval 执行的代码安全性需人工审核（无沙箱）。

## 处理机制

1. **核心处理链**：在Zotero内用GPT对论文全文进行问答、翻译、润色，并把结果写回笔记

## 开始方式

1. 选中论文：在Zotero主面板选中条目，或打开PDF阅读器定位到目标论文页面
2. 输入问题：在浮窗中输入问题，或点击AskPDF等命令标签使用预设的prompt模板
3. 检索相关段落：工具用PDF全文做向量检索，找出与问题最相关的段落作为回答上下文
4. GPT生成回答：将问题加检索段落拼接成prompt，通过OpenAI兼容API流式生成Markdown回答
5. 写回笔记：回答完成后双击复制，或自动写入Better Notes主笔记/条目笔记，完成阅读闭环

## 环境要求

- 安装Zotero（7或6版本）和zotero-gpt插件（xpi文件）
- 配置OpenAI兼容API的密钥、接口地址和模型名称（或使用内置免费API回退）
- 如需自动写入功能，安装zotero-better-notes插件
- 根据需要自定义命令标签的prompt模板（如翻译指令、摘要格式、润色风格要求）

## 关键限制

- 把GPT基于少数检索段落的回答当作对全文的完整理解（上下文窗口有限，只看相似度最高的top段）
- 使用内置免费API回退时未评估隐私风险（论文全文会发送到第三方免费API，无隐私保障）
- 安装社区标签模板后未经审核直接使用（标签中的eval代码可执行任意JavaScript，无沙箱隔离）
- 忽略AGPL-3.0许可限制（将插件代码集成到闭源工具或商业化服务中会触发传染条款）

## 已核对范围

- 静态确认 getGPTResponseByOpenAI 调 POST ${api}/v1/chat/completions，stream=true，SSE onprogress 增量解析 data: JSON 的 choices[0].delta.content
- 静态确认 会话消息切片最后 chatNumber 条（默认 3），user/assistant 角色对 push 到 messages
- 静态确认 deltaTime（默认 100ms）setInterval 平滑渲染，逐字拼接直到 responseText 等于 textArr
- 静态确认 secretKey 未配置时回退到免费第三方 API（requestArgs: aigpt.one / chatbot.theb.ai）
- 静态确认 api URL 规范化（去尾部 /v1），model/temperature 从 Prefs 读取
- 静态确认 错误分支解析 error.code/type/message 并 ProgressWindow 显示
- 静态确认 pdf2documents 从 PDF.js 读取 textContent，合并同行、去页眉页脚、段落聚类，遇到 references/acknowledgements 停止
- 静态确认 selectedItems2documents 把选中条目 JSON（截断 500 字）作为候选文档

## 仍待核对

- 未运行真实 OpenAI 调用（需 secretKey 授权）
- 免费第三方 API（aigpt.one/chatbot.theb.ai）可用性与稳定性未验证，代码注释标"可补充很多免费API"
- SSE 解析在 token 超限时的降级行为未验证（onprogress catch 静默）
- 未运行真实 PDF 检索（需 OpenAI secretKey 授权）
- embeddingBatchNum 分批的实际限流行为未验证
- PDF 段落聚类的字体阈值在不同 PDF 版式下的效果未验证
- 跨页去页眉页脚的 isIntersectLines 误杀率未验证
- 未在真实 Better Notes 环境验证写回（需安装 zotero-better-notes 插件）

## 许可边界

本页的能力分析、结构与说明由 bo老师整理，采用 CC BY 4.0。上游项目、代码、名称和商标继续遵循各自许可。这里不重新发布上游代码。
