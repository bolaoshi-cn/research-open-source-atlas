# Unstructured

把PDF/HTML/Word/图片等非结构化文档转成LLM可消费的结构化元素列表

适用于需要批量预处理大量异构文档（论文PDF、网页、Office文件、扫描图片）再送入RAG或LLM分析的研究者。它会自动识别文件类型，路由到对应的解析器，产出统一类型和元数据的结构化元素，支持多种导出格式和向量数据库对接。

- 官网详情：https://bolaoshi.cn/research-tools/unstructured
- 原始来源：https://github.com/Unstructured-IO/unstructured
- 读取版本：445c95735c4045057f51f399bc04c657751923bd
- 核对日期：2026-07-10
- GitHub Stars：15,106（2026-07-11）
- 上游许可：Apache-2.0
- 验证程度：catalogued
- 编辑判断：recommended

## 什么时候使用

- 需要从PDF论文中提取文字、表格和图片块做后续分析而非仅复制粘贴时
- 构建RAG系统前需要将混排文档（PDF/HTML/Word/邮件/图片）统一预处理时
- 有扫描版PDF或图片需要OCR识别文字时
- 不想为每种文档格式单独编写解析代码，希望一个函数处理所有格式时

## 先准备什么

- 准备好要处理的PDF、HTML、Word、图片、邮件或Markdown文件
- 如需高精度布局检测，安装unstructured-inference及对应模型权重（模型下载较大）
- 如需OCR识别扫描文档，安装系统级tesseract和poppler-utils
- 如需使用托管API模式（免安装重型依赖），注册Unstructured托管服务获取API密钥

## 它会怎样推进

1. **传入文档**：将文件路径、文件对象或URL传给partition函数，无需手动指定文件类型
2. **自动识别类型**：工具用libmagic检测文件真实类型，路由到PDF/HTML/Word等格式专用解析器
3. **解析文档**：PDF走hi_res布局模型/OCR/fast文本三条路径之一，HTML/Word走各自解析器
4. **补全元数据**：每个元素补充坐标、页码、文件类型、链接等元数据，Table元素补text_as_html
5. **后处理与分块**：按标题边界切分语义块，控制块大小和重叠，或导出为JSON/Markdown/CSV
6. **对接下游**：将结构化元素送入RAG系统的embedding步骤，或导出到Weaviate等向量数据库

## 第一次这样开始

帮我安装这个库：https://github.com/Unstructured-IO/unstructured
安装前先告诉我需要什么环境，安装后帮我跑通一个最小示例。

## 结果出来后先看什么

- 检查元素类型分类是否正确（Title/NarrativeText/Table是否被正确识别，尤其多栏PDF）
- 检查表格是否被完整抽取为text_as_html，关键数据是否缺失或行列错位
- 对于扫描版PDF，核验OCR文字准确率，尤其是专业术语、数学符号和公式
- 检查chunking策略是否在语义边界上切割，未被标题误判或页眉页脚残留导致过度切分

## 常见误区

- 把所有文件类型的解析效果等同看待（hi_res模式需额外安装模型，fast模式无表格结构和OCR）
- 依赖auto策略默认结果而不核对降级路径（auto可能因依赖缺失静默降级到fast纯文本抽取）
- 将按标题分块后的输出直接送入LLM而不检查语义断裂（Title误判会破坏完整的论证段落）
- 认为托管API处理结果与本地完全一致（托管侧是黑盒，模型版本和参数可能与本地不同）

## 能力拆解

### 1. HTML分区

把 HTML 文档/网页转成带类型、语言、链接的结构化元素，支持 v1（lxml）与 v2（ontology schema）两套解析器，可跳过 header/footer 并保留图片 alt 文本。

证据：source-reviewed

输入

- filename / file（"r" 模式）/ text / url 四选一
- encoding（默认 utf-8）、headers、sslverify
- skipheadersandfooters：丢弃 <header/<footer 内容

输出

- HTML分区的结构化结果

人工检查

- v2 ontology 解析质量需人工抽样复核。
- header/footer 过滤可能误伤正文导航，需人工确认。

### 2. PDF高精度与OCR解析

把 PDF 转成带类型、坐标、页码的结构化元素列表，按需走布局检测模型（hires）、OCR（ocronly）或纯文本抽取（fast），并可推断表格结构、抽取图片块。

证据：source-reviewed

输入

- filename / file（二选一）
- strategy：auto(默认)/hires/ocronly/fast
- infertablestructure：仅 hires 有效，Table 元素补 textashtml

输出

- PDF高精度与OCR解析的结构化结果

人工检查

- auto 降级选择是否满足精度需人工复核。
- 表格 html 正确性依赖模型，关键文档需人工校对。
- OCR 多语言需人工确认 tesseract 语言包已装。

### 3. 元素序列化与导出staging

把 list[Element] 序列化为可持久化/可回读格式（JSON/NDJSON/Markdown/CSV/text/coco/base64-gzip），并导出为标注/向量平台（HuggingFace/Argilla/Weaviate/Labelbox/LabelStudio/Prodigy/Datasaur/Baseplate）所需形态，形成 partition → 持久化/导出闭环。

证据：source-reviewed

输入

