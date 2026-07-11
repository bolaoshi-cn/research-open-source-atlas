# Claude Scientific Writer

科研写作工具包，覆盖论文、基金、临床报告和视觉产物的生成

以Claude Code插件和Python API形式提供19个科研写作skill。适合需要从论文、基金申请到海报、幻灯片一站式生成的研究者，也覆盖临床报告和同行评审场景。

- 官网详情：https://bolaoshi.cn/research-tools/claude-scientific-writer
- 原始来源：https://github.com/K-Dense-AI/claude-scientific-writer
- 读取版本：2d323a17101fcd6458eea591867495a3aa05e908
- 核对日期：2026-06-16
- GitHub Stars：2,083（2026-07-11）
- 上游许可：MIT
- 验证程度：partially-verified
- 编辑判断：worth-watching

## 什么时候使用

- 需要从研究主题和数据一键生成论文草稿
- 准备基金申请书并需要按NSF/NIH结构组织
- 想把论文转成学术海报或会议幻灯片
- 需要一个集成文献检索、引用管理和格式化的写作环境

## 先准备什么

- 准备好研究主题、数据文件和相关论文素材
- 配置Anthropic API密钥用于Claude Agent调用
- 如需文献检索需配置research-lookup后端

## 它会怎样推进

1. **初始化项目**：安装插件，运行初始化命令，把科研写作规范注入项目配置文件
2. **输入材料**：提供研究主题、数据文件、图片和已有草稿，工具按类型分到对应目录
3. **检索文献**：按关键词搜索文献，提取DOI和元数据，生成BibTeX并校验引用字段
4. **生成草稿**：按IMRAD结构生成论文，或按基金模板生成申请书的各部分内容
5. **输出产物**：草稿、终稿、引用、图表分别写入标准化输出目录

## 第一次这样开始

帮我安装这个库：https://github.com/K-Dense-AI/claude-scientific-writer
安装前先告诉我需要什么环境，安装后帮我跑通一个最小示例。

## 结果出来后先看什么

- 引用是否真实存在且支持正文中的主张
- 输出目录的drafts/final/references/figures是否完整
- 基金申请书的预算和团队信息是否来自真实数据
- 临床报告是否已确认无个人健康信息泄露

## 常见误区

- 直接使用CLI模式时未注意它会删除data目录中的原始文件
- 把AI生成的引用当作已验证而不逐条核对其真实性
- 在临床场景下跳过人工复核直接把AI输出用于诊疗决策

## 能力拆解

### 1. CLI与API调度写作产物生成

通过 Python API 或 scientific-writer CLI 收集研究主题、模型、输出目录、API key 和数据文件，把 .claude 写作上下文复制到工作目录，再调用 Claude Agent SDK 生成论文或报告产物，并扫描 writingoutputs 得到结果对象。

证据：source-reviewed

输入

- query
- outputdir
- apikey 或 ANTHROPICAPIKEY

输出

- writingoutputs/<timestamp<topic/
- drafts/
- final/

人工检查

- 是否允许 CLI 删除原始 data 文件。
- 是否允许以 bypass 权限运行 Claude Agent。
- 是否接受当前 model 映射和 Stop hook 继续策略。

### 2. 临床科研文档与决策支持

把病例、试验、队列、诊断、治疗方案和临床证据整理为临床报告、医学论文、决策支持文档或治疗计划，并用报告指南、证据分级和隐私约束约束输出。

证据：source-reviewed

输入

- 病例资料、试验结果、队列数据或诊断信息。
- 临床问题、患者群体、干预和结局。
- 报告指南：CARE、CONSORT、STROBE、PRISMA 或 GRADE。

输出

- 临床病例报告。
- 临床试验或队列分析报告。
- 治疗建议报告。

人工检查

- 是否含个人健康信息。
- 输出是否用于真实临床决策。
- 指南和证据是否为最新版。

### 3. 基金申请与投稿格式装配

围绕资助机构、投稿场景或学术 venue 要求，把研究目标、创新点、方法、预算、影响和格式规范装配为基金申请书、投稿材料或符合 venue 模板的研究产物。

证据：source-reviewed

输入

- 资助机构或项目类型。
- 研究目标、背景、方法、团队和预算。
- specific aims、broader impacts、timeline、deliverables。

输出

- specific aims。
- project summary。
- research strategy。

人工检查

- 资助机构和申请类型是否明确。
- 预算、团队和时间线是否真实。
- 模板是否为当前周期最新版。

### 4. 插件初始化与skill分发

