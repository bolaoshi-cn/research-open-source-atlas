# arxiv-mcp-server

让AI助手通过MCP协议搜索、下载和阅读arXiv论文

面向需要在AI助手中程序化访问arXiv文献的研究者。先搜索论文候选列表，再下载全文为本地Markdown缓存，最后分页阅读，附带语义检索和引用图谱功能。

- 官网详情：https://bolaoshi.cn/research-tools/arxiv-mcp-server
- 原始来源：https://github.com/blazickjp/arxiv-mcp-server
- 读取版本：d58901760d7ede4adb162eaba1725209a933f100
- 核对日期：2026-07-10
- GitHub Stars：2,950（2026-07-11）
- 上游许可：Apache-2.0
- 验证程度：catalogued
- 编辑判断：worth-watching

## 什么时候使用

- 需要让AI助手直接搜索和获取arXiv论文全文时
- 需要在本地构建已读论文的语义检索索引时
- 需要按类别、日期范围和排序过滤arXiv论文时

## 先准备什么

- 安装Python 3.11+环境和arxiv-mcp-server包
- 如需PDF转换需额外安装pymupdf4llm
- 如需语义检索需安装sentence-transformers

## 它会怎样推进

1. **搜索论文**：用自然语言或arXiv查询语法搜索，按类别、日期、排序过滤
2. **获取摘要**：按arXiv ID单独获取论文元数据，评估相关性后再决定是否下载
3. **下载全文**：论文HTML转Markdown缓存到本地，PDF作为回退方案
4. **分页阅读**：大论文按字符偏移分页返回，避免上下文溢出
5. **语义检索**：对已下载论文做向量相似度检索，或以一篇论文找相似论文

## 第一次这样开始

帮我安装这个库：https://github.com/blazickjp/arxiv-mcp-server
安装前先告诉我需要什么环境，安装后帮我跑通一个最小示例。

## 结果出来后先看什么

- 搜索结果的数量和排序是否符合预期
- 下载的Markdown全文是否完整保留了方法、公式和表格
- 语义检索返回的相似论文是否确实与查询相关
- 缓存命中的论文是否是最新版本

## 常见误区

- 把arXiv论文内容直接当作可信指令输入给AI（内容可能含恶意prompt）
- 连续大量请求触发arXiv API限速导致暂时封禁
- 用语义检索结果替代人工文献筛选和相关性判断

## 能力拆解

### 1. arxiv搜索

用自然语言或 arXiv 高级查询语法搜索 arXiv 论文，支持类别过滤、日期范围过滤和排序，返回结构化论文候选列表（含 id/title/authors/abstract/categories/published/url/resourceuri）。该能力还包含按 arXiv ID 单独获取论文元数据（getabstract），用于在下载全文前评估相关性。这是 arxiv-mcp-server 工作流的第一步（searchpapers → downloadpaper → readpaper）。

证据：source-reviewed

输入

- query（str，必填）：arXiv 查询语法。支持引号短语（"machine learning"）、字段限定（ti:/au:/abs:/cat:）、布尔（AND/OR/ANDNOT）。
- maxresults（int，可选，默认 10，上限 50）：最大返回数，受 settings.MAXRESULTS（默认 50）封顶。
- datefrom（str，可选，YYYY-MM-DD）：起始日期。

输出

- 搜索结果：{totalresults, papers: [papercandidate...]}。
- getabstract 结果：单篇元数据 dict（含 pdfurl）。
- 注意：本来源是单源候选（仅 arXiv），无跨源去重。候选对象 supportstatus: unchecked（候选不等于证据）。

人工检查

- arXiv 3s 限速在多并发 MCP 调用下是否足够（README 称 arXiv 要求 = 3s）。
- 429/503 后 60s 等待建议的准确性（README 与代码一致，但未验证实际封禁时长）。
- raw HTTP 路径与 arxiv 包路径在无日期过滤时结果是否一致（两条路径返回字段相同，但排序/分页可能有细微差异）。

### 2. 本地论文语义检索

对已下载到本地的论文集合做语义相似度检索，支持自由文本查询或以一篇已知论文找相似论文，返回按余弦相似度排序的候选列表。该能力是实验性 [pro] 功能，仅在 downloadpaper 已建立本地缓存后生效（空集合返回空结果）。底层用 sentence-transformers all-MiniLM-L6-v2 嵌入摘要，SQLite 存储向量，归一化向量矩阵点积计算余弦相似度。还包含重建索引（reindex）能力。

证据：source-reviewed

输入

- query（str，可选）：自由文本语义查询（如 "attention mechanisms for long sequences"）。
- paperid（str，可选）：找与此论文相似的其他论文。
- maxresults（int，可选，默认 10）：最大返回数，受 settings.MAXRESULTS 封顶。

输出

