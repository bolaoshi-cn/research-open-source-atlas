# medical-research-skills

554个医学研究专用skill，覆盖协议设计到写作合规的全生命周期

适合让AI agent辅助医学研究设计、数据分析、证据综合和学术写作的研究者。每个skill内置医学专用提示词逻辑和硬性规则，禁止编造文献、禁止直接诊疗。

- 官网详情：https://bolaoshi.cn/research-tools/medical-research-skills
- 原始来源：https://github.com/aipoch/medical-research-skills
- 读取版本：source-reviewed
- 核对日期：2026-06-19
- GitHub Stars：1,397（2026-07-11）
- 上游许可：MIT
- 验证程度：catalogued
- 编辑判断：worth-watching

## 什么时候使用

- 需要辅助设计临床研究协议并用A-L分节结构化输出
- 需要识别研究gap但要求每个gap必须基于真实文献检索
- 需要验证论文中的claim是否被引用文献真实支持
- 需要在医学写作中确保报告指南（CONSORT、STROBE等）合规

## 先准备什么

- 安装skill到支持SKILL.md规范的agent宿主中
- 明确研究类型（队列、RCT、RWE、MR等）以选择对应的专项planner
- 准备好PICO要素和待验证的论文claim列表
- 了解目标期刊的报告指南要求

## 它会怎样推进

1. **选择skill**：根据任务类型选择协议设计、gap识别、数据分析或写作技能
2. **输入研究条件**：提供PICO、暴露-结局设想、数据可得性等约束信息
3. **执行skill**：skill按内置硬规则执行：先判研究类型再匹配方法，禁止编造文献
4. **审查输出**：人工审查生成的协议蓝图或gap分析，验证每个claim的引用来源
5. **合规检查**：通过MedSkillAudit审计输出是否违反医学伦理和文献真实性红线

## 第一次这样开始

帮我安装这个库：https://github.com/aipoch/medical-research-skills
安装前先告诉我需要什么环境，安装后帮我跑通一个最小示例。

## 结果出来后先看什么

- 检查每个gap声明是否都有真实检索结果支撑（无检索不得给gap结论）
- 验证生成协议中的time-zero定义和变量分类是否正确
- 确认输出不包含直接诊疗建议和编造的PMID/DOI

## 常见误区

- 把泛化的'加大样本量'或'加单细胞验证'当作真正的research gap
- 跳过IRB/IACUC伦理审查流程，skill只做提醒不能替代真实伦理审批
- 将skill输出的协议直接当作最终方案而不经过PI和统计师审校

## 能力拆解

### 1. MedSkillAudit双层审计门控

把一个医学研究 skill 经双层审计门控（静态规则检查 + 动态 AI 评估），决定分级部署（通过/带警告部署/拒绝）。双层 veto：Practice Boundaries（M2 红线：禁止直接诊疗）、Scientific Integrity（M1 红线：禁止编造数据）、Methodological Ground（M3 红线：禁止关联冒充因果）、Research Veto（M4：代码可用性硬门）。动态评估用 evaluateskill.py 自动生成 7 类输入（Canonical/Variant A/Edge/Variant B/Stress/Scope Boundary/Adversarial）测试 skill。

证据：source-reviewed

输入

- 一个 skill 目录（SKILL.md + references + 可选 scripts/tests）。

输出

- evalreport.json 审计报告。
- 分级部署决定。

人工检查

- 核对结果是否符合当前研究问题和证据边界

### 2. claim验证与引文真实性核验

把待验证的 claim + cited paper 转成结构化裁决（directly/partially/weakly/unsupported）+ 引用安全改写版本，经 claim 拆解→溯源链→证据类型识别→claim vs paper 范围比对→支持强度判定→因果边界审计→不匹配分类（citation drift/wording inflation/selective citation/causality inflation）。15 条 hard rules 禁止把 review 转述等同于原始证据、禁止关联→因果/模型性能→临床效用/in vitro→人类的越界升级。

证据：source-reviewed

输入

- 待验证 claim + cited paper。

输出

- claim 裁决（四级）。
- 引用安全改写版本。
- 不匹配分类清单。

人工检查

- 核对结果是否符合当前研究问题和证据边界

### 3. 临床研究协议设计

把临床/科研问题转成结构化协议蓝图（A-L 分节：研究意图→设计适配性→队列类型→源人群/入排/time-zero→随访架构→终点→变量收集框架→主分析线→偏倚审查→可行性检查→推荐版本→关键假设），经研究设计类型匹配、time-zero/随访/终点操作化定义、变量分类（exposure/confounder/effect modifier/follow-up）、样本量计算。30+ 疾病/方法专属 planner（队列/RWE/MR/FAERS/单细胞/网络毒学等）+ IRB/IACUC/PROSPERO 注册。

证据：source-reviewed

输入

- 临床/科研问题、PICO 元素、暴露-结局设想、数据可得性约束。

输出

- 结构化协议蓝图（A-L 分节）。

人工检查

- IRB/IACUC/注册助手把伦理审查流程显式嵌入。

### 4. 医学学术写作与报告合规

