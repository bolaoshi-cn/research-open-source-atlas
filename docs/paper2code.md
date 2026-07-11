# Paper2Code

把机器学习论文自动转为复现计划、代码文件和评估报告

适合需要复现机器学习论文结果的研究者和工程师。输入论文 PDF 或 LaTeX 源文件，工具先生成复现计划与系统架构，再逐文件生成代码、运行脚本和调试修补，最后用模型评估生成仓库的复现正确性。

- 官网详情：https://bolaoshi.cn/research-tools/paper2code
- 原始来源：https://github.com/going-doer/Paper2Code
- 读取版本：ba9169978043d5799c8d4f4a0963e6b66a24c2e1
- 核对日期：2026-06-16
- GitHub Stars：4,795（2026-07-11）
- 上游许可：Apache-2.0
- 验证程度：catalogued
- 编辑判断：worth-watching

## 什么时候使用

- 拿到一篇机器学习论文后需要快速搭建复现代码骨架
- 想验证论文方法是否可以在不手工阅读全部细节的情况下被代码化
- 需要对比论文声称结果与模型生成代码的运行结果
- 论文复现工作量很大，需要先获得结构化的任务列表和依赖配置

## 先准备什么

- 论文 PDF 文件或 LaTeX 源文件
- OpenAI API 密钥或可访问的 vLLM 服务
- 目标输出目录和 Python 运行环境
- 论文中的公式、图表和算法描述

## 它会怎样推进

1. **论文材料清洗**：将 PDF 转为 JSON 并清洗，或直接读取 LaTeX 正文，提取结构化论文内容
2. **生成复现计划**：从论文内容生成复现计划、系统架构设计、文件级任务列表和依赖配置 YAML
3. **逐文件逻辑分析**：按任务列表顺序分析每个文件的输入输出、数据结构、算法步骤和模块间依赖
4. **逐文件代码生成**：根据分析结果生成代码文件，已完成文件作为后续文件的上下文
5. **运行调试与评估**：生成运行脚本并尝试执行，根据错误修补代码，最后用模型评估复现正确性

## 第一次这样开始

帮我安装这个库：https://github.com/going-doer/Paper2Code
安装前先告诉我需要什么环境，安装后帮我跑通一个最小示例。

## 结果出来后先看什么

- 生成的任务列表是否能稳定覆盖论文的关键实验
- 配置 YAML 中的依赖是否与实际生成代码一致
- 逐文件生成是否保持正确的 import 依赖顺序
- 模型评估给出的分数和理由是否能对应到具体代码行

## 常见误区

- 把生成的代码当作可直接运行的最终产物——模型可能忽略论文中未明确描述的细节或超参数
- 跳过计划审查直接执行——如果复现计划遗漏了关键模块，后续代码链条会整体偏离
- 依赖模型评估来判断复现正确性——生成代码和论文结果之间的差异仍需人工对照原文验证

## 能力拆解

### 1. 复现计划与任务设计生成

把论文内容转成一组面向代码复现的计划对象：总体复现计划、系统架构设计、文件任务拆分和配置 YAML。这个能力是 PaperCoder 后续分析和代码生成的中心合同。

证据：source-reviewed

输入

- JSON 或 LaTeX 论文内容。
- papername、gptversion 或 modelname。
- outputdir 作为计划响应和轨迹保存位置。

输出

- planningresponse.json
- planningtrajectories.json
- 可被 1.1extractconfig.py、2analyzing.py、3coding.py 读取的计划上下文。

人工检查

- 需要人工判断计划是否覆盖论文核心方法、数据、训练和评价。
- 需要人工决定 OpenAI 或 vLLM 路径。
- 高成本模型调用前需要确认预算和 API key。

### 2. 模型评估裁决与评分汇总

把论文、生成仓库代码和可选 gold 仓库代码输入评估模型，要求模型按核心方法正确性给出 critique list 和 1 到 5 分，多次采样后汇总平均分和有效输出数量。

证据：source-reviewed

输入

- 论文 JSON。
- 目标仓库目录。
- 可选 gold 仓库目录。

输出

- results/<papereval<evaltype<model<time.json
- 控制台评估摘要。
- outputdir/costinfo.log 中的成本记录。

人工检查

- 需要人工决定 ref-free 或 ref-based。
- 需要人工确认 gold repo 是否是合格参照。
- 需要人工审查 critique list 中的位置和严重性。

### 3. 论文材料清洗与双输入摄取

把论文材料摄取为 PaperCoder 后续可消费的输入对象。仓库支持 PDF 转 JSON 后清洗，也支持直接读取已清理的 LaTeX 文本，运行脚本用环境变量和参数把输入路径传给 planning、analysis 和 coding 阶段。

证据：source-reviewed

输入

- S2ORC 风格的论文 JSON，例如 examples/Transformer.json。
- 已清洗 JSON，例如 examples/Transformercleaned.json。
- 已清洗 LaTeX 文件，例如 examples/Transformercleaned.tex。

输出

- cleaned JSON 文件。
- LaTeX 文本输入分支。
- 传给 planning、analysis 和 coding 阶段的论文材料对象。

人工检查

- 用户需要确认使用 JSON 路径、LaTeX 路径或外部 PDF 转换路径。
- 自有论文进入前需要确认版权、解析质量和是否允许发送给模型服务。
- 若清洗删除字段影响复现，应人工保留或补充字段。

