# Science-Skills

为AI科研助手提供37个可调用的生物信息学数据库查询和分子分析能力。

面向做生物信息学、药物发现或基因组学研究的科学家，让AI代理能自动查询公共数据库、检索文献、分析蛋白结构和变异。需要为每个数据源配置API访问。

- 官网详情：https://bolaoshi.cn/research-tools/science-skills
- 原始来源：https://github.com/google-deepmind/science-skills
- 读取版本：33557e0f1faf0f281d255940de58935c61b2143b
- 核对日期：2026-06-16
- GitHub Stars：2,330（2026-07-11）
- 上游许可：Apache-2.0
- 验证程度：catalogued
- 编辑判断：worth-watching

## 什么时候使用

- 需要批量查询多个生物信息学数据库（ClinVar、Ensembl、PDB等）而不写脚本
- 想用AI代理自动从候选基因列表查到蛋白结构再到通路富集分析
- 需要查询药物靶点的临床试验和监管数据
- 做文献检索时想同时搜arXiv、bioRxiv、PubMed和OpenAlex

## 先准备什么

- 待查询的基因列表、变异标识（rsID/HGVS/坐标）、UniProt ID或化合物名
- 各数据源的API访问配置（部分需要注册和key）
- 确认目标数据源是否在该skill包的37个skill覆盖范围内

## 它会怎样推进

1. **安装skill包**：安装uv环境和science-skills插件，保证共享HTTP运行层可用
2. **描述任务**：用自然语言告诉AI你要查什么数据库、输入什么标识、想要什么信息
3. **链式查询**：AI代理依次调用相关skill，如从基因查到蛋白结构再查到通路富集
4. **收口报告**：AI汇总查询结果并生成结构化报告，标注数据来源和检索日期

## 第一次这样开始

帮我安装这个库：https://github.com/google-deepmind/science-skills
安装前先告诉我需要什么环境，安装后帮我跑通一个最小示例。

## 结果出来后先看什么

- 返回结果的数据来源和检索日期是否明确标注
- 变异注释的临床意义、频率和约束分数是否一致
- 蛋白结构检索结果是否包含了置信度评估
- 药物靶点查询是否区分了实验证据等级

## 常见误区

- 不验证数据源API的可用性就直接跑——部分数据库有访问限制和速率上限
- 把AlphaFold预测结构当已解析的实验结构使用——预测和实验结构置信度不同
- 跨数据库查询时不核验标识符的版本和物种一致性
- 用生物医学数据库前不检查SKILL_LICENSES.md——不同数据源有不同使用条款

## 能力拆解

### 1. AlphaGenome单变异预测与解释报告

把单个非编码或调控变异、组织或细胞类型、本体标识、轨道类型和目标基因转换成 AlphaGenome 分数、REF/ALT 轨道图、ISM motif 线索和结构化解释报告。

证据：source-reviewed

输入

- 变异输入，通常为 chr:pos:refalt。
- 组织、细胞类型或 ontology CURIE，例如 UBERON、CL 或 EFO。
- 轨道类型或输出类型，例如 RNA-seq、DNASE、ChIP、splicing。

输出

- AlphaGenome prediction 或 score 摘要。
- REF/ALT 轨道图、detail view、whole gene view。
- ISM SeqLogo 和 motif 解释。

人工检查

- 用户是否接受 AlphaGenome 条款。
- 用户是否本地写入 API key。
- 目标组织、细胞类型和 ontology 是否正确。

### 2. skill包分发与运行治理

把一个科学研究 agent 需求转换成可发现、可安装、可运行、可提示授权边界的 skill 包运行环境。

证据：source-reviewed

输入

- 用户或 agent 的科学任务需求。
- Science plugin 安装入口，例如 README 中的 npx skills add google-deepmind/science-skills/ 或 Antigravity Science plugin。
- 每个 skill 目录中的 SKILL.md、可选 scripts/ 和 references/。

输出

- 已安装或可发现的 Science skill 包。
- 目标 skill 的运行前置检查结果。
- uv 隔离运行环境。

人工检查

- 是否安装 uv。
- 是否接受第三方数据源条款。
- 是否在本地 .env 写入 API key 或 contact email。

### 3. 多源文献候选与全文进入

把论文查询词、 DOI、文献 ID 或受限日期范围转换成可落盘的文献候选对象、全文材料、引用列表和后续解析入口。

证据：source-reviewed

输入

- arXiv 查询语法、arXiv ID、下载格式和分页参数。
- bioRxiv 或 medRxiv 的 DOI、server、起止日期、category、keyword、author。
- Europe PMC 查询语法、PMCID、source、article ID、cursor、sort 和 page size。

输出

- 文献候选 JSON。
- PDF、HTML、fulltext text/XML 或 LaTeX source 包。
- PMID、PMCID、DOI、OpenAlex ID、arXiv ID 等身份字段。

人工检查

- 是否允许调用外部文献 API。
- 是否提供 OpenAlex 或 NCBI key。
- 是否接受特定论文或数据源条款。

### 4. 生物医学标识解析与变异注释