- semanticsearch 返回：{mode(semanticquery/similartopaper), query, totalresults, papers: [{id, title, abstract, authors, categories, published, score, resourceuri}]}。
- reindex 返回：{status, indexed, failed[], totallocalpapers}。
- 注意：这是本地单源检索（仅已下载论文），score 是相似度排序信号，不等于相关性判断。supportstatus: unchecked。

人工检查

- all-MiniLM-L6-v2 的实际嵌入维度（常见为 384，未验证）。
- 语义检索质量（基于摘要嵌入，不含全文，可能遗漏正文相关论文）。
- rebuildindex 对大量论文（100 篇）的耗时与模型加载开销。

### 3. 论文下载与全文获取

按 arXiv paperid 下载论文，把论文内容转为 markdown 全文，本地缓存为 .md 文件，并支持分页读取大论文。这是 arxiv-mcp-server 工作流的中间环节（searchpapers → downloadpaper → readpaper）。该能力还包含列出本地已下载论文（listpapers）和读取已下载论文全文（readpaper），三者共同构成论文获取与缓存生命周期。HTML 优先、PDF 回退，PDF 走 pymupdf4llm 转 markdown。所有返回内容带 [UNTRUSTED EXTERNAL CONTENT] 安全警告前缀。

证据：source-reviewed

输入

- paperid（str，必填）：arXiv ID（如 2103.12345）。
- start（int，可选，=0）：分页起始字符偏移。
- maxchars（int，可选，=1）：返回最大字符数。

输出

- downloadpaper 返回：{status, source(cache/html/pdf), paperid, content, contentlength, start, returnedchars, nextstart, istruncated}。
- readpaper 返回：同结构（source 缺省）。
- listpapers 返回：{totalpapers, papers: [id...]}。

人工检查

- HTML 端点解析的文本质量（ArticleTextExtractor 是简化版，可能丢失表格/公式格式）。
- PDF 回退路径的实际转换质量（pymupdf4llm，未装 [pdf] extra 未验证）。
- 缓存命中时后台索引刷新的实际行为（best-effort + RuntimeError 捕获）。

## 处理机制

1. **核心处理链**：让AI助手通过MCP协议搜索、下载和阅读arXiv论文

## 开始方式

1. 搜索论文：用自然语言或arXiv查询语法搜索，按类别、日期、排序过滤
2. 获取摘要：按arXiv ID单独获取论文元数据，评估相关性后再决定是否下载
3. 下载全文：论文HTML转Markdown缓存到本地，PDF作为回退方案
4. 分页阅读：大论文按字符偏移分页返回，避免上下文溢出
5. 语义检索：对已下载论文做向量相似度检索，或以一篇论文找相似论文

## 环境要求

- 安装Python 3.11+环境和arxiv-mcp-server包
- 如需PDF转换需额外安装pymupdf4llm
- 如需语义检索需安装sentence-transformers

## 关键限制

- 把arXiv论文内容直接当作可信指令输入给AI（内容可能含恶意prompt）
- 连续大量请求触发arXiv API限速导致暂时封禁
- 用语义检索结果替代人工文献筛选和相关性判断

## 已核对范围

- 只读上游快照 本地只读快照，commit d58901760d7ede4adb162eaba1725209a933f100
- src/arxivmcpserver/tools/search.py（handlesearch、rawarxivsearch、parsearxivatomresponse、ratelimitedget、validatecategories、optimizequery、searchtool schema）
- src/arxivmcpserver/tools/getabstract.py（handlegetabstract、abstracttool schema）
- src/arxivmcpserver/config.py（Settings、getarxivclient）
- README.md（Paper Search 段、Available Tools）
- src/arxivmcpserver/tools/semanticsearch.py（handlesemanticsearch、semanticsearchtool/reindextool schema、connect、getmodel、embedtext、upsertindexrecord、loadvectors、rankbysimilarity、getindexedpapervector、indexpaperbyid、indexpaperfromresult、rebuildindex、IndexedPaper dataclass）
- src/arxivmcpserver/tools/download.py（runindexbyid/runindexfromresult 后台索引触发，getindexsemaphore 串行化）
- src/arxivmcpserver/tools/listpapers.py（isvalidarxivid 用于 rebuildindex 筛选）

## 仍待核对

- 未调用真实 arXiv API
- 未验证 rawarxivsearch 的日期范围 URL 编码在实际查询中的正确性
- 未验证 429/503 限速的实际触发与 60s 等待建议的准确性
- 未验证 arxiv Python 包路径与 raw HTTP 路径在无日期过滤时的结果差异
- 未安装 [pro] extra（sentence-transformers/numpy）
- 未运行语义检索或索引
- 未验证嵌入模型 all-MiniLM-L6-v2 的实际维度与检索质量
- 未验证 SQLite 向量存储的查询性能

## 许可边界

本页的能力分析、结构与说明由 bo老师整理，采用 CC BY 4.0。上游项目、代码、名称和商标继续遵循各自许可。这里不重新发布上游代码。
