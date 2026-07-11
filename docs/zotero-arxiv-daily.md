# Zotero arXiv Daily

根据你的Zotero文献库兴趣画像，每日自动推荐arXiv/bioRxiv/medRxiv新论文到邮箱

适用于每天需要追踪最新预印本但又不想手动刷arXiv的研究者。它以你Zotero文献库作为个人兴趣画像，自动抓取昨日新论文，用embedding相似度加时间衰减加权排序，再让LLM生成一句话总结，最终渲染成邮件推送给你。

- 官网详情：https://bolaoshi.cn/research-tools/zotero-arxiv-daily
- 原始来源：https://github.com/TideDra/zotero-arxiv-daily
- 读取版本：05b20ec5c14ef82f8634c21a0876acd40e02c2b2
- 核对日期：2026-07-10
- GitHub Stars：5,675（2026-07-11）
- 上游许可：AGPL-3.0
- 验证程度：catalogued
- 编辑判断：recommended

## 什么时候使用

- 每天需要追踪arXiv/bioRxiv/medRxiv最新论文但不想手动逐类目浏览时
- 希望论文推荐结果反映个人研究兴趣变化（Zotero库越新的文献权重越高）时
- 需要一个零成本的每日论文自动推送服务（部署在GitHub Actions免费额度上）时

## 先准备什么

- 准备一个Zotero账号并在文献库中积累有摘要的论文条目（作为兴趣画像基础）
- 在Zotero中按研究领域建立集合目录，用于过滤兴趣范围
- 在GitHub仓库中配置Actions secrets（Zotero API密钥、SMTP邮箱授权码、OpenAI API密钥）
- 确定要追踪的arXiv分类代码（如cs.AI、q-bio.GN）和bioRxiv/medRxiv分类

## 它会怎样推进

1. **拉取兴趣画像**：从Zotero拉取有摘要的论文作为个人兴趣语料库，按集合路径glob过滤
2. **抓取昨日新论文**：从arXiv RSS/bioRxiv/medRxiv API抓取昨日新论文，构造候选列表
3. **相似度加权排序**：用embedding计算候选与兴趣语料的cosine相似度，按时间对数衰减加权求和排序
4. **生成一句话总结**：LLM为每篇推荐论文生成TLDR和作者机构，失败时降级为abstract原文
5. **渲染并推送邮件**：渲染HTML邮件，通过SMTP发送给订阅者，GitHub Actions cron每日自动触发

## 第一次这样开始

帮我安装这个库：https://github.com/TideDra/zotero-arxiv-daily
安装前先告诉我需要什么环境，安装后帮我跑通一个最小示例。

## 结果出来后先看什么

- 检查每日推荐的论文标题和摘要是否与研究兴趣匹配（抽查前10篇）
- 检查TLDR总结是否准确概括论文核心贡献而非仅复述标题
- 观察连续多日结果，判断排序是否合理反映兴趣变化，新加入Zotero的论文是否拉高了相关推荐
- 检查Zotero集合路径过滤是否准确，corpus是否覆盖了所有关注领域且未被无关条目污染

## 常见误区

- 把它当作完整文献综述工具（它只看摘要embedding相似度，不评估论文质量、引用数或撤稿状态）
- 忘记配置Zotero集合路径过滤或glob模式写错（corpus为空时工具直接停止整轮，不发送任何邮件）
- 用邮箱登录密码而非SMTP授权码配置GitHub Secrets（会导致login失败）
- 将AGPL-3.0许可的工具代码直接集成到闭源或SaaS项目中（强copyleft传染条款要求衍生代码开源）

## 能力拆解

### 1. embedding相似度排序

把 Zotero 文献库作为用户兴趣画像，对新抓取的候选论文按 embedding 相似度排序。排序分 = 候选摘要与每篇 Zotero 论文摘要的 cosine 相似度的加权和，权重按 Zotero 论文加入时间做对数时间衰减（越新权重越高）。排序后截断到 maxpapernum。

证据：source-reviewed

输入

- corpus: list[CorpusPaper]：用户 Zotero 文献库，每篇带 title、abstract、addeddate、paths。
- candidates: list[Paper]：新抓取的候选论文，每篇带 abstract。
- config.executor.reranker：选择 local 或 api。

输出

- list[Paper]，按 score 降序，截断到 maxpapernum，每个 Paper.score 已写回。
- 输出交给 TLDR / 机构生成和邮件渲染。

人工检查

- 用户需确认 includepath / ignorepath glob 模式正确圈定兴趣集合。
- maxpapernum 过大会超出 GitHub Actions 配额，README 明确提示限制。
- 排序结果是否符合兴趣，需要人工抽查；README 自述算法简单。

### 2. 多源论文抓取与候选构造

把用户配置的 arXiv / bioRxiv / medRxiv 类别和昨日时间窗口，转换成统一的 Paper 候选对象列表。每个候选带标题、作者、摘要、URL、PDF URL 和可选全文。arXiv 还会尝试下载 LaTeX 源码 tar、HTML 版或 PDF，提取全文供后续机构抽取使用。

证据：source-reviewed

输入

