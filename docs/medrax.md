# MedRAX

ICML 2025胸片推理agent，用LLM调度多个专用模型回答临床问题

适合研究医学影像AI或需要胸片多维度分析的研究者。MedRAX不训练新模型，而是让大语言模型动态调度7个胸片专用模型完成分类、分割、定位、问答和报告生成。

- 官网详情：https://bolaoshi.cn/research-tools/medrax
- 原始来源：https://github.com/bowang-lab/MedRAX
- 读取版本：source-reviewed
- 核对日期：2026-06-19
- GitHub Stars：1,194（2026-07-11）
- 上游许可：Apache-2.0
- 验证程度：catalogued
- 编辑判断：worth-watching

## 什么时候使用

- 需要同时用多种胸片分析模型（分类+分割+报告生成）回答复杂临床问题
- 研究AI agent如何动态调度领域专用模型完成多步推理
- 需要将DICOM格式医学影像标准化转换为分析就绪的PNG图像
- 需要ChestAgentBench评测集评估胸片AI系统的综合能力

## 先准备什么

- 准备胸部X光片（支持JPG、PNG或DICOM格式）
- 安装模型权重（7个胸片专用模型，需GPU支持）
- 配置多模态LLM的API密钥或本地模型
- 明确要分析的临床问题和期望的输出类型

## 它会怎样推进

1. **加载影像**：上传胸片，DICOM格式自动做窗宽窗位转换并标准化
2. **输入问题**：用自然语言描述要分析的临床问题
3. **Agent决策**：LLM动态判断需要调用哪些专用模型、调用几次
4. **工具执行**：依次运行分类、分割、定位、VQA等模型获取结果
5. **汇总回答**：LLM综合各工具结果，输出结构化报告或可视化图像

## 第一次这样开始

帮我安装这个库：https://github.com/bowang-lab/MedRAX
安装前先告诉我需要什么环境，安装后帮我跑通一个最小示例。

## 结果出来后先看什么

- 检查生成的放射报告中FINDINGS与IMPRESSION是否逻辑一致
- 验证分类结果与分割标注在图像上的空间对应关系
- 确认所有分析结果在临床使用前已经放射科医师复核

## 常见误区

- 将AI生成的胸片分析直接用于临床诊断而不经医生复核
- 在非GPU环境下运行导致推理时间过长或失败
- 把7个模型的输出当作独立诊断而不关注LLM整合过程中的推理逻辑

## 能力拆解

### 1. DICOM到PNG标准化转换

把医学影像的标准 DICOM 格式（.dcm）转换成所有下游胸片分析工具都能接受的 PNG 格式，并提取 DICOM 元数据（PatientID/StudyDate/Modality/PixelSpacing/ImageOrientation 等）。这是多模态医学管线的格式门：其余所有工具只吃 JPG/PNG。

证据：source-reviewed

输入

- .dcm 文件路径。
- 可选 windowcenter、windowwidth。

输出

- PNG 文件路径。
- DICOM 元数据字典（9 字段）。

人工检查

- 无。

### 2. LangGraph多步推理agent编排

给定胸片（经界面 base64 + imagepath 双通道编码）和复杂临床问题（如"对比两张片、定位积液、给出鉴别诊断"），用 GPT-4o 视觉推理配合多个医学工具，经多步 ReAct 循环（先推理→调工具→批判性评估工具输出→决定是否再调），产出针对该问题的综合自然语言回答。这是 MedRAX 区别于"单模型"的根本价值。

证据：source-reviewed

输入

- 胸片（经界面 base64 + imagepath 双通道编码）。
- 复杂临床问题。

输出

- 针对复杂临床问题的多步推理最终自然语言回答（可能附多张工具产出图）。
- toolcalls JSON 审计日志。

人工检查

- 所有工具产出和最终回答均为 AI 辅助，临床诊断必须放射科医师复核。

### 3. 胸片放射报告生成

把一张胸片转换成固定格式的放射报告，包含 FINDINGS 和 IMPRESSION 两段，分别由两个独立模型生成，拼成标准"CHEST X-RAY REPORT"结构。

证据：source-reviewed

输入

- 胸片路径。

输出

- 固定格式文本报告（FINDINGS + IMPRESSION）。

人工检查

- AI 草拟报告，临床使用必须放射科医生复核。

### 4. 胸片病灶短语定位

