# AutoFigure-Edit

把论文方法文本或已有位图转成可编辑SVG插图并自动替换图标

面向需要将科研方法描述或已有位图转为可编辑矢量插图的研究者。先生成或导入位图，再经SAM3分割、去背景、SVG重建和图标替换，最后在浏览器编辑器中微调。

- 官网详情：https://bolaoshi.cn/research-tools/autofigure-edit
- 原始来源：https://github.com/ResearAI/AutoFigure-Edit
- 读取版本：63f3f81cdecaae9d009d9a9b7af2dc7384fbaaec
- 核对日期：2026-06-16
- GitHub Stars：3,949（2026-07-11）
- 上游许可：MIT
- 验证程度：catalogued
- 编辑判断：worth-watching

## 什么时候使用

- 需要将论文方法描述自动生成可编辑的科研插图时
- 已有位图草稿想转成可编辑SVG并替换图标时
- 需要通过Web界面逐步确认分割、重建和优化效果时

## 先准备什么

- 准备论文方法文本，或准备已有第一阶段科研位图文件
- 配置图像生成和SVG生成的provider及API密钥
- 如需SAM分割需配置SAM backend（本地/云端API）
- 如需图标去背景需配置HuggingFace token访问RMBG-2.0模型

## 它会怎样推进

1. **生成位图**：从方法文本生成figure.png，或直接导入已有位图
2. **SAM分割**：用文本提示词检测图中可替换的图标区域，生成标注图和坐标文件
3. **去背景**：按坐标裁切图标并用RMBG-2.0生成透明背景素材
4. **SVG重建**：多模态模型根据原图、标注图和坐标生成带占位符的SVG模板
5. **图标替换**：将透明图标嵌入SVG对应位置，可选LLM优化坐标对齐
6. **编辑器微调**：在浏览器内置svg-edit中手动微调最终效果

## 第一次这样开始

帮我安装这个库：https://github.com/ResearAI/AutoFigure-Edit
安装前先告诉我需要什么环境，安装后帮我跑通一个最小示例。

## 结果出来后先看什么

- SAM检测框是否准确定位了目标图标区域
- SVG模板的占位符位置是否与标注图对齐
- 透明图标替换后尺寸和清晰度是否符合预期
- final.svg是否能在矢量编辑器中正常打开和编辑

## 常见误区

- 把生成位图质量直接等同于最终出版级图表
- 不检查SAM检测框是否漏检或误检就进入后续环节
- 未验证图标去背景后是否保留了必要细节

## 能力拆解

### 1. RMBG图标裁切与透明素材生成

根据 boxlib.json 中的候选框从 figure.png 裁切图标区域，再用 RMBG-2.0 生成透明背景素材，为最终 SVG 中的 <image 嵌入提供可替换图标资产。

证据：source-reviewed

输入

- figure.png
- boxlib.json
- 输出目录

输出

- icons/iconAFxx.png
- icons/iconAFxxnobg.png
- iconinfos

人工检查

- 是否具备 RMBG 模型访问权限。
- 是否允许把图像素材送入本地或远程模型。
- 透明素材是否保留了必要图标细节。

### 2. SAM3多提示词分割与boxlib构建

把 figure.png 中可替换的图标、人物、动物或图形区域识别为带标签的候选 box，生成可视化标注图 samed.png 和结构化 boxlib.json，为 SVG 占位符和图标替换建立坐标合同。

证据：source-reviewed

输入

- figure.png
- SAM prompt 字符串，可用逗号分隔多个 prompt。
- minscore

输出

- samed.png
- boxlib.json
- validboxes

人工检查

- SAM prompt 是否覆盖目标图标类型。
- 阈值是否过低导致候选污染。
- 无图标回退是否符合用户预期。

### 3. SVG优化坐标对齐与最终合成

把初始 template.svg 经过可选多模态优化、坐标系对齐和透明图标替换，输出最终可编辑 SVG final.svg，并在无图标或失败场景下保留可显示的回退产物。

证据：source-reviewed

输入

- template.svg
- figure.png
- samed.png

输出

- optimizedtemplate.svg
- optimizedtemplate.png
- final.svg

人工检查

- 优化后是否比模板更接近原图。
- 图标是否替换到正确位置。
- SVG 是否仍可编辑。

### 4. Web任务调度与可编辑产物回看

把用户在网页配置页或导入页提交的参数转成后端 CLI 子进程任务，并通过 SSE、artifact 列表、历史页和内置 svg-edit 把中间产物与最终 SVG 暴露给用户继续检查和编辑。

证据：source-reviewed

输入

- Web 表单方法文本或导入图片路径。
- provider、模型、key、base URL、SAM backend、prompt、优化轮数和放大开关。
- 用户上传参考图或导入图。

输出

