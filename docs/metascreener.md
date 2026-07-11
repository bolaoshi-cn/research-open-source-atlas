# MetaScreener

多LLM协作的系统综述自动化平台，从检索到偏倚评估全链路

适合做系统综述或Meta分析的研究团队。MetaScreener用4个以上LLM并行独立判断每篇文献的纳入排除，通过四层共识机制做出决策，任何不确定结果自动路由至人工复核。

- 官网详情：https://bolaoshi.cn/research-tools/metascreener
- 原始来源：https://github.com/ChaokunHong/MetaScreener
- 读取版本：532ee3cc10a7e22e54747d7c3413509c31b9a99b
- 核对日期：2026-07-10
- GitHub Stars：1,319（2026-07-11）
- 上游许可：Apache-2.0
- 验证程度：catalogued
- 编辑判断：worth-watching

## 什么时候使用

- 需要做系统综述的标题摘要筛选，希望用多LLM投票减少单模型偏差
- 需要对大量文献做PICO标准的结构化数据和效应量提取
- 需要用RoB 2、ROBINS-I或QUADAS-2工具评估纳入研究的偏倚风险
- 需要在PubMed、Scopus等多个数据库同时检索并自动去重

## 先准备什么

- 明确PICO要素（人群、干预、对照、结局）和研究设计纳入排除标准
- 准备OpenRouter API密钥用于调用多个LLM后端
- 上传待筛选文献（支持RIS、BibTeX、CSV、Excel格式）
- 如需要PDF全文筛选，确保网络可访问文献下载源

## 它会怎样推进

1. **生成PICO标准**：用多模型共识生成PICO纳排标准，人工审核后锁定
2. **检索与去重**：在多数据库中执行检索并六层去重，下载PDF全文
3. **HCN四层筛选**：多个LLM并行独立判断，经规则修正和校准聚合后路由决策
4. **数据提取**：对纳入文献自动提取PICO结构化数据和效应量
5. **偏倚评估**：用RoB 2等工具评估每篇纳入文献的偏倚风险

## 第一次这样开始

帮我安装这个库：https://github.com/ChaokunHong/MetaScreener
安装前先告诉我需要什么环境，安装后帮我跑通一个最小示例。

## 结果出来后先看什么

- 检查Tier 3（人工复核）队列的纳入排除决策是否合理
- 抽查自动提取的效应量数据与原文是否一致
- 验证去重后的文献集是否还存在重复条目

## 常见误区

- 把Tier 1和Tier 2的自动决策当作最终结果而不人工抽检
- 跳过PICO标准的人工审核环节而直接用模型生成的版本
- 把MetaScreener的偏倚评估结果替代研究者对研究质量的独立判断

## 能力拆解

### 1. HCN四层共识筛选决策

把一条文献记录（title+abstract 或 full-text）和 PICO 纳排标准，通过 4+ 开源 LLM 并行独立判断、语义规则修正、校准置信度聚合，最终路由成带 tier 的纳入/排除/人工复核决策，且保守策略保证任何不确定性默认进人工复核而非自动排除。

证据：source-reviewed

输入

- Record（recordid、title、abstract、authors、year、doi、pmid、fulltext）。
- PICOCriteria（framework、elements、studydesigninclude/exclude、publicationtypeexclude、languagerestriction）。
- LLM backends 列表（4+ OpenRouterAdapter）。

输出

- ScreeningDecision（recordid、decision、tier、confidence、modeloutputs、ruleadjustments、elementscores、rationale）。
- SSE 事件流（progress、pdfstart、pdfdone）。
- 导出 CSV/Excel/JSON/RIS（writers.py）。

人工检查

- Tier 3 记录全部进入人工复核队列（核心人工门）。
- 试点批次（pilotsize=5）结果需用户审查并反馈，反馈进入主动学习校准。
- 用户可 override 单条决策（/api/screening/results/{id}/override），反馈纳入后续校准。

### 2. PDF结构化数据提取

把一篇 PDF（经 docengine 解析为 StructuredDocument）和一份 Excel 提取模板，通过模板编译、启发式字段路由、双模型+双 prompt 并行提取、四层验证、效应量计算与置信度聚合，转成带置信度分级和证据句的结构化字段数据，可导出 RevMan XML / R metafor CSV / 填充模板。