- elements: Iterable[Element]（partition/chunking 输出）
- 序列化选项（输出路径、是否 fixmetadatafieldprecision）
- connector 目标平台参数

输出

- 元素序列化与导出staging的结构化结果

人工检查

- connector 导出后目标平台字段映射正确性需人工核对。
- Formula 转 markdown 的 display/inline 判定需抽样复核。

### 4. 元素类型化与元数据合同

为所有格式分区结果提供统一的元素类型分类表与元数据字段合同，使不同来源（PDF/HTML/Office/邮件/图片）的输出可被下游（chunking、staging、embedding）统一消费。

证据：source-reviewed

输入

- 与元素类型化与元数据合同相关的任务材料和配置

输出

- 元素类型化与元数据合同的结构化结果

人工检查

- 核对结果是否符合当前研究问题和证据边界

### 5. 图片OCR分区

把图片（PNG/JPG/HEIC/TIFF）转成带类型、坐标的结构化元素，默认走 hires 布局检测，可切 ocronly 纯 OCR；不能走 fast。

证据：source-reviewed

输入

- filename / file
- strategy：默认 hires，可 ocronly（fast 非法）
- languages、hiresmodelname

输出

- 图片OCR分区的结构化结果

人工检查

- 核对结果是否符合当前研究问题和证据边界

### 6. 按标题分块

把 partition 产出的元素序列按标题（section heading）与元数据变化切成语义块（CompositeElement），并对超长块按字符/ token 硬切，对 Table 单独成块，供 RAG/嵌入下游使用。

证据：source-reviewed

输入

- elements: Iterable[Element]（partition 输出）
- maxcharacters 或 maxtokens（+tokenizer）：硬上限（二选一）
- combinetextundernchars：合并小段，默认=hardmax

输出

- 按标题分块的结构化结果

人工检查

- Title 误判会导致过度切分，combinetextundernchars 是补救，但可能破坏语义边界，需人工权衡。
- 表格切分是否保留可读性需人工复核。

## 处理机制

1. **核心处理链**：把PDF/HTML/Word/图片等非结构化文档转成LLM可消费的结构化元素列表

## 开始方式

1. 传入文档：将文件路径、文件对象或URL传给partition函数，无需手动指定文件类型
2. 自动识别类型：工具用libmagic检测文件真实类型，路由到PDF/HTML/Word等格式专用解析器
3. 解析文档：PDF走hi_res布局模型/OCR/fast文本三条路径之一，HTML/Word走各自解析器
4. 补全元数据：每个元素补充坐标、页码、文件类型、链接等元数据，Table元素补text_as_html
5. 后处理与分块：按标题边界切分语义块，控制块大小和重叠，或导出为JSON/Markdown/CSV
6. 对接下游：将结构化元素送入RAG系统的embedding步骤，或导出到Weaviate等向量数据库

## 环境要求

- 准备好要处理的PDF、HTML、Word、图片、邮件或Markdown文件
- 如需高精度布局检测，安装unstructured-inference及对应模型权重（模型下载较大）
- 如需OCR识别扫描文档，安装系统级tesseract和poppler-utils
- 如需使用托管API模式（免安装重型依赖），注册Unstructured托管服务获取API密钥

## 关键限制

- 把所有文件类型的解析效果等同看待（hi_res模式需额外安装模型，fast模式无表格结构和OCR）
- 依赖auto策略默认结果而不核对降级路径（auto可能因依赖缺失静默降级到fast纯文本抽取）
- 将按标题分块后的输出直接送入LLM而不检查语义断裂（Title误判会破坏完整的论证段落）
- 认为托管API处理结果与本地完全一致（托管侧是黑盒，模型版本和参数可能与本地不同）

## 已核对范围

- 静态确认 partitionhtml 四种输入（filename/file/text/url）
- 静态确认 v1/v2 两个解析器版本，v2 用 ontology schema
- 静态确认 skipheadersandfooters 与 imagealtmode 行为
- 静态确认 partitionpdf 三策略（auto/hires/ocronly/fast）路由
- 静态确认 determinepdforimagestrategy 的依赖回退逻辑
- 静态确认 hires 走 unstructuredinference 布局模型、ocronly 走 tesseract、fast 走 pdfminer 文本抽取
- 静态确认表格推断输出 textashtml、图片抽取 base64/落盘两条路
- 静态确认 staging.base 提供双向序列化（dict/json/ndjson/base64-gzip/markdown/text/csv/dataframe/coco）

## 仍待核对

- HTML → 元素的具体映射规则（标题/段落/列表/表格）未深读
- v2 ontology schema 解析路径未深读
- URL 抓取的重试与限流未验证
- unstructuredinference 布局模型（detectron2/yolo）权重与推理细节未深读
- OCR agent（tesseract/paddle）切换与多语言准确率未运行验证
- hires 单页到大文件 auto 降级阈值未深读
- 各 connector 的目标平台 schema 映射细节未深读
- base64-gzip 压缩 round-trip 完整性未运行验证

## 许可边界

本页的能力分析、结构与说明由 bo老师整理，采用 CC BY 4.0。上游项目、代码、名称和商标继续遵循各自许可。这里不重新发布上游代码。