把协议/分析流程/草稿转成 IMRAD 分节正文 + 报告指南合规审查 + claim 强度校准，经研究类型→报告指南匹配（RCT→CONSORT，观察性→STROBE 等）、claim→证据等级映射（correlation/prediction/mechanism/causation/utility）、过度声明模式识别（causal/mechanism/validation/translational inflation）、改写边界控制。

证据：source-reviewed

输入

- 协议/分析流程、稿件草稿、研究类型、目标期刊、报告指南、reviewer comments。

输出

- IMRAD 分节正文。
- 报告指南合规审查（major gap/moderate/minor/N-A 分级）。
- claim 强度校准报告。

人工检查

- 核对结果是否符合当前研究问题和证据边界

### 5. 医学数据分析确定性管线

把表达矩阵/生存表/基因列表等医学数据转成确定性统计计算结果（非 LLM 幻觉）+ 出版级图表，覆盖差异表达（limma/DESeq2/edgeR）、ML（LightGBM/XGBoost/RF）、聚类（consensus/PCA/UMAP）、富集（GO/KEGG/GSEA）、免疫浸润（CIBERSORT/ssGSEA）、网络（WGCNA/PPI/ceRNA）、生存（KM/Cox/nomogram/decision curve）、Meta 分析（forest/funnel）、质量评估（ROB2/NOS/QUADAS/PROBAST）。代码绑定技能是可执行管线，不是纯 LLM 推理。

证据：source-reviewed

输入

- 表达矩阵 CSV（gene×sample）、分组文件、基因列表、生存表（time/status/group）、CSV/TSV/Excel。

输出

- 差异分析表+火山图+热图 PDF、KM 曲线+风险表 PDF、富集 dot chart、forest plot、sessioninfo.txt、.rda 中间对象。

人工检查

- 核对结果是否符合当前研究问题和证据边界

### 6. 证据综合与研究gap识别

把疾病/基因/通路领域转成结构化研究 gap 图，经证据检索→景观审计→候选 gap→pseudo-gap 拒绝（把"加单细胞/加临床验证/做大样本"等泛化建议默认判为伪 gap，除非绑定到具体未解问题）→置信度分级→研究机会转化。硬规则："No retrieval, no gap claim"：没有真实检索就不许给 gap 结论。配套：MeSH Boolean 检索式构建、证据景观扫描、证据等级排序。

证据：source-reviewed

输入

- 疾病/基因/通路/疗法领域、PICO、锚定论文、主题词。

输出

- 可复制粘贴的 Boolean 查询串。
- 证据景观审计表。
- gap 分类图（八类）。

人工检查

- 核对结果是否符合当前研究问题和证据边界

## 处理机制

1. **医学研究方法论护栏与伦理工程化**：medical-research-skills 的 554 个 skill 共享一套方法论护栏：每个研究类技能内置 Hard Rules（因果边界/文献真实性/越界拒绝），独立合规技能（IRB/IACUC/PHI/注册），MedSkillAudit 双层 veto 作为发布门控，gap-finder 的 pseudo-gap 拒绝和 claim-verifier 的 citation drift 分类作为证据纪律。本机制卡说明这些护栏如何跨技能传递。

## 开始方式

1. 选择skill：根据任务类型选择协议设计、gap识别、数据分析或写作技能
2. 输入研究条件：提供PICO、暴露-结局设想、数据可得性等约束信息
3. 执行skill：skill按内置硬规则执行：先判研究类型再匹配方法，禁止编造文献
4. 审查输出：人工审查生成的协议蓝图或gap分析，验证每个claim的引用来源
5. 合规检查：通过MedSkillAudit审计输出是否违反医学伦理和文献真实性红线

## 环境要求

- 安装skill到支持SKILL.md规范的agent宿主中
- 明确研究类型（队列、RCT、RWE、MR等）以选择对应的专项planner
- 准备好PICO要素和待验证的论文claim列表
- 了解目标期刊的报告指南要求

## 关键限制

- 把泛化的'加大样本量'或'加单细胞验证'当作真正的research gap
- 跳过IRB/IACUC伦理审查流程，skill只做提醒不能替代真实伦理审批
- 将skill输出的协议直接当作最终方案而不经过PI和统计师审校

## 已核对范围

- 静态确认双层 veto（Practice Boundaries M2/Scientific Integrity M1/Methodological Ground M3）
- 静态确认 Research Veto M4（代码可用性硬门）
- 静态确认 evaluateskill.py 的 7 类输入生成（Canonical/Variant A/Edge/Variant B/Stress/Scope Boundary/Adversarial）
- 静态确认分级部署机制
- 静态确认 paper-to-claim-verifier 的 15 条 hard rules
- 静态确认 citation drift 分类法
- 静态确认 claim 裁决四级（directly/partially/weakly/unsupported）
- 静态确认 30+ 疾病/方法专属 planner 的 A-L 分节结构

## 仍待核对

- 未真实运行 evaluateskill.py
- 未真实运行 claim 验证
- 未真实运行任何 planner
- 未验证生成协议的科研质量
- 未真实运行写作技能
- 未真实运行任何分析管线
- 未验证 R 包依赖可用性
- 未真实运行 gap-finder

## 许可边界

本页的能力分析、结构与说明由 bo老师整理，采用 CC BY 4.0。上游项目、代码、名称和商标继续遵循各自许可。这里不重新发布上游代码。