证据：source-reviewed

输入

- Excel 模板（.xlsx，含 DATA/MAPPING/REFERENCE sheet）。
- PDF 文件（经 docengine 解析为 StructuredDocument：sections/tables/figures/references）。
- 双 LLM backend（backenda、backendb）。

输出

- DocumentExtractionResult（docid、sheets、errors）。
- 导出：模板填充 Excel（保留格式）、平面 Excel（置信度颜色编码）、CSV、RevMan XML、R metafor CSV。
- SSE 事件（progress、pdfstart、docparsed、pdfdone、pdferror、batchdone）。

人工检查

- LOW/SINGLE/FAILED 字段需人工审查（平面 Excel 颜色编码辅助）。
- 仲裁结果与 V4 数值冲突需抽查。
- 导出数据用于 Meta 分析前需人工核对关键数值。

### 3. PICO标准多模型共识生成

把一个研究问题（topic）或已有标准文本，通过多 LLM 并行生成 + Delphi 共识投票 + 术语增强 + 质量验证，转成结构化的 PICO/PEO/SPIDER/PCC 纳排标准对象，供下游检索与筛选使用。

证据：source-reviewed

输入

- topic（研究问题文本）或已有标准文本（parsetextv1）或样例（inferfromexamplesv1）。
- LLM backends 列表。
- mode（thorough 等）。

输出

- PICOCriteria 对象（Pinia store + localStorage 持久化，跨页面共享）。
- 供 metascreener-能力-多数据库文献检索与六层去重 的 query 构建和 metascreener-能力-HCN四层共识筛选决策 的筛选使用。

人工检查

- 生成的标准需用户审查（CriteriaView 页），可手动编辑、refine 单个元素或导入已有标准。
- 就绪度评分引导用户判断是否达到可筛选门槛。

### 4. 偏倚风险评估

把一篇 PDF 全文和研究类型，通过文本分块、所有分块×所有模型并行评估、按模型最差情况合并分块、跨模型多数投票、工具驱动总体判定，转成带领域判定/理由/支持引文/人工复核标记的偏倚风险结果，支持 RoB 2（RCT）、ROBINS-I（观察性）、QUADAS-2（诊断准确性）三种工具。

证据：source-reviewed

输入

- PDF 全文文本（经 io 层 chunktext 分块，6000 tokens/块，200 重叠）。
- toolname（手动）或 studytype（自动选择：RCT/OBSERVATIONAL/DIAGNOSTIC）。
- LLM backends 列表。

输出

- RoBResult（overalljudgement、domainresults、requireshumanreview）。
- 前端交通灯矩阵（RoB 热力图，evaluation/visualizer）。
- 历史会话 JSON。

人工检查

- requireshumanreview=true 的领域全部进人工复核。
- 总体判定需人工确认（最差情况可能偏严）。
- 支持引文需人工核对是否真正支持判定。

### 5. 多数据库文献检索与六层去重

把 PICO 标准转成数据库无关的布尔查询 AST，翻译成 5 个数据库的原生语法并行检索，再用六层渐进式去重（精确 ID → 标题年份 → 语义相似度）合并跨库重复，输出带完整审计日志的规范记录，可选级联下载 PDF 与智能 OCR。

证据：source-reviewed

输入

- PICOCriteria（来自标准生成）。
- 检索源选择（PubMed/OpenAlex/EuropePMC/Scopus/S2）。
- maxresults。

输出

- RetrievalResult（searchcounts、totalfound、dedupcount、downloaded、ocrcompleted、records）。
- 去重后 RawRecord 列表（可导出 RIS 供筛选）。
- DedupResult.mergelog 完整审计日志。

人工检查

- 跨库弱匹配（L5 标题+年份）需人工复核合并正确性。
- 语义去重合并（L6）边界情况需抽查。
- 检索结果导出 RIS 后，用户决定是否进入筛选。

### 6. 筛选性能评价与校准

