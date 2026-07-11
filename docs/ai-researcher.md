# AI Researcher

把机器学习研究想法或参考论文自动转成可运行实验和LaTeX论文

面向机器学习研究者，适合已有研究想法或参考论文后需要快速实现验证的场景。系统先检索相关代码和论文，再由多Agent协作完成实验实现、评审修正和论文撰写。

- 官网详情：https://bolaoshi.cn/research-tools/ai-researcher
- 原始来源：https://github.com/HKUDS/AI-Researcher
- 读取版本：f9a6f8480860c193afff600eeffe3defcee8a978
- 核对日期：2026-06-16
- GitHub Stars：5,576（2026-07-11）
- 上游许可：not-found
- 验证程度：catalogued
- 编辑判断：use-with-caution

## 什么时候使用

- 已有明确的研究想法，需要快速实现代码并验证可行性
- 拿到一篇参考论文，希望在其基础上产生创新想法并实验
- 需要构建新的机器学习基准数据集供后续研究使用
- 希望自动完成实验代码、评审修正和论文初稿撰写

## 先准备什么

- 准备研究想法描述或参考论文的标题和arXiv链接
- 配置OpenAI API Key和GitHub Token用于模型调用和搜索
- 准备Docker运行环境，确保GPU端口和缓存目录可用
- 确认参考代码仓库和数据集的许可协议允许复用

## 它会怎样推进

1. **检索资源**：根据参考论文搜索GitHub和arXiv，选择最相关的候选代码仓库和论文源码
2. **生成计划**：根据研究想法和检索结果制定数据集、模型、训练测试的完整方案
3. **实现实验**：在Docker容器中编写项目代码，完成真实数据的两轮训练和测试
4. **评审修正**：对实现代码逐项审查，反复修正直至满足研究想法的全部要求
5. **分析提炼**：分析实验结果，生成下一轮实验计划并补充可视化或新实验
6. **撰写论文**：根据实验结果和代码生成各章节LaTeX论文，处理引用并编译

## 第一次这样开始

帮我安装这个库：https://github.com/HKUDS/AI-Researcher
安装前先告诉我需要什么环境，安装后帮我跑通一个最小示例。
请先说明许可证和安装风险，等我确认后再安装。

## 结果出来后先看什么

- 检查生成的项目代码是否完整运行，训练测试结果是否合理
- 验证评审建议已被采纳，代码覆盖了研究想法的全部学术概念
- 确认生成的论文忠实反映实验结果，引用和图表准确无误
- 审查实验分析报告中的结论是否有真实数据支撑

## 常见误区

- 在没有明确研究想法时启动系统，生成方向可能偏离预期
- 忽略候选代码仓库和数据集的使用许可，未确认版权风险
- 把自动评审的fully_correct当作绝对可信，跳过人工复核
- 让系统在缺少GPU、Docker或API Key时强行运行导致中断

## 能力拆解

### 1. Level1给定想法到实验实现闭环

把用户已经给出的机器学习研究 idea、benchmark instance、参考论文和候选代码仓，转成可运行项目、两轮训练测试、评审建议、提交实验结果和后续实验 refinements。

证据：source-reviewed

输入

- benchmark instance：benchmark/final/<category/<instanceid.json，包含 sourcepapers、task1 或 task2、目标论文 URL 和 instance id。
- 用户给定 idea：runinferplan.main(args, ideas, references) 的 ideas 参数。
- 参考论文文本：由 warpsourcepapers 或 Web GUI reference 输入进入。

输出

- /{workplacename}/project/ 下的代码、数据处理、训练和测试脚本。
- 两轮训练测试的运行结果。
- judge 评审建议和可能的 fullycorrect 状态。

人工检查

- 是否允许外部模型读取论文、代码和实验结果。
- 是否允许 Docker 拉起容器并挂载工作目录。
- GPU、端口、缓存目录和输出目录是否可用。

### 2. Level2参考论文到创新想法实现

把用户只提供参考论文、没有明确创新 idea 的任务，转成多个候选研究想法、一个精选并细化的 idea、代码实现调查、实验实现、评审修正和实验 refinement。

证据：source-reviewed

输入

- benchmark instance：benchmark/final/<category/<instanceid.json。
- 参考论文列表：用户输入或 benchmark JSON 中的 sourcepapers。
- 类别任务说明：benchmark.process.datasetcandidate.<category.metaprompt 的 TASK 和 dataset 描述。

输出

- 多个 candidate idea。
- 精选和增强后的 idea report。
- code survey implementation report。

人工检查

- 生成 idea 是否真实新颖，是否和参考论文冲突。
- 参考代码和数据集是否允许复用。
- code survey 是否覆盖了 idea 中每个学术概念。

### 3. 创新基准数据构建

把一组论文标题或关键词、PDF 和 LLM 指令模板，转成包含目标论文、source papers、task1、task2、领域元数据和匿名化处理的 innovation benchmark 数据。

证据：source-reviewed

输入

- benchmarkcollection/paperstosearch：论文标题或关键词。
- exactmatch：是否精确匹配标题。
- arXiv、Semantic Scholar 和 Crossref API。

