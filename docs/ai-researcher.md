# AI-Researcher

围绕机器学习论文完成想法、代码实验、评审修正和论文生成。

把参考论文、代码仓、Docker 执行、多 Agent 实验和 LaTeX 写作放进同一套机器学习科研自动化系统。

- 官网详情：https://bolaoshi.cn/research-tools/ai-researcher
- 原始来源：https://github.com/HKUDS/AI-Researcher
- 读取版本：f9a6f8480860c193afff600eeffe3defcee8a978
- 核对日期：2026-06-16
- 上游许可：not-found
- 验证程度：catalogued
- 编辑判断：use-with-caution

## 适合这些任务

- 研究机器学习自动科研系统的开发者
- 需要比较实验 Agent 机制的研究者

## 这些情况要谨慎

- 商业复制或分发上游内容
- 缺少隔离执行环境的用户

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

1. 先核对许可状态
2. 使用隔离容器阅读示例
3. 从单个 benchmark 实例开始

## 环境要求

- Docker
- GPU 或足够计算资源
- 外部模型和代码仓

## 关键限制

- 上游仓库未发现明确许可证
- 本轮只做静态读取
- 浏览器下载和代码执行具有安全风险

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
