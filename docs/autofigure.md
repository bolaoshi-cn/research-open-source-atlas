# AutoFigure

把论文方法文本或描述转成科研图代码，评审迭代后输出可编辑SVG

面向需要为论文生成或改进科研插图的研究者。从文本描述或论文文件出发，生成SVG或mxGraph XML图代码，经视觉评审和迭代修复后输出可编辑的最终图形。

- 官网详情：https://bolaoshi.cn/research-tools/autofigure
- 原始来源：https://github.com/ResearAI/AutoFigure
- 读取版本：45895304cc270181abf94bb59742648b1d949ed0
- 核对日期：2026-06-16
- GitHub Stars：1,735（2026-07-11）
- 上游许可：MIT
- 验证程度：catalogued
- 编辑判断：worth-watching

## 什么时候使用

- 需要将论文方法描述快速转为可编辑的科研示意图时
- 需要从完整论文文件中自动抽方法段并生成对应图形时
- 需要对已有图形做多轮评审和迭代改进时

## 先准备什么

- 准备论文方法文本、Markdown文件或PDF文件
- 配置LLM provider的API密钥（生成、评审和改进共用或分用）
- 如需图像增强需额外配置图像生成API
- 确定输出格式：SVG（可编辑矢量）或mxGraph XML（draw.io兼容）

## 它会怎样推进

1. **输入文本**：提供方法描述或上传论文文件，系统自动抽取方法段
2. **生成初稿**：LLM根据文本和参考图风格生成初始SVG或mxGraph代码和PNG预览
3. **评审打分**：视觉模型从美观度、内容保真度和占位符使用三维评分
4. **迭代改进**：根据评审问题和可选人工反馈重写图代码，直到质量达标
5. **最终输出**：保存最优版本的代码、PNG和生成报告

## 第一次这样开始

帮我安装这个库：https://github.com/ResearAI/AutoFigure
安装前先告诉我需要什么环境，安装后帮我跑通一个最小示例。

## 结果出来后先看什么

- 生成图形的文字、连线和结构是否与输入描述一致
- 评审分数和具体问题是否指向真实存在的缺陷
- 最终SVG或mxGraph文件是否能在编辑器中正常打开和修改
- 增强后的图形是否仍保留原始科学含义

## 常见误区

- 把模型评审分数等同于期刊审稿人对图形的评价
- 让图像增强模型改变图形的科学含义（文字丢失或连线关系改变）
- 把全篇论文上传到外部模型而不检查数据外发风险

## 能力拆解

### 1. Web会话化人工反馈与增强调度

把文本输入、模型配置、每轮 XML、PNG 预览、evaluation、人类反馈、final XML 和增强变体组织成 Web session，让用户通过前端继续迭代、确认布局和启动增强。

证据：source-reviewed

输入

- 用户输入文本。
- 前端 AutoFigureConfig。
- 用户 API key、base URL、模型、内容类型、迭代参数和增强参数。

输出

- Web session JSON。
- 每轮 XML 和 PNG base64。
- final XML 和 final PNG base64。

人工检查

- 是否允许杀掉占用端口的进程。
- 是否允许把 key 输入浏览器并发送到本地 backend。
- 每轮图是否继续迭代或 finalize。

### 2. 图代码语法校验渲染与产物记录

把 LLM 生成或人工编辑的 SVG、mxGraph XML 转成可渲染预览和可复查文件，并在失败时尝试语法修复或写出错误产物。

证据：source-reviewed

输入

- SVG 代码或 mxGraph XML 代码。
- 输出格式。
- 目标 PNG 路径。

输出

- 每轮代码文件。
- 每轮 PNG。
- 每轮 evaluation JSON。

人工检查

- 代码修复是否改变视觉含义。
- final PNG 是否代表可发布图。
- 是否允许依赖在线 draw.io。

### 3. 图像增强与code2prompt美化

把已生成的布局预览图和可选图代码转成一个或多个更美观的 PNG 变体，其中 code2prompt 模式先让 LLM 分析图代码，再把结构化视觉规格交给图像生成模型。

证据：source-reviewed

输入

- 已生成的 PNG 预览图。
- 可选 SVG 或 mxGraph XML 代码。
- 增强模式：none、code、code2prompt。

输出

- enhanced.png
- enhancedpaths
- Web enhancedimages 数组

人工检查

- 风格描述是否符合论文或展示场景。
- 增强图是否保持原图文字、连线和含义。
- 是否允许上传图像和图代码给外部模型。

### 4. 文本描述到初始科研图代码生成

把用户提供的科研图描述、主题类型、参考图和输出格式配置转成初始 SVG 或 mxGraph XML 图代码，并生成后续评审和迭代可以读取的预览图。

证据：source-reviewed

输入