- Web job id。
- /outputs/<jobid/ 下的 artifact。
- Canvas 中的 SVG 编辑器状态。

人工检查

- 是否允许 Web 服务持久保存上传和输出。
- 是否需要清理历史产物。
- 是否允许将 API key 传给后端进程。

### 5. 多模态SVG模板生成与语法修复

用 figure.png、samed.png 和 boxlib.json 作为多模态上下文，让模型生成可编辑的 SVG 模板，并通过 XML 解析和 LLM 修复把模板推进到可解析状态。

证据：source-reviewed

输入

- figure.png
- samed.png
- boxlib.json

输出

- template.svg
- 可能被修复后的 SVG 代码。
- 后续优化和最终合成的输入。

人工检查

- SVG 模板是否保留图表结构。
- 占位符是否和 samed.png 对齐。
- 是否需要跳过优化直接进入最终合成。

### 6. 已有位图导入与预处理

当用户已经有第一阶段科研位图时，把上传或本地路径中的图片归一成常规流程的 figure.png，跳过生图成本，直接进入 SAM 分割和 SVG 重建。

证据：source-reviewed

输入

- 本地已有第一阶段学术位图路径，或 Web 上传后返回的 uploads/<uuid.<ext。
- 支持的图片扩展名：PNG、JPG、JPEG、WEBP、BMP、GIF。
- enableupscale 开关。

输出

- 标准化后的 figure.png。
- 后续 SAM 分割、SVG 生成和 Web artifact 列表。

人工检查

- 图片是否确实是第一阶段科研位图。
- 是否允许把图片上传到当前服务。
- 是否需要开启 4K 放大。

## 处理机制

1. **provider与外部服务路由边界**：AutoFigure-Edit 把生图、文本、多模态 SVG、SAM 和 RMBG 分成多条外部服务路线。本机制卡说明 provider 配置、key 复用、custom base URL、SAM backend 和 HuggingFace gated 模型如何影响能力边界。
2. **五步产物状态链与回退边界**：AutoFigure-Edit 的多张能力卡共享同一条产物状态链：figure.png 先被分割成 samed.png 和 boxlib.json，再生成 icons/、template.svg、optimizedtemplate.svg 和 final.svg。本机制卡说明这些产物如何衔接，以及空检测、SVG 失败和编辑器缺失时如何降级。

## 开始方式

1. 生成位图：从方法文本生成figure.png，或直接导入已有位图
2. SAM分割：用文本提示词检测图中可替换的图标区域，生成标注图和坐标文件
3. 去背景：按坐标裁切图标并用RMBG-2.0生成透明背景素材
4. SVG重建：多模态模型根据原图、标注图和坐标生成带占位符的SVG模板
5. 图标替换：将透明图标嵌入SVG对应位置，可选LLM优化坐标对齐
6. 编辑器微调：在浏览器内置svg-edit中手动微调最终效果

## 环境要求

- 准备论文方法文本，或准备已有第一阶段科研位图文件
- 配置图像生成和SVG生成的provider及API密钥
- 如需SAM分割需配置SAM backend（本地/云端API）
- 如需图标去背景需配置HuggingFace token访问RMBG-2.0模型

## 关键限制

- 把生成位图质量直接等同于最终出版级图表
- 不检查SAM检测框是否漏检或误检就进入后续环节
- 未验证图标去背景后是否保留了必要细节

## 已核对范围

- 静态读取 README 的 RMBG-2.0 说明、HuggingFace token 要求和 cropandremovebackground
- 静态确认 boxlib boxes 会生成 iconAFxx.png 与 iconAFxxnobg.png
- 静态读取 README 的 SAM3 分割说明、segmentwithsam3、box 合并和 API response 解析函数
- 静态确认输出包含 samed.png、boxlib.json、promptsused、boxes 和 noiconmode 标记
- 静态读取 optimizesvgwithllm、getsvgdimensions、calculatescalefactors、replaceiconsinsvg 和 final fallback
- 静态确认输出包含 optimizedtemplate.svg、可选 PNG 预览和 final.svg
- 静态读取 FastAPI 路由、Job 监控、artifact 扫描、Web run payload、SSE 和 svg-edit 加载逻辑
- 静态确认服务把 CLI 子进程产物映射到可回看 artifact 和浏览器编辑器

## 仍待核对

- 未下载 briaai/RMBG-2.0
- 未配置 HFTOKEN
- 未运行真实去背景模型
- 未安装本地 SAM3
- 未调用 fal.ai 或 Roboflow
- 未验证真实检测框质量和 prompt 召回率
- 未渲染真实 SVG
- 未验证图标替换成功率

## 许可边界

本页的能力分析、结构与说明由 bo老师整理，采用 CC BY 4.0。上游项目、代码、名称和商标继续遵循各自许可。这里不重新发布上游代码。
