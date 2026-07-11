# AI Scientist

从研究想法自动生成、实验验证到论文发表的端到端科研流水线

面向希望探索多个研究方向的机器学习研究者，适合从零生成想法并自动验证的场景。系统先生成新颖研究想法，再通过模板实验改写、运行、论文写作和评审形成完整闭环。

- 官网详情：https://bolaoshi.cn/research-tools/ai-scientist
- 原始来源：https://github.com/SakanaAI/AI-Scientist
- 读取版本：1de1dbc1f4ee2c5f61e9c94348d55eb51d7fa2eb
- 核对日期：2026-06-16
- GitHub Stars：14,197（2026-07-11）
- 上游许可：AI-Scientist-Source-Code-License-1.0
- 验证程度：catalogued
- 编辑判断：use-with-caution

## 什么时候使用

- 希望从零生成多个新颖研究方向并快速验证可行性
- 基于已有模板实验代码，快速探索不同改进方向
- 需要端到端完成实验验证、论文写作和LaTeX编译
- 希望用自动评审快速评估论文质量并迭代改进

## 先准备什么

- 准备模板实验目录，包含可运行的experiment.py和plot.py
- 手动运行一次基线实验，生成baseline对比结果文件
- 配置LLM API Key和Semantic Scholar或OpenAlex访问权限
- 确保Linux环境、CUDA GPU和pdflatex等LaTeX工具链就绪

## 它会怎样推进

1. **生成想法**：基于模板和种子想法生成候选研究想法，用文献检索筛选新颖性
2. **改写实验**：让LLM根据选中想法修改模板实验代码，运行并收集实验结果
3. **撰写论文**：自动撰写各章节论文，搜索补全参考文献并插入BibTeX条目
4. **编译输出**：完成LaTeX编译和格式检查，生成可投稿的PDF文件
5. **评审改进**：用LLM模拟会议评审，可选根据反馈改进论文并重新编译

## 第一次这样开始

帮我安装这个库：https://github.com/SakanaAI/AI-Scientist
安装前先告诉我需要什么环境，安装后帮我跑通一个最小示例。
请先说明许可证和安装风险，等我确认后再安装。

## 结果出来后先看什么

- 检查生成的论文PDF是否完整，图表和引用是否正确嵌入
- 验证实验结果是否真实有效，指标是否与论文报告一致
- 确认引用文献真正支持对应论点，而非仅为格式填充
- 审查LLM评审是否指出实验缺陷，评分是否与意见一致

## 常见误区

- 把LLM自动评审当作正式同行评审的替代品，忽略实验缺陷
- 不准备基线运行结果就启动流水线，实验缺乏对比基准
- 让系统在没有沙箱限制的环境中执行LLM生成代码，存在安全风险
- 忽略AI Scientist许可协议中的披露要求，发表时不声明AI参与

## 能力拆解

### 1. LLM改写实验代码与运行闭环

把一个研究 idea、baseline 结果和模板项目交给 Aider，让 LLM 修改 experiment.py、运行固定格式的 python experiment.py --outdir=runi，按运行结果继续修正实验，再修改 plot.py 和 notes.txt，形成可被论文写作节点读取的实验记录。

证据：source-reviewed

输入

- idea["Title"] 和 idea["Experiment"]。
- baselineresults，由 run0/finalinfo.json 读取并抽取 means。
- 结果目录中的 experiment.py、plot.py 和 notes.txt。

输出

- runi.py
- runi/finalinfo.json
- plot 生成的图片文件。

人工检查

- 实验改动是否仍回答原 idea。
- 是否允许删除失败 run 目录。
- 指标是否能比较 baseline。

### 2. LLM评审集成与改进闭环

把生成论文 PDF 抽取成文本，按机器学习会议 review form 让 LLM 生成一组结构化评审，必要时做多评审集成和反思修订，再把 review JSON 写入结果目录，并可用 review 触发论文改进和二次评审。