- 文本描述或 Markdown 内容。
- topic，允许 paper、survey、blog、textbook。
- 输出格式，允许 svg、mxgraphxml，SDK 还把 mxgraph 归一为 mxgraphxml。

输出

- iteration0.svg 或 iteration0.drawio
- iteration0.png
- GenerationResult 中的 svgpath、mxgraphpath、previewpath、finalscore、iterationsused 和 error

人工检查

- 文本描述是否足以支持科研图。
- 输出格式选择是否适合后续编辑。
- 是否允许把论文内容传给外部模型。

### 5. 论文文件到方法段抽取与图生成

把论文 PDF、Markdown、txt 或 Web 端论文内容先压缩成核心方法段，再调用图生成链路生成科研图代码和预览图。

证据：source-reviewed

输入

- paperpath 指向 PDF、Markdown、markdown 或 txt。
- Web 端 inputcontent 和 contenttype=paper。
- 方法抽取 LLM 的 provider、API key、模型和 base URL。

输出

- methodologytext
- extractedmethodology
- 图代码、PNG 预览和后续迭代输入

人工检查

- 抽取方法段是否忠实保留原文边界。
- 是否允许上传全文论文到外部方法抽取模型。
- 被排除的实验、结果或数据集内容是否确实不应进入图生成。

### 6. 评审反馈驱动的代码修复迭代

把当前图代码、PNG 预览、论文内容、参考图、模型评审结果和可选人类反馈转成下一轮改进图代码，并按质量阈值或最大迭代停止。

证据：source-reviewed

输入

- 当前 SVG 或 mxGraph XML。
- 当前 PNG 预览。
- 论文或方法文本。

输出

- 下一轮 SVG 或 mxGraph XML。
- 下一轮 PNG 预览。
- evaluation JSON。

人工检查

- 模型指出的问题是否真实存在。
- 人类分数是否覆盖模型分数。
- 质量阈值是否足以停止。

## 处理机制

1. **provider配置与凭证边界**：解释 AutoFigure 如何在 SDK、backend 和 frontend 三层处理 provider、base URL、模型名、API key 和用户凭证输入。这个机制支撑生成、方法抽取、评审、图像增强和 Web 会话能力。
2. **迭代产物状态链与停止线**：解释 AutoFigure 从输入文本到图代码、预览图、评审记录、迭代历史、最终图和增强变体之间的状态链，以及哪些失败会让链路停止或降级。

## 开始方式

1. 输入文本：提供方法描述或上传论文文件，系统自动抽取方法段
2. 生成初稿：LLM根据文本和参考图风格生成初始SVG或mxGraph代码和PNG预览
3. 评审打分：视觉模型从美观度、内容保真度和占位符使用三维评分
4. 迭代改进：根据评审问题和可选人工反馈重写图代码，直到质量达标
5. 最终输出：保存最优版本的代码、PNG和生成报告

## 环境要求

- 准备论文方法文本、Markdown文件或PDF文件
- 配置LLM provider的API密钥（生成、评审和改进共用或分用）
- 如需图像增强需额外配置图像生成API
- 确定输出格式：SVG（可编辑矢量）或mxGraph XML（draw.io兼容）

## 关键限制

- 把模型评审分数等同于期刊审稿人对图形的评价
- 让图像增强模型改变图形的科学含义（文字丢失或连线关系改变）
- 把全篇论文上传到外部模型而不检查数据外发风险

## 已核对范围

- 静态读取 backend Flask routes、frontend context、types、start script 和 docs
- 静态确认 Web 会话保存 iteration、final XML、enhancement progress 和用户反馈
- 静态读取 SVG、mxGraph XML 校验、渲染、修复和 saveiterationresults
- 静态确认每轮代码、PNG 和 evaluation JSON 会写入输出目录
- 静态读取 README enhancement 示例、Agent enhancement 分支、ImageEnhancer 和 backend enhancement route
- 静态确认可用 none、code、code2prompt 三种输入模式生成增强图变体
- 静态读取 README、SDK agent、generator prompt、provider 调用和代码抽取逻辑
- 静态确认文本输入可生成 SVG 或 mxGraph XML，并可返回预览路径与分数

## 仍待核对

- 未启动 backend 或 frontend
- 未验证本地端口、CORS 和浏览器 draw.io 行为
- 未验证并发 session 的内存占用
- 未运行 CairoSVG
- 未运行 Playwright draw.io export
- 未验证复杂 mxGraph XML 的浏览器导出稳定性
- 未调用图像生成模型
- 未验证增强图是否保持科学含义

## 许可边界

本页的能力分析、结构与说明由 bo老师整理，采用 CC BY 4.0。上游项目、代码、名称和商标继续遵循各自许可。这里不重新发布上游代码。
