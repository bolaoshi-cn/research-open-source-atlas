# ChatPaper

围绕论文 PDF 完成总结、翻译、审稿和综述草稿。

把 arXiv 候选获取、PDF 解析、模型总结、全文翻译和审稿回复封装为脚本与轻量入口。

- 官网详情：https://bolaoshi.cn/research-tools/chatpaper
- 原始来源：https://github.com/kaixindelele/ChatPaper
- 读取版本：f19b3a4510c24d49a14549a41de4ad4dd32dc6c1
- 核对日期：2026-06-16
- 上游许可：CC-BY-NC-ND-4.0
- 验证程度：catalogued
- 编辑判断：use-with-caution

## 适合这些任务

- 希望了解论文处理脚本结构的研究者
- 需要比较总结、翻译和审稿入口的开发者

## 这些情况要谨慎

- 需要商业改造和再分发的项目
- 无法核查模型输出和引用的用户

## 能力拆解

### 1. PDF全文翻译与润色

把一篇本地 PDF 解析为标题、摘要和章节列表，先判断论文领域，再逐段调用 ChatCompletion 做中文翻译或润色，并把结果追加写成 Markdown 全文。

证据：source-reviewed

输入

- pdfpath：本地 PDF 文件。
- rootpath：输出 Markdown 所在目录。
- baseurl 和 key：OpenAI 兼容接口地址和 API key。

输出

- 和 PDF 同名的 Markdown 翻译或润色文件。
- token 使用量日志。

人工检查

- 翻译是否保留原始术语、公式、引用和章节层级，需要人工目检。
- 领域判断错误会影响后续术语表达。
- 若要复用为正式论文翻译，需要补术语表和格式校验。

### 2. PDF章节解析与三段式论文总结

把本地 PDF 或 arXiv 下载后的 PDF 解析成标题、摘要、章节和正文片段，再分三次调用 ChatCompletion，生成面向论文初筛的固定格式总结、方法说明和结论评价。

证据：source-reviewed

输入

- 本地 PDF 文件或目录。
- 可选 arXiv 查询和摘要关键词过滤结果。
- apikey.ini 或环境变量中的 OpenAI key。

输出

- export/ 下的 Markdown 或 TXT 总结文件。
- 终端日志中的 token、响应时间和模型输出。

人工检查

- 输出能否用于精读决策，需要人工确认。
- 总结中的方法、数值和 Github 链接需要回原文复核。
- 本地复用前需要确认 OpenAI key、代理、模型和许可证边界。

### 3. arXiv近期论文搜索下载与候选筛选

把用户关键词、页数、时间窗口和结果数量转成 arXiv 网页搜索请求，按提交日期筛出近期论文，再下载 PDF 并构造待总结的 Paper 对象列表。

证据：source-reviewed

输入

- query：arXiv 搜索关键词。
- pagenum：读取搜索页数，每页固定按 arXiv 网页 size=50。
- days：只保留最近几天提交的论文。

输出

- 本地 pdffiles/ 下的 PDF 文件。
- Paper 对象列表。
- 后续 summarywithchat 的输入。

人工检查

- 用户需要确认关键词、时间窗口和结果数量是否符合调研目标。
- 下载后的 PDF 是否真的属于目标主题，需要人工或后续质量门复核。
- 若用于研究证据池，需要补 DOI、版本、撤稿和来源质量检查。

### 4. 审稿回复草拟

把审稿意见文本读入模型，要求模型逐条抽取审稿人 concern，并以作者口吻生成点对点回复草稿，最后写入 response 文件。

证据：source-reviewed

输入

- commentpath：审稿意见文本文件。
- fileformat：输出格式后缀。
- language：输出语言。

输出

- responsefile/ 下的审稿回复草稿。
- 终端中的 token 用量和模型输出。

人工检查

- 每条回复是否对应真实修改。
- 是否需要补充图表、实验或文字修订位置。
- 是否符合目标期刊或会议 rebuttal 格式。

### 5. 审稿意见生成

把一篇论文 PDF 解析成标题、摘要、引言、结论和少量关键章节，先让模型选择最多两个需要补读的章节，再生成包含总体评价、优点、弱点、问题和分数的审稿意见。

证据：source-reviewed

输入

- paperpath：单个 PDF 或包含 PDF 的目录。
- researchfields：审稿领域。
- language：输出语言。

输出

- outputfile/ 下的审稿意见文本。
- 终端中的 token 用量、响应时间和模型输出。

人工检查

- 审稿意见是否真实、准确、公平，需要人工逐条对照论文。
- 是否可以用于正式审稿，需要人工按伦理和期刊规范判断。
- 任何分数和接收倾向都不应自动采用。

### 6. 文献综述草稿生成

把一个研究主题标题转成关键词、参考文献候选、可选领域知识和论文组成部分，再生成 related works 类综述章节、Markdown 中文稿和压缩包。

证据：source-reviewed

输入

- title：目标综述或研究主题。
- template：LaTeX 模板。
- maxkwrefs、bibrefs、maxtokensref：参考文献候选和 prompt 预算。

输出

- 模板目录中的 section .tex。
- survey.md 和 surveychinese.md。
- bibtex 文件。

人工检查

- 参考文献是否相关、可靠、去重，需要人工或后续候选对象质量门复核。
- 草稿中的每个引用是否真的支持对应句子，需要人工核对。
- 中文稿是否保留 citation 和术语一致性，需要人工目检。

## 处理机制

1. **多入口LLM论文处理骨架**：解释 ChatPaper 多个脚本怎样共享一套“材料进入、章节或评论抽取、LLM 处理、文件落盘”的骨架，同时在论文总结、翻译、审稿、审稿回复和综述草稿中分化出不同产物。

## 开始方式

1. 先阅读许可证和配置文件
2. 只处理可合法使用的论文
3. 抽查总结、翻译和引用

## 环境要求

- Python
- 外部 LLM API
- PDF 解析环境

## 关键限制

- 许可包含非商业和禁止演绎限制
- 本轮未调用真实模型
- 综述草稿不能代替证据矩阵

## 已核对范围

- 读取 README 的 PDF 全文翻译配置教程
- 读取 chattranslate.py 的 scipdf 解析、领域判断、章节翻译和 Markdown 写入路径
- 读取 README 的论文总结技术原理和本地 PDF 总结命令
- 读取 chatpaper.py 的 Paper 解析、Reader.summarywithchat 和三类 chat 调用
- 读取 README 中 arXiv 批量搜索命令和参数说明
- 读取 chatarxiv.py 的网页检索、日期过滤、PDF 下载和 Paper 对象构造路径
- 读取 ChatReviewerAndResponse/chatresponse.py
- 核验 commentpath 到 responsefile 的主路径

## 仍待核对

- 未启动 GROBID 或 scipdf 服务
- 未调用 OpenAI API
- 未验证长章节切分效果
- 未运行 PyMuPDF 解析 demo.pdf
- 未验证输出 Markdown 格式
- 未真实请求 arXiv 搜索页
- 未下载 PDF
- 未验证搜索页 HTML 结构在当前日期是否兼容

## 许可边界

本页的能力分析、结构与说明由 bo老师整理，采用 CC BY 4.0。上游项目、代码、名称和商标继续遵循各自许可。这里不重新发布上游代码。