证据：source-reviewed

输入

- 论文 PDF 路径。
- LLM client 和 review 模型。
- Review form，默认是 NeurIPS 风格。

输出

- review.txt
- 可选 reviewimproved.txt
- 可选 improved PDF。

人工检查

- review 是否指出真实实验缺陷。
- score 是否和 weaknesses 一致。
- 改进后是否引入新主张或虚假结果。

### 3. 容器化沙箱入口

以 experimental/Dockerfile 作为全仓唯一的容器化入口，把 AI-Scientist 运行时、所有版本 pin 的依赖、NPEET commit 锁定、texlive-full（LaTeX 编译）、aider-chat（coder 主体）和全模板 baseline 预跑封装成可复现镜像，消除人工准备 run0/finalinfo.json baseline 的痛点，并经构建期生成的 entrypoint.sh 把容器启动参数透传给 launchscientist.py。

证据：source-reviewed

输入

- 基础镜像 python:3.11-bullseye。
- apt 包（全版本 pin）：wget/git/build-essential/libssl-dev/zlib1g-dev/texlive-full。
- pip 包（全版本 pin）：pip/anthropic/aider-chat/backoff/openai/matplotlib/pypdf/pymupdf4llm/torch/numpy/transformers/datasets/tiktoken/wandb/tqdm/scikit-learn/einops。

输出

- 可复现的 ai-scientist Docker 镜像。
- 全模板的 run0/finalinfo.json baseline（消除人工准备痛点）。
- NanoGPT 数据集（预下载）。

人工检查

- 运维确认 docker build 在目标环境的成功率（texlive-full 较大）。
- 研究者确认 NPEET commit pin 是否符合互信息估计需求。
- 开发者确认是否需要补一个转发 launchoescientist.py 的 entrypoint 变体。

### 4. 开放式Idea增量生成与Score回写闭环

以 experimental/launchoescientist.py 为第二条主流程入口，相对 launchscientist.py 的批量生成，改用 generatenextidea 单步增量生成，把每轮 doidea 的评审 Score 回写到共享 ideaarchive，作为下一轮生成的 prompt 上下文，形成"生成→新颖性筛→执行打分→Score 回写→下一轮生成"的开放式自迭代闭环。

证据：source-reviewed

输入

- basedir（模板目录，含 experiment.py/plot.py/prompt.json/seedideas.json）。
- client/model（LLM 客户端与模型名）。
- 共享 ideaarchive（列表，跨 worker 累积，初始为空）。

输出

- 持续增长的 ideaarchive（含带 Score 的历史 idea）。
- 每轮的实验产物（runi/）、论文草稿（.tex）、评审（review.txt/reviewimproved.txt）。
- 最终的 ideas.json（含所有 idea 及其 Score）。

人工检查

- 研究者确认 numideas 和 numreflections 是否符合开放式探索需求。
- 研究者确认 Score 反馈是否真的引导 LLM 向高质量方向演化（而非模式坍缩）。
- 运维确认 parallel 和 sleep 150 在目标 API 限流下的稳定性。

### 5. 模板复制与多GPU论文流水调度

把已标记为新颖的 idea 和一个模板目录复制成独立结果目录，按实验、写作、评审和可选改进的顺序调度多个节点，并在并行模式下把 idea 分发到 GPU worker。

证据：source-reviewed

输入

- 模板目录：templates/<experiment/
- 通过新颖性筛查的 idea。
- templates/<experiment/run0/finalinfo.json，由用户提前跑 baseline 生成。

输出

- results/<experiment/<timestamp<ideaname/
- notes.txt
- log.txt

人工检查

- 是否允许执行 LLM 生成代码。
- 是否已经为每个模板准备 run0 baseline。
- 是否允许外部 API、网络和文件写入。

### 6. 研究想法生成与新颖性筛查