输出

- papertitles/papertitles.json
- papertitles/pdfs/.pdf
- innovationgraph/innovationgraph.json

人工检查

- 论文标题和 PDF 是否匹配。
- source paper 是否真的支撑 target task。
- LLM 生成的 task 是否保留作者原意。

### 4. 参考论文与代码资源选择下载

把 benchmark 中的参考论文、日期边界和用户任务，转成一组 reference codebases、reference paths、reference papers，并把被选论文的 arXiv source 下载成本地 TeX 文件。

证据：source-reviewed

输入

- metadata["sourcepapers"]：每项包含 reference 和 usage。
- metadata["datelimit"]：来自目标论文 arXiv metadata。
- GitHub token：GITHUBAITOKEN。

输出

- referencecodebases
- referencepaths
- referencepapers

人工检查

- reference codebases 是否真的对应论文。
- arXiv title 是否匹配正确版本。
- 下载的论文 source 是否可用于后续模型调用。

### 5. 实验实现评审与迭代修正

把 ML agent 已经写出的项目、研究 idea、计划、survey notes、训练测试结果和 reference codebases，转成实现审查、修正建议、再次实现、正式提交运行结果、实验分析报告和下一轮实验计划。

证据：source-reviewed

输入

- innovative idea 或精选 idea。
- prepare agent 的 reference codebases。
- coding plan 和 model survey notes。

输出

- code review report。
- judge fullycorrect 判断和 suggestion。
- 修正后的 project/。

人工检查

- judge suggestion 是否可信。
- 是否继续消耗 GPU 做修正和进一步实验。
- submit 阶段是否允许只改 epochs。

### 6. 研究结果到LaTeX论文生成

把研究领域、instance id、实验项目、agent 输出、benchmark 数据、相关论文和写作模板，转成完整 LaTeX 论文各章节、清理后的引用文件和可尝试编译的论文项目。

证据：source-reviewed

输入

- researchfield：例如 vq 或 rec。
- instanceid：目标论文或项目 id。
- agent 输出目录、model 代码目录、project 目录和 benchmark JSON。

输出

- methodology.tex
- relatedwork.tex
- experiments.tex

人工检查

- 论文目标会议和格式是否正确。
- 章节是否引用真实实验结果。
- 相关工作引用是否可查。

## 处理机制

1. **Docker执行环境与浏览器下载边界**：解释 AI-Researcher 如何把 agent 的文件读写、命令执行、Python 运行、网页下载和数据集准备限制在 Docker workspace 与本地挂载目录之间，以及这个机制有哪些失败边界。
2. **MetaChain多agent工具调用与缓存**：解释 AI-Researcher 多张能力卡共享的 agent 运行骨架：模型消息怎样调用工具、工具结果怎样进入上下文、agent 怎样切换、错误怎样留在消息里、缓存怎样影响长流程恢复。
3. **基准实例与产物目录状态链**：解释 AI-Researcher 如何把 benchmark instance、dataset candidate、workspace project、paper source、paper agent target sections 和 cache/log 组织成一条产物状态链。

## 开始方式

1. 检索资源：根据参考论文搜索GitHub和arXiv，选择最相关的候选代码仓库和论文源码
2. 生成计划：根据研究想法和检索结果制定数据集、模型、训练测试的完整方案
3. 实现实验：在Docker容器中编写项目代码，完成真实数据的两轮训练和测试
4. 评审修正：对实现代码逐项审查，反复修正直至满足研究想法的全部要求
5. 分析提炼：分析实验结果，生成下一轮实验计划并补充可视化或新实验
6. 撰写论文：根据实验结果和代码生成各章节LaTeX论文，处理引用并编译

## 环境要求

- 准备研究想法描述或参考论文的标题和arXiv链接
- 配置OpenAI API Key和GitHub Token用于模型调用和搜索
- 准备Docker运行环境，确保GPU端口和缓存目录可用
- 确认参考代码仓库和数据集的许可协议允许复用

## 关键限制

- 在没有明确研究想法时启动系统，生成方向可能偏离预期
- 忽略候选代码仓库和数据集的使用许可，未确认版权风险
- 把自动评审的fully_correct当作绝对可信，跳过人工复核
- 让系统在缺少GPU、Docker或API Key时强行运行导致中断

## 已核对范围

- README Level 1 usage
- researchagent/runinferplan.py
- researchagent/inno/agents/innoagent/
- researchagent/inno/workflow/flowcache.py
- researchagent/inno/environment/dockerenv.py
- README Level 2 usage
- researchagent/runinferidea.py
- ideaagent and codesurveyagent definitions

## 仍待核对

- true LLM calls
- Docker execution
- GPU training
- external GitHub and arXiv calls
- generated project correctness
- generated idea novelty
- code survey completeness
- true end to end run

## 许可边界

本页的能力分析、结构与说明由 bo老师整理，采用 CC BY 4.0。上游项目、代码、名称和商标继续遵循各自许可。这里不重新发布上游代码。