把基因、蛋白、rsID、HGVS、基因组坐标、accession 或 PMID 关联线索转换成规范标识、临床解释、群体频率、转录本结构、蛋白序列和变异后果候选。

证据：source-reviewed

输入

- ClinVar Entrez 查询、Variation ID、基因名、临床显著性条件。
- dbSNP rsID、VCF 坐标、HGVS 字符串、assembly。
- Ensembl gene symbol、ENSG、ENST、ENSP、genomic range、species、assembly。

输出

- 规范 ID 和候选对象。
- 临床意义、review status、phenotype、conflict flags。
- rsID、coordinates、placements、HGVS。

人工检查

- 用户是否确认物种和 assembly。
- 是否需要完整 raw payload。
- 是否接受 NCBI、Ensembl、gnomAD、UniProt 条款。

### 5. 药物靶点临床与监管数据查询

把 compound、SMILES、ChEMBL ID、PubChem CID、target、EFO disease、trial filters 或 FDA query 转换成药物、靶点、疾病、试验、监管事件和标签等候选数据对象。

证据：source-reviewed

输入

- molecule name、ChEMBL ID、SMILES、InChIKey、PubChem CID。
- target gene、UniProt accession、Ensembl ID、EFO disease ID。
- ClinicalTrials.gov condition、intervention、phase、status、fields。

输出

- compound、target、disease、trial 和 FDA 候选 JSON。
- activity、drug indication、warning、known drugs、credible sets。
- trial study details、eligibility、outcome、adverse events 模块。

人工检查

- 用户是否接受各数据源条款。
- 是否允许真实调用外部 API。
- 跨库身份冲突时是否合并。

### 6. 蛋白结构序列检索与可视化

把 UniProt ID、蛋白序列、FASTA、多序列文件或结构坐标文件转换成结构文件、置信度分析、序列或结构相似候选、MSA、结构图像和可复查的可视化产物。

证据：source-reviewed

输入

- UniProt accession、PDB ID、FASTA 文件、裸蛋白序列、多序列 FASTA。
- .cif、.mmcif 或 .pdb 结构文件。
- 结构搜索数据库、BLAST 数据库、Clustal Omega 输出路径。

输出

- mmCIF、PAE、metadata 和 pLDDT/PAE 文本解释。
- RCSB PDB 查询 JSON、坐标文件和实验 metadata。
- Foldseek 结构相似候选。

人工检查

- 输入是否是正确的 UniProt ID 或结构文件。
- 结构质量是否足以进入 downstream 分析。
- 是否允许调用外部结构和序列服务。

## 处理机制

1. **共享HTTP与外部API运行治理**：这个横向机制解释 science-skills 怎样在大量外部科学数据库 skill 之间共享 HTTP 请求、限速、重试、凭据、条款通知和大输出落盘规则。

## 开始方式

1. 安装skill包：安装uv环境和science-skills插件，保证共享HTTP运行层可用
2. 描述任务：用自然语言告诉AI你要查什么数据库、输入什么标识、想要什么信息
3. 链式查询：AI代理依次调用相关skill，如从基因查到蛋白结构再查到通路富集
4. 收口报告：AI汇总查询结果并生成结构化报告，标注数据来源和检索日期

## 环境要求

- 待查询的基因列表、变异标识（rsID/HGVS/坐标）、UniProt ID或化合物名
- 各数据源的API访问配置（部分需要注册和key）
- 确认目标数据源是否在该skill包的37个skill覆盖范围内

## 关键限制

- 不验证数据源API的可用性就直接跑——部分数据库有访问限制和速率上限
- 把AlphaFold预测结构当已解析的实验结构使用——预测和实验结构置信度不同
- 跨数据库查询时不核验标识符的版本和物种一致性
- 用生物医学数据库前不检查SKILL_LICENSES.md——不同数据源有不同使用条款

## 已核对范围

- 只读上游仓库 33557e0f1faf0f281d255940de58935c61b2143b
- skills/alphagenomesinglevariantanalysis/SKILL.md
- skills/alphagenomesinglevariantanalysis/docs/alphagenome-api.md
- skills/alphagenomesinglevariantanalysis/docs/interpretation-guide.md
- skills/alphagenomesinglevariantanalysis/docs/report-templates.md
- skills/alphagenomesinglevariantanalysis/scripts/visualizevarianteffects.py
- skills/alphagenomesinglevariantanalysis/scripts/analyzeism.py
- skills/alphagenomesinglevariantanalysis/scripts/visualizegenometracks.py

## 仍待核对

- 未调用 AlphaGenome API
- 未读取或验证 ALPHAGENOMEAPIKEY
- 未生成真实 prediction、ISM、track plot 或报告
- 未验证 ontology mapping、GTF feather 和 splicing zoom 在本机环境的真实行为
- 未安装 Antigravity Science plugin
- 未运行 npx skills add
- 未触发 uv 真实依赖安装
- 未验证个人 skill 目录覆盖策略

## 许可边界

本页的能力分析、结构与说明由 bo老师整理，采用 CC BY 4.0。上游项目、代码、名称和商标继续遵循各自许可。这里不重新发布上游代码。
