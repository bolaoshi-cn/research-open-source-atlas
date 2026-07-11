# OpenScholar

用检索增强生成回答科研问题，并自动插入带编号的引用文献

适合需要基于学术文献库生成带引用科研回答的研究者，可选用离线检索或在线 Semantic Scholar 补充论文。先构建文献索引或准备候选上下文，再让系统从论文片段中检索证据并生成约束引用格式的答案。

- 官网详情：https://bolaoshi.cn/research-tools/openscholar
- 原始来源：https://github.com/AkariAsai/OpenScholar
- 读取版本：0e9b8fb912273d3dae39e593da86e4f6d3bf8de1
- 核对日期：2026-06-16
- GitHub Stars：1,557（2026-07-11）
- 上游许可：Apache-2.0
- 验证程度：catalogued
- 编辑判断：worth-watching

## 什么时候使用

- 需要对一个科研问题生成基于真实论文片段和编号引用的综述性答案
- 已有大量论文 PDF 或文本，想建立本地可检索的文献证据库
- 需要对已生成的答案事后逐句插入缺失的引用标记
- 需要让系统自我审查初始答案的覆盖缺口并补充检索和改写

## 先准备什么

- 研究问题或查询列表
- 论文语料或已有候选上下文（ctxs）
- Semantic Scholar API 密钥（如需在线补充检索）
- GPU 环境（如需加载本地 OpenScholar-8B 模型）
- 候选论文的标题、摘要和正文片段

## 它会怎样推进

1. **准备候选论文池**：从离线索引、Semantic Scholar 或补充检索获取候选论文的标题和正文片段
2. **重排与筛选**：用 cross-encoder 对候选段落按相关性重排，可叠加引用次数加权
3. **约束引用生成答案**：将前 N 个论文片段格式化为编号参考文献，让模型基于这些片段生成带引用的答案
4. **反馈与改写**：可选让模型审查初稿答案，生成改进建议并补充检索新文献后改写答案
5. **事后引用归因**：对生成答案中未带引用的陈述逐句检查，在确认被片段支持时才插入引用编号

## 第一次这样开始

帮我安装这个库：https://github.com/AkariAsai/OpenScholar
安装前先告诉我需要什么环境，安装后帮我跑通一个最小示例。

## 结果出来后先看什么

- 答案中的引用编号是否都能回溯到具体的论文片段
- 事后归因插入的引用是否被模型强行插入弱支持陈述中
- cross-encoder 重排分数与引用次数加权是否合理（引用次数不等于证据质量）
- 反馈改写后的答案是否保留了原答案中正确的内容

## 常见误区

- 把引用次数高的论文等同于强证据——引用次数只是影响力信号，不能替代原文内容检查
- 让事后引用归因和生成时证据处于同一模型调用中，导致无法区分“原支持”和“补引用的推测”
- 依赖 Semantic Scholar 或 peS2o 在线服务但未配置 API 密钥——部分路径会在启动阶段因缺环境变量直接失败

## 能力拆解

### 1. OpenScholar训练入口与模型产物路径

把 OpenScholar 的模型产物来源登记出来：embedding model 用 peS2o 继续训练 Contriever，reranker 用 FlagEmbedding 训练 BGE reranker，generator 用 OSTrainData 和修改版 torchtune 训练 Llama 3.1 8B。

证据：source-reviewed

输入

- peS2o 训练数据。
- FlagEmbedding reranker training data。
- OpenScholar/OSTrainData，README 声称约 13k instruction-tuning data。

输出

- OpenScholar 生成模型、reranker 模型和 embedding model 的训练来源登记。
- 供推理能力卡引用的模型产物边界。

人工检查

- 训练数据许可证、模型权重许可和非商业条款需人工确认。
- 需要确认 OpenScholar 最终训练所用 config 与仓库示例 config 是否一致。
- 需要确认训练产物是否可在本地工作流中使用。

### 2. RAG证据排序与引文约束生成

把 query 和候选 ctxs 转成带编号 references 的提示词，按可选 cross-encoder 重排、引用数归一加权、每篇论文片段上限和任务模板生成科研回答，并把输出、初稿、成本和耗时写回结果文件。

证据：source-reviewed

输入

- inputfile 或 Hugging Face dataset。
- ctxs：每个候选含 title、text，可选 abstract、citationcounts。
- 模型参数：modelname、api、apikeyfp、downloaddir、llama3。

输出

- outputfile JSON。
- 每个 item 的答案、初稿、排序结果、引用上下文、成本和耗时。

人工检查

- 引用是否真正支持答案需要人工或自动引用归因复核。
- reranker 分数与 citation count 混合权重需要按任务校准。
- API 模型和本地模型输出格式是否稳定，需要小样本确认。

### 3. peS2o大规模索引构建与离线搜索

把 peS2o 论文语料切成分片 passages，生成检索向量，构建 FAISS 或 BM25 索引，再按 query 批量检索 top-k passages，并把检索结果写回评测数据。

证据：source-reviewed

输入

- peS2o v2 或 v3 JSONL 目录。
- Hydra 配置：tasks.datastore.embedding、tasks.datastore.index、tasks.eval.search、datastore.rawdatapath、datastore.embedding.numshards、datastore.embedding.shardids、datastore.index.indexshardids。
- 模型配置：query encoder、datastore encoder、tokenizer、projection size、FP16 开关。

输出

- 供 OpenScholar 推理读取的离线 retrieval results。
- 可被 HTTP 服务加载的 FAISS index 与 passage map。

人工检查

- 确认 peS2o 数据许可、下载范围和本地存储成本。
- 确认检索结果是否可作为候选，不能直接当作支持证据。
- 确认 shard 和 index 的路径命名能被下游稳定读取。