通过 Claude Code marketplace 插件安装、初始化命令、模板 CLAUDE.md 和 skills/ 目录，把科研写作相关能力分发到用户项目，并让 Claude Code 在后续会话中按任务自动加载相应 skill。

证据：source-reviewed

输入

- Claude Code plugin marketplace URL。
- marketplace.json 中的 skill path 列表。
- 初始化命令。

输出

- 目标项目的 CLAUDE.md。
- 被 Claude Code 可发现的 skill 描述。
- 初始化时可能产生的备份或合并版本。

人工检查

- 是否覆盖、备份或合并现有 CLAUDE.md。
- 是否接受 marketplace 只分发 19 个 skill 的范围。
- README 命令和 command 文件命令哪一个是当前真实入口。

### 5. 文档转换与办公文件处理

把 PDF、DOCX、PPTX、XLSX、图片、音频、HTML、CSV、JSON、XML、ZIP、EPUB 和网页等材料转换为可读 Markdown 或可编辑办公文件，并为科研写作流程提供来源材料、表格、图像、草稿和最终产物处理能力。

证据：source-reviewed

输入

- PDF、DOCX、PPTX、XLSX、CSV、JSON、XML、HTML、图片、音频、ZIP、EPUB 或网页 URL。
- 目标输出格式。
- MarkItDown、pandoc、qpdf、LibreOffice 等本地工具。

输出

- Markdown 来源文件。
- 编辑后的 DOCX、PDF、PPTX 或 XLSX。
- 提取的表格、图片或文字。

人工检查

- 是否允许修改原始办公文件。
- 是否需要保留原文件。
- license 是否允许当前使用场景。

### 6. 文献检索与引用治理

围绕研究主题执行文献检索、综述构建、引用管理和 citation validation，把外部搜索结果、DOI、PMID、arXiv 编号、BibTeX 条目和引用检查结果组织为可追溯的写作来源。

证据：source-reviewed

输入

- 研究问题和关键词。
- 文献数据库范围。
- DOI、PMID、arXiv ID、URL 或论文题名。

输出

- 文献综述草稿。
- 检索结果和 sources 文件。
- BibTeX 文件。

人工检查

- 检索范围是否覆盖目标领域。
- 引用是否真实、可访问且支持对应句子。
- 是否允许调用外部搜索后端。

## 处理机制

1. **核心处理链**：科研写作工具包，覆盖论文、基金、临床报告和视觉产物的生成

## 开始方式

1. 初始化项目：安装插件，运行初始化命令，把科研写作规范注入项目配置文件
2. 输入材料：提供研究主题、数据文件、图片和已有草稿，工具按类型分到对应目录
3. 检索文献：按关键词搜索文献，提取DOI和元数据，生成BibTeX并校验引用字段
4. 生成草稿：按IMRAD结构生成论文，或按基金模板生成申请书的各部分内容
5. 输出产物：草稿、终稿、引用、图表分别写入标准化输出目录

## 环境要求

- 准备好研究主题、数据文件和相关论文素材
- 配置Anthropic API密钥用于Claude Agent调用
- 如需文献检索需配置research-lookup后端

## 关键限制

- 直接使用CLI模式时未注意它会删除data目录中的原始文件
- 把AI生成的引用当作已验证而不逐条核对其真实性
- 在临床场景下跳过人工复核直接把AI输出用于诊疗决策

## 已核对范围

- 读取 Python API generatepaper 的参数、环境变量、Claude Agent SDK 调用和输出扫描
- 读取 CLI 主流程、交互入口、已有论文检测和 data 文件处理分支
- 读取 core.py 对 .claude 目录复制、输出目录和文件路由的实现
- 读取 clinical-reports skill 的临床报告类型和合规约束
- 读取 clinical-decision-support skill 的队列分析、治疗建议和证据分级流程
- 读取 docs/SKILLS 中 clinical 和 treatment plan 相关能力概览
- 读取 research-grants skill 的申请书结构、机构类型和审查维度
- 读取 docs/SKILLS 中 venue-templates 与科研申请相关 skill 概览

## 仍待核对

- 未运行 generatepaper
- 未调用 Claude Agent SDK
- 未验证真实 token 统计、Stop hook 或输出质量
- 未在 data 目录放入真实文件验证删除行为
- 未处理真实临床数据
- 未生成病例报告、临床试验报告或治疗建议
- 未验证 HIPAA、CARE、CONSORT、GRADE 或监管合规
- 未生成基金申请书

## 许可边界

本页的能力分析、结构与说明由 bo老师整理，采用 CC BY 4.0。上游项目、代码、名称和商标继续遵循各自许可。这里不重新发布上游代码。
