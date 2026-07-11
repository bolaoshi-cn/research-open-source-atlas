# llm-for-zotero

Zotero插件，让LLM在论文库中阅读、检索、对比和写笔记

适合用Zotero管理文献库并希望AI直接操作馆藏的研究者。安装后可以用自然语言搜索论文、阅读PDF问答、比较多篇文献、生成带引用的笔记，并保存回Zotero或本地Markdown文件。

- 官网详情：https://bolaoshi.cn/research-tools/llm-for-zotero
- 原始来源：https://github.com/yilewang/llm-for-zotero
- 读取版本：source-reviewed
- 核对日期：2026-06-16
- GitHub Stars：2,227（2026-07-11）
- 上游许可：AGPL-3.0
- 验证程度：catalogued
- 编辑判断：worth-watching

## 什么时候使用

- 想用自然语言搜索Zotero馆藏中与某问题相关的论文
- 需要阅读论文时向AI提问，并让回答中的引用能跳转回原文位置
- 需要同时比较多篇论文的方法、结论和数据
- 希望AI帮助撰写文献笔记并直接写入Zotero或Obsidian等本地笔记

## 先准备什么

- 安装Zotero插件并配置LLM API密钥或本地模型端点
- 确保Zotero馆藏中有足够的论文和附件（PDF全文优先）
- 如果需要笔记写入，设置好本地Markdown目录或Obsidian vault路径
- 如果需要MinerU解析增强，配置MinerU服务地址

## 它会怎样推进

1. **打开论文**：在Zotero中打开一篇或多篇论文PDF
2. **提问**：用自然语言向AI提问论文内容、方法或结论
3. **查看证据**：AI返回带引用标记的回答，点击引用跳转到原文对应位置
4. **对比多篇**：选择多篇论文让AI做横向对比，生成比较矩阵或综述
5. **保存笔记**：将AI生成的回答或笔记保存为Zotero note或本地Markdown文件

## 第一次这样开始

帮我安装这个库：https://github.com/yilewang/llm-for-zotero
安装前先告诉我需要什么环境，安装后帮我跑通一个最小示例。

## 结果出来后先看什么

- 检查回答中的引用是否跳转到正确的论文章节和页码
- 验证AI生成的论文摘要和结论是否与原文一致
- 确认笔记写入后内容完整、格式正确、引用可追溯

## 常见误区

- 把AI生成的论文解读当作原文原意而不回原文验证
- 在多论文对比中不确认论文范围是否完整覆盖了相关文献
- 将AI写入的笔记当作最终产物而不人工审校和补充

## 能力拆解

### 1. Agent写操作确认与撤销

Agent 能修改标签、collection、元数据、导入文件、删除或合并条目。系统怎样在写入前设置人审门，并在写入后保留有限撤销能力。

证据：source-reviewed

输入

- 与Agent写操作确认与撤销相关的任务材料和配置

输出

- Agent写操作确认与撤销的结构化结果

人工检查

- 核对结果是否符合当前研究问题和证据边界

### 2. Agent文库检索与证据检索

Agent Mode 怎样把 Zotero 馆藏搜索、论文读取、广域证据检索和在线学术搜索暴露成工具，使模型能先查库再回答。

证据：source-reviewed

输入

- 与Agent文库检索与证据检索相关的任务材料和配置

输出

- Agent文库检索与证据检索的结构化结果

人工检查

- 核对结果是否符合当前研究问题和证据边界

### 3. Claude-Code桥接与独立会话

插件怎样把 Claude Code 作为独立会话系统嵌入 Zotero，并通过 companion bridge 与 scoped Zotero 上下文交互。

证据：source-reviewed

输入

- 与Claude-Code桥接与独立会话相关的任务材料和配置

输出

- Claude-Code桥接与独立会话的结构化结果

人工检查

- 核对结果是否符合当前研究问题和证据边界

### 4. Codex-App-Server与Zotero-MCP桥接

插件怎样让 Codex App Server 在 Zotero 对话中使用 scoped Zotero MCP 工具，同时限制内置危险 approval。

证据：source-reviewed

输入

- 与Codex-App-Server与Zotero-MCP桥接相关的任务材料和配置

输出

- Codex-App-Server与Zotero-MCP桥接的结构化结果

人工检查

- 核对结果是否符合当前研究问题和证据边界

### 5. MinerU解析缓存与同步包治理

系统怎样用 MinerU 把 PDF 解析为高质量 Markdown、图片和 manifest，并通过本地缓存与 Zotero file sync 包复用。

证据：source-reviewed

输入

- 与MinerU解析缓存与同步包治理相关的任务材料和配置

输出

- MinerU解析缓存与同步包治理的结构化结果

人工检查

- 核对结果是否符合当前研究问题和证据边界

### 6. WebChat浏览器转发与历史同步

系统怎样通过浏览器扩展把 Zotero 中的问题转发到 ChatGPT 或 DeepSeek 网页，并把响应和历史同步回插件。

证据：source-reviewed

输入

- 与WebChat浏览器转发与历史同步相关的任务材料和配置

输出

- WebChat浏览器转发与历史同步的结构化结果

人工检查

- 核对结果是否符合当前研究问题和证据边界

## 处理机制

1. **工具面确认缓存与证据账本**：该项目的关键横向机制是：让 agent 能读很多资料，也能执行部分写动作，同时把读过什么、是否需要确认、确认后如何撤销这几件事放在同一层治理。

## 开始方式

1. 打开论文：在Zotero中打开一篇或多篇论文PDF
2. 提问：用自然语言向AI提问论文内容、方法或结论
3. 查看证据：AI返回带引用标记的回答，点击引用跳转到原文对应位置
4. 对比多篇：选择多篇论文让AI做横向对比，生成比较矩阵或综述
5. 保存笔记：将AI生成的回答或笔记保存为Zotero note或本地Markdown文件

## 环境要求

- 安装Zotero插件并配置LLM API密钥或本地模型端点
- 确保Zotero馆藏中有足够的论文和附件（PDF全文优先）
- 如果需要笔记写入，设置好本地Markdown目录或Obsidian vault路径
- 如果需要MinerU解析增强，配置MinerU服务地址

## 关键限制

- 把AI生成的论文解读当作原文原意而不回原文验证
- 在多论文对比中不确认论文范围是否完整覆盖了相关文献
- 将AI写入的笔记当作最终产物而不人工审校和补充

## 已核对范围

- 工具确认层
- library mutation service
- undo store
- undolastaction 工具
- Agent tool 注册
- libraryretrieve 服务
- paper evidence 检索
- literaturesearch 暴露口径

## 仍待核对

- Zotero 写入真实事务
- 多轮撤销冲突
- review card UI
- Zotero FTS 实机结果
- embedding 后端可用性
- CrossRef 与 Semantic Scholar 实时结果质量
- companion bridge repo
- Claude CLI 实机

## 许可边界

本页的能力分析、结构与说明由 bo老师整理，采用 CC BY 4.0。上游项目、代码、名称和商标继续遵循各自许可。这里不重新发布上游代码。
