# GROBID

把学术 PDF 转成带结构、元数据和参考文献的 TEI XML。

面向学术出版物的机器学习解析基座，覆盖全文结构、头部元数据、参考文献、坐标和模型训练。

- 官网详情：https://bolaoshi.cn/research-tools/grobid
- 原始来源：https://github.com/kermitt2/grobid
- 读取版本：ae58f2dd30e38207d2e4a919663de14f3695b836
- 核对日期：2026-07-10
- 上游许可：Apache-2.0
- 验证程度：catalogued
- 编辑判断：recommended

## 适合这些任务

- 需要批量结构化学术 PDF 的团队
- 构建文献数据管线的开发者

## 这些情况要谨慎

- 只处理少量 PDF 的零配置用户
- 无法维护 Java 服务和模型资源的环境

## 能力拆解

### 1. PDF全文结构化解析

把学术论文 PDF 完整解析成结构化 TEI XML（header + body + references），通过级联序列标注模型识别文档区域、段落、章节标题、图表、公式和参考文献标记，并可选附加 PDF 坐标、句子分段和页面范围控制。

证据：source-reviewed

输入

- input：PDF 文件（multipart 上传或本地路径）
- consolidateHeader / consolidateCitations / consolidateFunders：0/1/2/3 控制元数据归一化程度
- includeRawCitations / includeRawAffiliations / includeRawCopyrights：是否保留原始字符串

输出

- PDF全文结构化解析的结构化结果

人工检查

- flavor 选择是否匹配文档类型需人工判断。
- 坐标正确性依赖 pdfalto 与 PDF CropBox/MediaBox 一致，需人工校对。
- 非主流出版商/领域的解析质量需人工复核（训练数据偏向生命科学）。

### 2. PDF坐标标注与增强

在 PDF 全文解析结果上附加结构坐标（bounding boxes），使提取的结构可定位回原始 PDF 页面，支持三种输出：TEI XML 的 @coords 属性、JSON 标注（含页面尺寸+bounding box 列表，用于浏览器 PDF.js 渲染）、以及实验性的增强 PDF（写入 PDF 注释）。

证据：source-reviewed

输入

- PDF 文件（需先走 fulltext 或 references 解析）
- teiCoordinates：要附加坐标的结构类型列表（ref/biblStruct/persName/figure/formula/head/s/p/note/title/affiliation）
- consolidateCitations：标注端点是否调外部服务补全 DOI/arXiv 链接

输出

- TEI XML：结构带 @coords 属性，<facsimile 声明页面尺寸。
- JSON：{pages: [...], annotations: [{pos: [...], ...}]}。
- 增强 PDF：带 goto/link 注释的 PDF 文件。

人工检查

- 坐标精度（尤其多栏、复杂版面）需人工校对。
- JSON 标注的 bounding box 与浏览器渲染对齐需人工验证。
- 增强PDF 的注释正确性需人工检查。

### 3. PDF头部元数据提取

从 PDF 文档首页区域提取结构化书目元数据（标题、作者、摘要、作者单位、关键词、DOI、日期、版权许可证等），输出 TEI XML 或 BibTeX，并可选通过外部服务归一化补全。

证据：source-reviewed

输入

- input：PDF 文件
- consolidateHeader：0（不归一化）/1（归一化并注入全部额外元数据，默认）/2（归一化但仅注入 DOI）/3（仅用已提取 DOI 归一化）
- includeRawAffiliations：0/1 是否保留原始单位地址字符串

输出

- PDF头部元数据提取的结构化结果

人工检查

- 作者名拆分（forename/middle/surname）对非西方名字格式需人工校对。
- 单位地址拆分对复杂多机构场景需人工复核。
- consolidation 补全的元数据准确性需人工抽检。

### 4. 专利引文提取

从专利出版物中提取和解析专利引文（patent citations）和非专利引文（non-patent literature citations），支持纯文本（TXT）、ST.36 XML 和 PDF 三种输入格式，输出 TEI XML 引文列表，并对 PDF 输入提供带坐标的 JSON 标注。

证据：source-reviewed

输入

- TXT 模式：input（UTF-8 专利文本，application/x-www-form-urlencoded）
- ST36 模式：input（ST.36 标准 XML 文件，multipart/form-data）
- PDF 模式：input（专利 PDF 文件，需有文本层）

