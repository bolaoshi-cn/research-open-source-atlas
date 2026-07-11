# Nature Skills

覆盖论文精读、写作、润色、审稿回应与数据治理全流程的科研 Skill 集合

适合需要系统性提升论文表达质量的研究者，覆盖从文献检索、全文阅读、论文写作、图表生成到审稿回应与数据可用性检查的完整链路。使用时按任务选择对应 skill，先让工具诊断结构问题再输出文本或文件。

- 官网详情：https://bolaoshi.cn/research-tools/nature-skills
- 原始来源：https://github.com/Yuan1z0825/nature-skills
- 读取版本：e906070
- 核对日期：2026-06-14
- GitHub Stars：27,794（2026-07-11）
- 上游许可：MIT
- 验证程度：partially-verified
- 编辑判断：worth-watching

## 什么时候使用

- 需要从零起草 Nature 风格论文段落或全文
- 投稿前想做多视角预审稿发现论证和技术风险
- 收到审稿意见后需要逐条映射回应策略和修改清单
- 需要为论文检索 Nature/CNS 候选引用文献，或转为中文学术 PPTX 与发表级图表

## 先准备什么

- 论文正文、图表、数据或中文写作草稿
- 目标期刊名称与论文类型
- 审稿意见原文与修改说明（审稿回应场景）
- 需要支撑引用的论文段落或主张列表
- Python 或 R 运行环境（图表生成场景）

## 它会怎样推进

1. **选择技能入口**：根据当前任务（写作、润色、审稿、引用、图表、数据声明等）选择对应的 nature-* skill
2. **提交材料并接收诊断**：提交论文素材，工具先诊断结构、证据或数据缺口，而非直接输出成品
3. **生成结构化中间对象**：将论文转为 source map、论证书脊、评论追踪器或数据集清单等可检查对象
4. **输出草稿并标注风险**：产生段落、图表、回应信或 PPTX 草稿，同时标注缺失信息和不确定性
5. **逐项复核与迭代**：研究者逐一确认人工判断项（证据强度、数据可用性、回应可执行性），修正后交付

## 第一次这样开始

帮我安装这个库：https://github.com/Yuan1z0825/nature-skills
安装前先告诉我需要什么环境，安装后帮我跑通一个最小示例。

## 结果出来后先看什么

- 输出的段落是否标注了缺失证据和不确定范围
- 审稿回应 package 的 readiness 状态是否准确（draft/needs_input/blocked）
- 图表是否符合目标期刊尺寸并保留可编辑文本
- 引用候选是否只是 metadata-only，未与真实支持混同
- Data Availability 声明是否发明了不存在的 DOI 或仓库标识符

## 常见误区

- 提交太薄的材料却期待完整成品——缺少 claim/evidence/boundary 时工具会返回缺口清单而非正文
- 把 metadata-only 引用候选当成真实引文支持，跳过人工核查 abstract 或出版商全文
- 跳过润色前的结构诊断直接做句子级语言修改，掩盖了段落逻辑错误
- 把预审稿三评审报告当作真实同行评审结论或录用概率判断

## 能力拆解

### 1. skill路由与片段加载

把用户的科研任务请求，路由到某个 nature- skill，并按当前任务只加载必要的核心片段、轴片段和深度引用。

证据：source-reviewed

输入

- 用户任务文本。
- 已安装或已克隆的 skills/ 目录。
- 某个 skill 的 SKILL.md 与 manifest.yaml。

输出

- 命中 skill 的可执行上下文。
- 轴值或任务模式声明。
- 后续 skill 产物，例如 paper.md、引用文件、Data Availability、PPTX、图表脚本或回应包。

人工检查

- 哪些 skill 适合进入本地系统。
- MCP 是否需要改成常驻 HTTP/SSE。
- shared 是否要拆成用户级公共规范，还是只作为该来源事实保留。

### 2. 主张分段引用检索与导出

把论文段落或独立主张拆成可引用片段，为每个片段检索 Nature/CNS 家族候选文献，并导出参考文献管理文件和筛选材料。

证据：runtime-verified

输入

- Manuscript text、claim 文本、claim 文件或 DOI 列表。
- 引用范围：nature、science、cell、cns、flagship。
- 日期限制、每段候选数、输出格式和 artifact 开关。

输出

- segment to references mapping。
- references.enw、references.ris 或 references.rdf。
- 可选 HTML、TSV、JSON、Markdown report，文件 stem 跟随输出文件名或默认 base。

人工检查

- 核对结果是否符合当前研究问题和证据边界

### 3. 全文阅读来源锚点构建

把论文 PDF、HTML、DOI、arXiv 或粘贴文本，转成带稳定来源锚点、双语段落、图表卡和 source map 的全文阅读包。

证据：source-reviewed

输入