把模板目录里的 prompt.json、experiment.py、seedideas.json 和已有 ideas.json 变成一组结构化研究想法，再用 Semantic Scholar 或 OpenAlex 检索相关论文，让 LLM 判断每个想法是否足够新颖，最后把 novel 状态写回 ideas.json。

证据：source-reviewed

输入

- 模板目录：templates/<experiment/
- prompt.json：至少包含 system 和 taskdescription。
- experiment.py：作为想法生成和新颖性判断的任务背景。

输出

- templates/<experiment/ideas.json
- 带 novel 字段的 idea 列表。
- 后续 launchscientist.py 中的 novelideas 队列。

人工检查

- idea 是否适合当前模板的可执行实验。
- 检索 query 是否覆盖核心同义词和已知相关工作。
- 检索空结果是否说明新颖，还是说明 query 质量不足。

## 处理机制

1. **多模型API与运行安全边界**：这个机制解释 AI-Scientist 的模型 provider、API key、外部检索、GPU、LaTeX、许可证和 LLM 写代码风险如何共同限制各能力的可运行范围。
2. **模板产物状态链路**：这个机制解释 AI-Scientist 如何把模板目录、idea、实验运行、论文写作和 review 串成一条文件状态链。它跨越 idea 生成、模板调度、实验执行、论文写作和评审改进五张能力卡。

## 开始方式

1. 生成想法：基于模板和种子想法生成候选研究想法，用文献检索筛选新颖性
2. 改写实验：让LLM根据选中想法修改模板实验代码，运行并收集实验结果
3. 撰写论文：自动撰写各章节论文，搜索补全参考文献并插入BibTeX条目
4. 编译输出：完成LaTeX编译和格式检查，生成可投稿的PDF文件
5. 评审改进：用LLM模拟会议评审，可选根据反馈改进论文并重新编译

## 环境要求

- 准备模板实验目录，包含可运行的experiment.py和plot.py
- 手动运行一次基线实验，生成baseline对比结果文件
- 配置LLM API Key和Semantic Scholar或OpenAlex访问权限
- 确保Linux环境、CUDA GPU和pdflatex等LaTeX工具链就绪

## 关键限制

- 把LLM自动评审当作正式同行评审的替代品，忽略实验缺陷
- 不准备基线运行结果就启动流水线，实验缺乏对比基准
- 让系统在没有沙箱限制的环境中执行LLM生成代码，存在安全风险
- 忽略AI Scientist许可协议中的披露要求，发表时不声明AI参与

## 已核对范围

- 读取 performexperiments.py 的 coder prompt、runi 执行、stderr 截断、plot loop 和 notes 写入
- 读取 launchscientist.py 中 Aider Coder.create 与 performexperiments 调用
- 读取 README 的单篇 review 与 batch analysis 示例
- 读取 performreview.py 的 PDF 文本抽取、few-shot review、ensemble、meta-review、reflection 和 improvement
- 读取 launchscientist.py 的 review.txt 与 improved review 写入路径
- 静态读取 experimental/Dockerfile 全量（基础镜像/apt pin/pip pin/NPEET commit pin/全模板 baseline 预跑/entrypoint 构建期生成）
- 静态读取 generateideas.py generatenextidea/checkideanovelty（生效版第二份定义）、experimental/launchoescientist.py worker/顺序模式/doidea/Score 回写
- 读取 README 的模板准备、主运行命令和并行说明

## 仍待核对

- 未运行 Aider
- 未执行任何模板 experiment.py
- 未验证 runi/finalinfo.json 结构
- 未验证 plot.py 产物
- 未解析真实 PDF
- 未调用 GPT-4o review
- 未运行 ICLR batch analysis
- 未验证评分与真实评审一致性

## 许可边界

本页的能力分析、结构与说明由 bo老师整理，采用 CC BY 4.0。上游项目、代码、名称和商标继续遵循各自许可。这里不重新发布上游代码。