### 4. 运行脚本生成与调试修补

在代码文件落盘后，为生成仓库补一个可运行脚本，并在执行出错后把错误日志、生成代码和配置交给模型，由模型输出 SEARCH/REPLACE 补丁，再由脚本应用到目标文件。

证据：source-reviewed

输入

- 已生成的输出仓库。
- planningconfig.yaml 和输出仓库中的 config.yaml。
- task list 中的 Python 文件。

输出

- outputrepodir/reproduce.sh
- 被修补的 Python 文件。
- 备份文件 <path.<savenum.bak

人工检查

- 需要人工提供错误日志。
- 需要人工检查模型补丁是否安全。
- 需要人工运行 reproduce.sh 和相关测试。

### 5. 逐文件代码生成与仓库落盘

根据论文、计划、设计、任务、配置和逐文件 analysis，按 task list 顺序生成代码文件，并写入目标输出仓库。已完成文件会作为上下文传给后续文件，使多文件生成形成简单状态记忆。

证据：source-reviewed

输入

- 论文 JSON 或 LaTeX 文本。
- planningconfig.yaml。
- planningtrajectories.json。

输出

- outputrepodir 下的代码文件。
- codingartifacts/<filecoding.txt
- 更新后的 accumulatedcost.json

人工检查

- 需要人工检查 task list 是否安全且路径合理。
- 需要人工运行生成仓库测试。
- 需要人工判断代码是否忠于论文方法。

### 6. 逐文件逻辑分析生成

在真正写代码前，为 task list 中每个目标文件生成一份更细的逻辑分析。该分析绑定论文内容、总体计划、系统设计、任务拆分和配置 YAML，用于降低逐文件代码生成时遗漏依赖和偏离论文的风险。

证据：source-reviewed

输入

- 论文 JSON 或 LaTeX 文本。
- planningconfig.yaml。
- planningtrajectories.json 中前三个 assistant 输出。

输出

- analyzingartifacts/<filesimpleanalysis.txt
- <filesimpleanalysisresponse.json
- <filesimpleanalysistrajectories.json

人工检查

- 需要人工确认 task list 是否合理。
- 需要人工检查每个 analysis 是否引用正确论文细节和配置。
- 若 output 解析失败，需要人工修复 planning 输出或提供 tasklist.json。

## 处理机制

1. **三阶段轨迹与产物目录链**：Paper2Code 的多张能力卡共享同一条产物链：输入论文先进入 planning，planning 产出 trajectories、config 和 task list，analysis 按 task list 生成逐文件分析，coding 再按相同顺序写入输出仓库，eval 和 debug 继续复用同一组文件。这个机制卡说明这些对象如何跨阶段传递。

## 开始方式

1. 论文材料清洗：将 PDF 转为 JSON 并清洗，或直接读取 LaTeX 正文，提取结构化论文内容
2. 生成复现计划：从论文内容生成复现计划、系统架构设计、文件级任务列表和依赖配置 YAML
3. 逐文件逻辑分析：按任务列表顺序分析每个文件的输入输出、数据结构、算法步骤和模块间依赖
4. 逐文件代码生成：根据分析结果生成代码文件，已完成文件作为后续文件的上下文
5. 运行调试与评估：生成运行脚本并尝试执行，根据错误修补代码，最后用模型评估复现正确性

## 环境要求

- 论文 PDF 文件或 LaTeX 源文件
- OpenAI API 密钥或可访问的 vLLM 服务
- 目标输出目录和 Python 运行环境
- 论文中的公式、图表和算法描述

## 关键限制

- 把生成的代码当作可直接运行的最终产物——模型可能忽略论文中未明确描述的细节或超参数
- 跳过计划审查直接执行——如果复现计划遗漏了关键模块，后续代码链条会整体偏离
- 依赖模型评估来判断复现正确性——生成代码和论文结果之间的差异仍需人工对照原文验证

## 已核对范围

- 静态确认 OpenAI 和 vLLM planning 脚本的四轮消息链
- 静态确认 plan、architecture design、task list 和 config YAML 的输出顺序
- 静态确认 ref-free 和 ref-based 两种评估 prompt
- 静态确认 eval 脚本读取论文、目标仓库、可选 gold 仓库并汇总 1 到 5 分
- 静态确认 JSON 清洗脚本、LaTeX 输入分支和四个运行脚本的入口参数
- 静态确认示例 JSON 结构和 cleaned JSON 的字段变化
- 静态确认 reproduce.sh 生成脚本读取生成仓库代码和配置
- 静态确认 debug 脚本按 SEARCH/REPLACE block 应用模型修补

## 仍待核对

- 未调用 OpenAI 或 vLLM 生成真实计划
- 未检查输出 JSON 是否总能被解析
- 未验证计划能否覆盖任意机器学习论文的关键实验
- 未调用评估模型
- 未验证评分稳定性
- 未验证无有效模型输出时的失败处理
- 未运行 s2orc-doc2json 或 Grobid 服务
- 未用真实 PDF 生成新的 JSON

## 许可边界

本页的能力分析、结构与说明由 bo老师整理，采用 CC BY 4.0。上游项目、代码、名称和商标继续遵循各自许可。这里不重新发布上游代码。