- config.source.arxiv.category：arXiv 分类列表，例如 ["cs.AI","cs.CV"]。
- config.source.arxiv.includecrosslist：是否包含跨分类论文，默认 false。
- config.source.biorxiv.category：bioRxiv 分类。

输出

- list[Paper]，每个带 source、title、authors、abstract、url、pdfurl、fulltext。
- 输出交给 reranker.rerank(allpapers, corpus)。

人工检查

- 用户需确认 category 列表覆盖自己的研究领域。
- debug 模式只取前 10 篇，不能代表真实每日规模。
- 若用于研究证据池，需补 DOI、版本、撤稿和来源质量检查。

### 3. 邮件渲染与每日推送

把排好序的候选论文列表（含 LLM 生成的 TLDR 和机构）渲染成 HTML 邮件，通过 SMTP 发送给订阅者。配合 GitHub Actions cron 每日自动触发。无论文时根据 sendempty 决定是否发送空邮件。

证据：source-reviewed

输入

- list[Paper]：排好序、截断后的候选，每个带 title、authors、tldr、affiliations、score、pdfurl。
- config.email.sender / receiver / smtpserver / smtpport / senderpassword。
- config.llm.api.key / baseurl、config.llm.generationkwargs.model、config.llm.language。

输出

- 订阅者邮箱收到的 HTML 邮件。
- 无文件落盘，无持久化产物。

人工检查

- 用户需确认 SMTP 授权码、sender/receiver、smtpserver/port 正确。
- 用户需确认 cron 时间符合时区预期（UTC 22:00）。
- LLM API key 和 baseurl 需有效。

## 处理机制

1. **embedding相似度时间衰减排序骨架**：这个横向机制解释 zotero-arxiv-daily 抓取、排序和交付三张能力卡之间的共同骨架：整个仓的核心是"用个人 Zotero 文献库的摘要 embedding 作为兴趣画像，对新论文摘要做 cosine 相似度，再按文献库加入时间做对数衰减加权求和，得到推荐分"。这个 embedding 相似度 + 时间衰减加权链路决定了候选如何变成排序结果，也决定了下游邮件内容和 LLM 成本。

## 开始方式

1. 拉取兴趣画像：从Zotero拉取有摘要的论文作为个人兴趣语料库，按集合路径glob过滤
2. 抓取昨日新论文：从arXiv RSS/bioRxiv/medRxiv API抓取昨日新论文，构造候选列表
3. 相似度加权排序：用embedding计算候选与兴趣语料的cosine相似度，按时间对数衰减加权求和排序
4. 生成一句话总结：LLM为每篇推荐论文生成TLDR和作者机构，失败时降级为abstract原文
5. 渲染并推送邮件：渲染HTML邮件，通过SMTP发送给订阅者，GitHub Actions cron每日自动触发

## 环境要求

- 准备一个Zotero账号并在文献库中积累有摘要的论文条目（作为兴趣画像基础）
- 在Zotero中按研究领域建立集合目录，用于过滤兴趣范围
- 在GitHub仓库中配置Actions secrets（Zotero API密钥、SMTP邮箱授权码、OpenAI API密钥）
- 确定要追踪的arXiv分类代码（如cs.AI、q-bio.GN）和bioRxiv/medRxiv分类

## 关键限制

- 把它当作完整文献综述工具（它只看摘要embedding相似度，不评估论文质量、引用数或撤稿状态）
- 忘记配置Zotero集合路径过滤或glob模式写错（corpus为空时工具直接停止整轮，不发送任何邮件）
- 用邮箱登录密码而非SMTP授权码配置GitHub Secrets（会导致login失败）
- 将AGPL-3.0许可的工具代码直接集成到闭源或SaaS项目中（强copyleft传染条款要求衍生代码开源）

## 已核对范围

- 读取 README 中排序算法说明（加权平均相似度 + 新论文权重更高）
- 读取 reranker/base.py 的 rerank 和时间衰减加权公式
- 读取 reranker/local.py 和 reranker/api.py 的 embedding 实现
- 读取 executor.py 中 fetchzoterocorpus、filtercorpus 和 rerank 调用链
- 读取 tests/reranker 下 testbasereranker、testlocalreranker、testapireranker
- 读取 README 中多源抓取、RSS 入口和 category 配置说明
- 读取 arxivretriever / biorxivretriever / medrxivretriever 的抓取、解析和 Paper 对象构造路径
- 读取 retriever/base.py 的注册器和 converttopaper 抽象

## 仍待核对

- 未真实加载 embedding 模型
- 未验证时间衰减权重在真实 Zotero 库上的排序效果
- 未验证 maxpapernum 截断对推荐质量的影响
- 未真实请求 arXiv RSS / arXiv API / bioRxiv / medRxiv
- 未验证 LaTeX tar 解析在当前 arXiv 源码格式下的成功率
- 未验证 PDF 提取在真实论文上的稳定性
- 未真实发送邮件
- 未真实调用 LLM 生成 TLDR 和机构

## 许可边界

本页的能力分析、结构与说明由 bo老师整理，采用 CC BY 4.0。上游项目、代码、名称和商标继续遵循各自许可。这里不重新发布上游代码。