给定一张正位胸片和一个病灶短语（如"Pleural effusion"），用多模态模型定位病灶位置，输出归一化 bbox 和红框可视化 PNG，并区分"检出"和"未检出（completednofinding）"两种状态。

证据：source-reviewed

输入

- 正位胸片路径。
- 病灶短语（如"Pleural effusion"）。

输出

- 预测列表（每项含 phrase + 模型坐标 bbox + 原图坐标 bbox）。
- 红框可视化 PNG。

人工检查

- 核对结果是否符合当前研究问题和证据边界

### 5. 胸片病理概率分类

把一张胸片（JPG/PNG）转换成 18 类病理的概率分数（0-1），每类含 Atelectasis/Cardiomegaly/Effusion/Pneumothorax/Pneumonia 等，作为下游推理的结构化病理打分输入。

证据：source-reviewed

输入

- JPG/PNG 胸片路径。

输出

- Dict[str, float]：18 类病理名 → 0-1 概率 + metadata。

人工检查

- 概率为辅助筛查信号，临床决策必须医生确认。

### 6. 胸片视觉问答CheXagent

给定多张胸片和自然语言问题（可做 VQA/报告/异常检测/对比/解剖描述/临床解读），用 CheXagent-2-3b 模型生成专家式回答。该工具设 returndirect=True，结果直接返回不再经编排 LLM 加工。

证据：source-reviewed

输入

- 多张胸片路径列表。
- 自然语言 prompt（VQA/报告/异常检测/对比/解剖描述/临床解读）。

输出

- 模型生成的自然语言回答字符串。

人工检查

- 核对结果是否符合当前研究问题和证据边界

## 处理机制

1. **多模态医学推理工具编排**：MedRAX 的多张能力卡共享一条对象链：胸片经 DICOM 标准化后，通过 base64+imagepath 双通道进入 GPT-4o 视觉推理，编排器经 ReAct 循环选择性调用 7 个医学专用工具（分类/分割/grounding/VQA/报告/生成），工具结果回填后由 LLM 批判性评估并决定下一步。本机制卡说明这些对象如何跨能力传递，以及哪些状态会阻断或降级。

## 开始方式

1. 加载影像：上传胸片，DICOM格式自动做窗宽窗位转换并标准化
2. 输入问题：用自然语言描述要分析的临床问题
3. Agent决策：LLM动态判断需要调用哪些专用模型、调用几次
4. 工具执行：依次运行分类、分割、定位、VQA等模型获取结果
5. 汇总回答：LLM综合各工具结果，输出结构化报告或可视化图像

## 环境要求

- 准备胸部X光片（支持JPG、PNG或DICOM格式）
- 安装模型权重（7个胸片专用模型，需GPU支持）
- 配置多模态LLM的API密钥或本地模型
- 明确要分析的临床问题和期望的输出类型

## 关键限制

- 将AI生成的胸片分析直接用于临床诊断而不经医生复核
- 在非GPU环境下运行导致推理时间过长或失败
- 把7个模型的输出当作独立诊断而不关注LLM整合过程中的推理逻辑

## 已核对范围

- 静态确认 DicomProcessorTool 的 pydicom.dcmread → windowing → PNG 保存链路
- 静态确认元数据字典 9 个字段
- 静态确认 Agent 类用 LangGraph StateGraph 构建 process↔execute ReAct 双节点循环
- 静态确认 hastoolcalls 条件边和 execute 并行执行 toolcalls
- 静态确认 systemprompts.txt 的"先推理再调工具、批判性评估工具输出"设计
- 静态确认 MemorySaver 按 threadid 持久化和 savetoolcalls JSON 审计
- 静态确认 ChestXRayReportGeneratorTool 用双 ViT-BERT 模型分别生成 findings 和 impression
- 静态确认固定格式报告结构和 beamwidth=2

## 仍待核对

- 未真实处理 DICOM 文件
- 未验证多帧/增强 DICOM/压缩传输语法的支持边界
- 未真实运行 agent 循环
- 未验证 LangGraph 默认 25 步递归上限是否够用
- 未验证 VLM 自身视觉与工具视觉的互补效果
- 未真实运行报告生成
- 未验证生成报告的临床准确性和连贯性
- 未真实运行 maira-2 推理

## 许可边界

本页的能力分析、结构与说明由 bo老师整理，采用 CC BY 4.0。上游项目、代码、名称和商标继续遵循各自许可。这里不重新发布上游代码。