### 4. 事后引用归因插入

在答案已经生成后，逐句或逐段找出未带引用的陈述，把陈述和 references 一起交给模型，要求只在被 passages 支持时插入引用编号，再把替换后的答案写回 item["output"]。

证据：source-reviewed

输入

- 已生成的 item["output"]。
- item["finalpassages"]，或在缺少该字段时由 item["ctxs"] 拼接 passages。
- 生成或 API 模型。

输出

- 追加引用后的 item["output"]。
- attribution 调用成本。

人工检查

- 插入的引用是否真正支持陈述，需要人工或独立 verifier 复核。
- 多句合并段落可能包含多个主张，单个引用不能自动覆盖全部主张。
- 对高风险科研结论，不能只凭 posthoc 引用通过质量门。

### 5. 反馈驱动答案改写与补充检索

在初始答案之后让模型生成反馈和可选补充问题，再按反馈改写答案。若反馈包含新问题且启用 Semantic Scholar 检索，系统会尝试补充新论文候选、重排新片段，并用新增证据改写答案。

证据：source-reviewed

输入

- 已生成 item["output"]。
- 已格式化 item["finalpassages"]。
- 可选：feedback=True、ssretriever=True。

输出

- 更改后的答案。
- 反馈记录和每轮改写稿。
- 追加后的上下文候选。

人工检查

- 反馈是否真指出事实或覆盖缺口，需要人工或独立标准复核。
- 新增论文是否支持改写内容，需要引用归因或人工读原文。
- 改写是否改变了原答案中已经正确的内容，需要对比检查。

### 6. 多源论文候选补充与上下文标准化

把已有离线检索结果、Semantic Scholar 搜索结果、arXiv 或 PubMed 网页解析结果、You.com 搜索结果和 peS2o 检索结果合并到同一个 ctxs 上下文列表中，使后续重排、生成和引用步骤都能按统一字段读取候选论文片段。

证据：source-reviewed

输入

- jsonl、json 或 tsv 输入文件。
- 字段：input、question、query、ctxs、retrieval text、pes2opaperid、origctxs。
- 环境变量：S2APIKEY。You.com 相关代码还需要 YOURAPIKEY，但源码中变量定义被注释。

输出

- 标准化后的 ctxs。
- 外部检索候选和已有离线候选的合并文件。
- 供重排、生成、反馈改写和引用归因使用的上下文对象。

人工检查

- 确认 API key、费用、限流和数据使用条款。
- 确认候选论文是否真能支撑回答，候选状态不能直接当作证据支持。
- 复核跨来源去重规则，尤其是同名论文、版本论文和低质量摘要。

## 处理机制

1. **科研RAG对象与状态链路**：OpenScholar 的多个能力围绕同一条对象链路运行：query 进入候选上下文，候选上下文变成 references，references 和答案在反馈、改写和引用归因中继续被读写。这个横向机制解释能力卡之间共享的对象、状态变化和断点。

## 开始方式

1. 准备候选论文池：从离线索引、Semantic Scholar 或补充检索获取候选论文的标题和正文片段
2. 重排与筛选：用 cross-encoder 对候选段落按相关性重排，可叠加引用次数加权
3. 约束引用生成答案：将前 N 个论文片段格式化为编号参考文献，让模型基于这些片段生成带引用的答案
4. 反馈与改写：可选让模型审查初稿答案，生成改进建议并补充检索新文献后改写答案
5. 事后引用归因：对生成答案中未带引用的陈述逐句检查，在确认被片段支持时才插入引用编号

## 环境要求

- 研究问题或查询列表
- 论文语料或已有候选上下文（ctxs）
- Semantic Scholar API 密钥（如需在线补充检索）
- GPU 环境（如需加载本地 OpenScholar-8B 模型）
- 候选论文的标题、摘要和正文片段

## 关键限制

- 把引用次数高的论文等同于强证据——引用次数只是影响力信号，不能替代原文内容检查
- 让事后引用归因和生成时证据处于同一模型调用中，导致无法区分“原支持”和“补引用的推测”
- 依赖 Semantic Scholar 或 peS2o 在线服务但未配置 API 密钥——部分路径会在启动阶段因缺环境变量直接失败

## 已核对范围

- 读取 README 的 embedding、reranker 和 generator training 说明
- 读取 training/README.md、training/pyproject.toml 和 llama31 LoRA 配置
- 读取 lorafinetunedistributed.py 的 recipe 状态、checkpoint、optimizer、dataset 和 dataloader setup
- 读取 README 的 standard RAG、Retriever+Reranker 和 proprietary LM 示例
- 读取 run.py 的 CLI 参数、模型装载和 OpenScholar.run 调用
- 读取 src/openscholar.py 的重排、topn、生成和输出字段
- 读取 retriever README 和 buildpes2oindex.md 的索引构建、搜索和服务说明
- 读取 retriever/ric/mainric.py 的任务开关

## 仍待核对

- 未下载 OSTrainData 或 Meta Llama 权重
- 未运行 torchtune 或 torchrun
- 未验证 OpenScholar-8B 训练数据格式
- 未验证 reranker 训练数据格式
- 未加载 OpenScholar/Llama-3.1OpenScholar-8B
- 未加载 OpenScholar/OpenScholarReranker
- 未调用 OpenAI、Together 或 Anyscale
- 未验证生成答案的引用正确性

## 许可边界

本页的能力分析、结构与说明由 bo老师整理，采用 CC BY 4.0。上游项目、代码、名称和商标继续遵循各自许可。这里不重新发布上游代码。