把筛选决策和金标准标签，通过混淆矩阵、AUROC、校准指标、Bootstrap 置信区间计算，转成系统综述标准性能报告（sensitivity/specificity/WSS@95/Brier/校准曲线），并可基于验证数据优化决策阈值与拟合校准器，回灌 HCN Layer 3/4。

证据：source-reviewed

输入

- 筛选决策列表（ScreeningDecision，含 recordid、decision、confidence、score）。
- 金标准标签（recordid → include/exclude）。
- seed=42、niter=1000（Bootstrap）。

输出

- EvaluationReport（指标 + AUROC + 校准 + Bootstrap CI + 元数据）。
- 可视化（ROC/校准曲线/混淆矩阵/Tier 分布/RoB 热力图）。
- Lancet 格式化文本（发表就绪）。

人工检查

- 优化阈值回灌前需人工确认（影响后续筛选行为）。
- 校准曲线偏差大时需人工判断是否重新校准。
- WSS@95 与 automationrate 的取舍需研究者判断。

## 处理机制

1. **HCN四层与分层决策路由**：解释 MetaScreener 多个模块（筛选、提取、RoB 评估）怎样共享一套"并行多模型推理 → 规则修正 → 校准聚合 → 分层路由（含人工门）"的骨架，同时在产物类型（决策/字段/判定）上分化，以及保守策略（不确定→人工、recall-first、最差情况合并）如何贯穿全系统成为共同运行治理。

## 开始方式

1. 生成PICO标准：用多模型共识生成PICO纳排标准，人工审核后锁定
2. 检索与去重：在多数据库中执行检索并六层去重，下载PDF全文
3. HCN四层筛选：多个LLM并行独立判断，经规则修正和校准聚合后路由决策
4. 数据提取：对纳入文献自动提取PICO结构化数据和效应量
5. 偏倚评估：用RoB 2等工具评估每篇纳入文献的偏倚风险

## 环境要求

- 明确PICO要素（人群、干预、对照、结局）和研究设计纳入排除标准
- 准备OpenRouter API密钥用于调用多个LLM后端
- 上传待筛选文献（支持RIS、BibTeX、CSV、Excel格式）
- 如需要PDF全文筛选，确保网络可访问文献下载源

## 关键限制

- 把Tier 1和Tier 2的自动决策当作最终结果而不人工抽检
- 跳过PICO标准的人工审核环节而直接用模型生成的版本
- 把MetaScreener的偏倚评估结果替代研究者对研究质量的独立判断

## 已核对范围

- 读取 README 的 HCN 四层架构图与 Decision Router tier 定义
- 读取 docs/SYSTEMARCHITECTURE.md 第 11 章 HCN 四层筛选系统
- 读取 hcnscreener.py、layer1/inference.py、layer2/ruleengine.py、layer3/aggregator.py、layer4/router.py 源码骨架
- 读取 core/modelsscreening.py 的 Record/ModelOutput/ScreeningDecision 字段
- 读取 docs/SYSTEMARCHITECTURE.md 第 12 章 PDF 数据提取系统与第 14 章 Meta 分析导出
- 读取 module2extraction/engine/neworchestrator.py、fieldrouter.py、llmextractor.py、computation.py、arbitrator.py 源码骨架
- 读取 module2extraction/validation/ 四层验证与 export/ 导出策略
- 读取 tests/unit/testextractionschema.py、testfieldschemaextension.py、testnumericalcoherence.py、testcompilerpipeline.py

## 仍待核对

- 未调用 OpenRouter API，未验证多模型并行推理实际输出
- 未运行任何筛选 smoke 或 Cohen/ASReview benchmark
- 未验证校准器在真实金标准上的拟合行为
- msactive 主动学习与 layer3 贝叶斯路由（bayesianrouter.py/glad.py/dawidskene.py）的运行行为未深读
- 未运行任何提取 smoke
- 未验证双模型 Alpha/Beta prompt 的实际分歧率
- 效应量计算在真实数据上的正确性未验证
- RevMan XML/R metafor 导出的下游可用性未验证

## 许可边界

本页的能力分析、结构与说明由 bo老师整理，采用 CC BY 4.0。上游项目、代码、名称和商标继续遵循各自许可。这里不重新发布上游代码。
