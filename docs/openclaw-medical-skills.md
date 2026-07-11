# OpenClaw Medical Skills

869 个面向临床研究、生物信息学和药物发现的医疗与生物医学 Skill 集合

适合需要将生物医学数据检索、临床试验匹配、变异分类或治疗计划生成嵌入研究流程的医学研究者与计算生物学家。使用前需仔细审查许可证冲突和临床安全边界，所有输出必须由专业人员复核。

- 官网详情：https://bolaoshi.cn/research-tools/openclaw-medical-skills
- 原始来源：https://github.com/FreedomIntelligence/OpenClaw-Medical-Skills
- 读取版本：source-reviewed
- 核对日期：2026-06-16
- GitHub Stars：2,826（2026-07-11）
- 上游许可：MIT
- 验证程度：catalogued
- 编辑判断：use-with-caution

## 什么时候使用

- 需要批量检索 PubMed、ClinicalTrials.gov、变异数据库等多源生物医学信息
- 需要按 ACMG 证据代码快速生成变异分类草稿
- 需要根据患者摘要初筛和排序候选临床试验
- 需要为慢病、康复或围手术期生成 LaTeX 治疗计划模板

## 先准备什么

- 明确的临床或研究问题
- 相关 API 密钥（PubMed、ClinicalTrials.gov、MyVariant 等服务）
- 患者摘要、变异证据代码或治疗场景说明
- OpenClaw 或兼容运行环境
- 临床专业人员参与复核

## 它会怎样推进

1. **选择并审查目标 Skill**：从 869 个 skill 中筛选所需模块，检查其许可证、依赖和未验证范围
2. **准备输入材料**：提供查询词、患者信息、变异代码或治疗计划类型等结构化输入
3. **运行检索或分类**：通过 BioMCP 多源检索、TrialGPT 匹配或 ACMG 分类器获取候选结果
4. **获取决策候选**：得到文献候选、临床试验排序、变异分类或治疗计划草稿
5. **专业复核与记录**：所有临床相关输出必须由专业人员复核，标注非诊疗边界

## 第一次这样开始

帮我安装这个库：https://github.com/FreedomIntelligence/OpenClaw-Medical-Skills
安装前先告诉我需要什么环境，安装后帮我跑通一个最小示例。
请先说明许可证和安装风险，等我确认后再安装。

## 结果出来后先看什么

- 输出是否明确标注了“非诊疗”或“draft”状态
- 变异分类的 unknown evidence code 是否被记录而非隐藏
- 临床试验匹配结果是否区分了“候选排序”和“入组结论”
- 治疗计划模板是否保留了需人工填写的关键临床字段

## 常见误区

- 忽略根许可证缺失和文件头 proprietary 声明之间的冲突，直接大规模安装使用
- 将简化 ACMG 打分脚本的结果当作正式临床遗传诊断报告
- 让模型在信息不足时做推断——临床试验匹配中缺少关键患者数据时应显式标注而非让模型补全
- 把医学影像 VLM 的 mock 示例或 confidence 值当作影像诊断结论

## 能力拆解

### 1. ACMG变异证据规则分类

把遗传变异证据代码转换成 Pathogenic、Likely Pathogenic、VUS、Likely Benign 或 Benign，并可选生成临床报告草稿。

证据：source-reviewed

输入

- evidence codes，例如 PVS1,PM2。
- 可选 variant name。
- 可选 --report 报告生成开关。

输出

- verdict。
- pathogenicity score。
- benign score。

人工检查

- evidence code 是否由合格人员或可靠数据库给出。
- 是否允许使用简化打分草稿。
- final verdict 是否需遗传专家复核。

### 2. BioMCP多源生物医学检索服务

把文献、临床试验、基因、疾病、药物和变异查询统一暴露成 MCP 工具，让 agent 能用同一接口取得生物医学候选对象和详情。

证据：source-reviewed

输入

- 自然语言 query 或结构化 domain search。
- domain：article、trial、variant、gene、drug、disease 等。
- 字段过滤：gene、variant、disease、trial condition、phase、status、article author、date 等。

输出

- 文献、试验、变异、基因、疾病和药物候选。
- 单条候选详情。
- 搜索 schema 或 query parse 解释。

人工检查

- 是否允许部署外部医学数据库 MCP 服务。
- 是否能接受外部 API 的隐私、权限和审计要求。
- 是否需要按本机 MCP 规则改成 launchd 常驻 HTTP。

### 3. TrialGPT临床试验检索匹配与排序

把患者摘要和条件关键词转换成候选临床试验、逐条入排标准判断、相关性分数和 eligibility 分数，作为后续人工入组评估的候选层。

证据：source-reviewed

输入

- 患者摘要，结构化 JSON 或自由文本。
- 条件关键词、诊断、biomarker、地理位置、phase、intervention 等过滤条件。
- ClinicalTrials.gov 数据 dump 或已检索 trial 集。

输出