输出

- 专利引文提取的结构化结果

人工检查

- 专利号标准化（国家代码、docNumber、kind code）准确性需人工校对。
- 专利 vs 非专利引文分流准确性需人工复核。
- 非主流格式的非专利引文解析质量需人工检查。

### 5. 原文片段解析

把原始文本片段（日期字符串、作者姓名序列、单位地址块、单条/多条引文字符串）独立解析成结构化 TEI XML 片段，不依赖 PDF 上下文，用于集成到其他工作流或单独调用。

证据：source-reviewed

输入

- date：原始日期字符串（如 "September 16th, 2001"）
- names：作者姓名序列（如 "John Doe and Jane Smith"，header 或 citation 上下文）
- affiliations：单位地址块字符串（如 "Stanford University, California, USA"）

输出

- 原文片段解析的结构化结果

人工检查

- 非西方姓名格式（如中文、日文）的 forename/surname 拆分需人工校对。
- 复杂单位地址（多层级机构）的拆分需人工复核。
- 非标准日期格式（如 "Spring 2021"）的 ISO 标准化需人工确认。

### 6. 参考文献外部归一化

用外部书目服务（CrossRef REST API 或 biblio-glutton）校正和补全 grobid 从 PDF 提取的头部元数据和参考文献，注入 DOI/PMID/PMCID 等身份字段和更完整的书目记录，并通过限流、退避和 post-validation 控制匹配质量和可用性。

证据：source-reviewed

输入

- grobid 提取的 BiblioItem（含 raw citation 字符串和已提取字段）
- consolidateMode：0/1/2/3（档位含义见头部元数据卡和参考文献卡）
- 配置项（grobid.yaml）：

输出

- 参考文献外部归一化的结构化结果

人工检查

- consolidation 补全的元数据准确性需人工抽检（尤其近似匹配）。
- DOI 匹配通常可靠，但 bibliographic 查询的弱匹配需人工复核。
- glutton vs CrossRef 的匹配差异需人工评估。

## 处理机制

1. **核心处理链**：把学术 PDF 转成带结构、元数据和参考文献的 TEI XML。

## 开始方式

1. 先使用官方服务或容器跑单篇 PDF
2. 检查 TEI 字段和参考文献
3. 再进入批量和并发配置

## 环境要求

- Java 或 Docker
- 模型和 PDF 解析依赖
- 足够内存

## 关键限制

- 本轮只完成静态源码核对
- 复杂版式和扫描件需要单独评估
- 批量运行需控制并发和外部归一化服务

## 已核对范围

- 静态确认 FullTextParser.processing 作为全文级联入口
- 静态确认级联顺序 segmentation→fulltext→header→reference-segmenter→citation→figure/table
- 静态确认 pdfalto 产出 LayoutToken（含坐标+字体），模型在 LayoutToken 上做序列标注而非纯文本
- 静态确认 REST /api/processFulltextDocument 和 batch processFullText 两条入口
- 静态确认 TEI XML 输出合同和可选坐标/句子分段/页面范围参数
- 静态确认 teiCoordinates 参数支持 ref/biblStruct/persName/figure/formula/head/s/p/note/title/affiliation 等结构附带坐标
- 静态确认 referenceAnnotations 端点返回 JSON 标注（含页面尺寸 + bounding box 列表）
- 静态确认 annotatePDF 端点返回修改后的 PDF（实验性，可能弃用）

## 仍待核对

- pdfalto 原生二进制内部解析细节未深读
- 各模型 Wapiti/DeLFT 内部推理链路未深读
- flavor（article/light-ref/sdo）对具体标注规则的影响未深读
- 大文件/超大 blocks 的降级阈值实际触发行为未验证
- 坐标计算的精度和 CropBox/MediaBox 差异处理未验证
- JSON 标注的完整 schema（哪些字段进入 annotations）未深读
- annotatePDF 的 PDF 修改内部实现未深读
- figure/formula 标注的覆盖率和准确性未验证

## 许可边界

本页的能力分析、结构与说明由 bo老师整理，采用 CC BY 4.0。上游项目、代码、名称和商标继续遵循各自许可。这里不重新发布上游代码。