- 可选择文本层 PDF、扫描 PDF、HTML、DOI、arXiv 链接或粘贴文本。
- 论文元数据、正文、图、表、图注、页码。
- 可选 PDF/OCR 工具。

输出

- paper.md 全文双语阅读包。
- sourcemap.json 稳定锚点。
- translationnotes.md 术语、不确定和布局说明。

人工检查

- 核对结果是否符合当前研究问题和证据边界

### 4. 图表契约到发布图形

把论文图表请求、数据和目标期刊要求，先转成 figure contract，再用选定 Python 或 R 后端生成可编辑、可审查的发布级科学图形。

证据：source-reviewed

输入

- 目标 claim、数据、图表类型、期刊或导出要求。
- 用户明确选择的 backend，或要求推荐 backend。
- 统计信息、source data、图像完整性信息。

输出

- figure contract。
- Python 或 R 绘图脚本。
- SVG、PDF、TIFF/PNG。

人工检查

- 核对结果是否符合当前研究问题和证据边界

### 5. 多源学术检索与引用文件管理

把用户的文献检索、引文核对、MeSH 策略或引用文件管理请求，转成多源检索、去重、元数据获取、引用格式生成和文件转换流程。

证据：runtime-verified

输入

- 主题 query、作者、ORCID、DOI、PMID、arXiv ID。
- 检索目标：多源搜索、引用核验、MeSH 策略、文件转换、参考文献管理。
- 可选 API key、mailto、NCBI key、pybliometrics 配置。

输出

- 统一论文结果表。
- DOI、PMID、arXiv 元数据。
- 格式化 citation。

人工检查

- 核对结果是否符合当前研究问题和证据边界

### 6. 审稿回应逐点映射

把 editor decision、reviewer comments、作者修改说明或回应草稿，转成逐点回应策略、comment tracker、回应信、修改清单和 readiness state。

证据：source-reviewed

输入

- editor decision letter。
- reviewer comments。
- author actions、manuscript change notes、line/page、figure/table/supplement locations。

输出

- Response strategy summary。
- Comment-response tracker。
- Draft point-by-point response letter。

人工检查

- 核对结果是否符合当前研究问题和证据边界

## 处理机制

1. **manifest最小复刻**：说明 nature-skills 中 manifest.yaml 的最小可复刻结构。它解释一个 skill 怎样声明必读材料、分支轴、默认值、多选规则和按需深读材料。
2. **全仓机制总图**：解释 nature-skills 怎样从用户科研任务进入 skill、加载材料、生成中间对象、调用工具、交付产物和记录失败。这张卡补的是跨 11 张能力卡的接力关系，不替代单项能力卡。
3. **失败输出合同**：说明 nature-skills 在材料不足、工具缺失、检索失败、证据不足、质量检查不通过时，应该怎样把失败、缺口和风险写进交付物。这张卡补的是“停止后交什么”，不替代各能力卡的具体失败分支。

## 开始方式

1. 选择技能入口：根据当前任务（写作、润色、审稿、引用、图表、数据声明等）选择对应的 nature-* skill
2. 提交材料并接收诊断：提交论文素材，工具先诊断结构、证据或数据缺口，而非直接输出成品
3. 生成结构化中间对象：将论文转为 source map、论证书脊、评论追踪器或数据集清单等可检查对象
4. 输出草稿并标注风险：产生段落、图表、回应信或 PPTX 草稿，同时标注缺失信息和不确定性
5. 逐项复核与迭代：研究者逐一确认人工判断项（证据强度、数据可用性、回应可执行性），修正后交付

## 环境要求

- 论文正文、图表、数据或中文写作草稿
- 目标期刊名称与论文类型
- 审稿意见原文与修改说明（审稿回应场景）
- 需要支撑引用的论文段落或主张列表
- Python 或 R 运行环境（图表生成场景）

## 关键限制

- 提交太薄的材料却期待完整成品——缺少 claim/evidence/boundary 时工具会返回缺口清单而非正文
- 把 metadata-only 引用候选当成真实引文支持，跳过人工核查 abstract 或出版商全文
- 跳过润色前的结构诊断直接做句子级语言修改，掩盖了段落逻辑错误
- 把预审稿三评审报告当作真实同行评审结论或录用概率判断

## 已核对范围

- 真实 naturecitation.py 脚本
- claim segment 生成
- CrossRef metadata 搜索
- Nature/CNS scope 过滤
- RIS 和 review artifact 导出
- 空候选 no candidate 输出

## 仍待核对

- full text 支持判断
- CrossRef 排序随时间漂移
- 10 段以上长文 batch 稳定性
- DOI 文件批量导出
- skill router 自然语言触发
- 非英文 claim 的真实翻译质量

## 许可边界

本页的能力分析、结构与说明由 bo老师整理，采用 CC BY 4.0。上游项目、代码、名称和商标继续遵循各自许可。这里不重新发布上游代码。