- 候选 trial ID 和召回排序。
- inclusion/exclusion criterion-level JSON。
- trial-level relevance 和 eligibility 分数。

人工检查

- 患者信息是否允许发给外部模型。
- 是否使用最新 ClinicalTrials.gov metadata。
- trial score 是否由 clinician 或 trial coordinator 复核。

### 4. 医学影像VLM初筛与报告草稿

把医学影像路径和临床问题传给 VLM，返回影像发现、严重程度、confidence 和建议，形成初筛或报告草稿。

证据：source-reviewed

输入

- 图像路径，文档声称 JPG 或 PNG。
- 临床问题或 generate report 指令。
- 模型名，默认 gemini-1.5-pro，可选 gpt-4o。

输出

- JSON findings。
- severity。
- confidence。

人工检查

- 是否允许上传或处理真实医学影像。
- 影像是否已脱敏。
- 报告草稿是否由合格医师复核。

### 5. 插件清单安装与技能目录治理

把一个大规模医疗 skill 集合登记成 OpenClaw 插件，并给用户提供复制、挂载、稀疏下载和基础校验路径。

证据：source-reviewed

输入

- GitHub 仓库快照。
- openclaw.plugin.json 的插件 ID、版本、描述和 skill 路径列表。
- skills/ 下每个目录的 SKILL.md。

输出

- 可安装 skill 路径清单。
- 基础校验结果。
- 入口文件名大小写不合规、命名不合规、frontmatter 缺失和授权疑点。

人工检查

- 是否允许把上游 skill 目录复制到本地长期 skill 真源。
- 是否接受 README MIT badge 与文件头版权声明之间的授权冲突。
- 是否需要全量扫描所有 869 个 skill 的 frontmatter、依赖和安全边界。

### 6. 治疗计划模板生成与质量校验

把临床场景和治疗计划类型转换成 LaTeX 模板、结构化治疗计划草稿和质量校验反馈。

证据：source-reviewed

输入

- 治疗计划类型。
- 患者去标识化信息。
- 诊断、目标、干预、监测参数、随访计划和风险控制。

输出

- .tex 治疗计划模板。
- 编译后的 PDF。
- validation score 和 missing items。

人工检查

- 临床目标是否适合患者。
- 干预和药物剂量是否符合指南。
- 是否完成去标识化和 HIPAA 检查。

## 处理机制

1. **大规模技能包清单与授权边界**：解释 OpenClaw Medical Skills 如何把 869 个医疗和生物医学 skill 聚合为一个插件包，以及清单、校验、外部依赖和授权冲突如何共同决定可迁移边界。

## 开始方式

1. 选择并审查目标 Skill：从 869 个 skill 中筛选所需模块，检查其许可证、依赖和未验证范围
2. 准备输入材料：提供查询词、患者信息、变异代码或治疗计划类型等结构化输入
3. 运行检索或分类：通过 BioMCP 多源检索、TrialGPT 匹配或 ACMG 分类器获取候选结果
4. 获取决策候选：得到文献候选、临床试验排序、变异分类或治疗计划草稿
5. 专业复核与记录：所有临床相关输出必须由专业人员复核，标注非诊疗边界

## 环境要求

- 明确的临床或研究问题
- 相关 API 密钥（PubMed、ClinicalTrials.gov、MyVariant 等服务）
- 患者摘要、变异证据代码或治疗场景说明
- OpenClaw 或兼容运行环境
- 临床专业人员参与复核

## 关键限制

- 忽略根许可证缺失和文件头 proprietary 声明之间的冲突，直接大规模安装使用
- 将简化 ACMG 打分脚本的结果当作正式临床遗传诊断报告
- 让模型在信息不足时做推断——临床试验匹配中缺少关键患者数据时应显式标注而非让模型补全
- 把医学影像 VLM 的 mock 示例或 confidence 值当作影像诊断结论

## 已核对范围

- 只读上游快照 本地只读快照
- skills/variant-interpretation-acmg/SKILL.md
- skills/variant-interpretation-acmg/README.md
- skills/variant-interpretation-acmg/acmgclassifier.py
- skills/biomcp-server/SKILL.md
- skills/biomcp-server/README.md
- skills/biomcp-server/biomcpserver.py
- skills/biomcp-server/repo/README.md

## 仍待核对

- 未执行 acmgclassifier.py
- 未接入 ClinVar、gnomAD、AlphaMissense、REVEL、CADD、SpliceAI 或 PubMed
- 未核对 ACMG/AMP 真实组合规则
- 未生成或复核临床遗传报告
- 未启动真实 BioMCP server
- 未发起 PubMed、ClinicalTrials.gov、NCI CTS、MyVariant、OpenFDA 或 OncoKB 请求
- 未验证 API key、限流、缓存和网络失败恢复
- 未验证 MCP 客户端连接和工具 schema

## 许可边界

本页的能力分析、结构与说明由 bo老师整理，采用 CC BY 4.0。上游项目、代码、名称和商标继续遵循各自许可。这里不重新发布上游代码。
