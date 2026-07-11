# arXiv LaTeX Cleaner

一键清理LaTeX工程并优化体积，准备arXiv提交包

适用于准备向arXiv提交LaTeX论文的研究者。它自动删除开发态注释和条件块，剔除未引用文件，缩放图片并压缩PDF，输出可直接打包上传的发布目录。

- 官网详情：https://bolaoshi.cn/research-tools/arxiv-latex-cleaner
- 原始来源：https://github.com/google-research/arxiv-latex-cleaner
- 读取版本：bcc1460cc4be72ddac08f22c54b23f23f14b102d
- 核对日期：2026-07-10
- GitHub Stars：6,939（2026-07-11）
- 上游许可：Apache-2.0
- 验证程度：catalogued
- 编辑判断：recommended

## 什么时候使用

- 准备向arXiv提交论文前，需要清理开发态的注释和私有命令
- 论文工程包含大量未引用的tex文件和图片，需要自动剔除冗余文件
- 论文图片和PDF体积过大，需要达到arXiv 50MB限制要求

## 先准备什么

- 准备完整的LaTeX论文工程目录，确保可正常编译
- 确认commands_to_delete等清理配置列表，防止误删正文命令
- 可选：安装Ghostscript以启用PDF压缩功能
- 确认输出目录_arXiv中无需要保留的内容（会被覆盖重建）

## 它会怎样推进

1. **清理源文件**：自动删除行内和整行注释、comment环境、条件编译块和自定义命令，保护隐私
2. **筛选文件**：从根tex出发迭代收敛引用闭包，剔除未被引用的tex文件和辅助文件
3. **处理图片**：按配置缩放图片最长边、用Ghostscript压缩PDF，可选将大PNG转JPG
4. **回写引用**：图片格式变化后用正则策略回写tex中的includegraphics引用路径
5. **输出发布包**：将全部清理产物输出到_arXiv目录，打包后即可上传arXiv系统

## 第一次这样开始

帮我安装这个库：https://github.com/google-research/arxiv-latex-cleaner
安装前先告诉我需要什么环境，安装后帮我跑通一个最小示例。

## 结果出来后先看什么

- 清理后的tex文件是否仍能正常编译，确认没有误删关键命令或引用
- 图片质量和分辨率是否满足期刊或arXiv的清晰度要求
- 确认注释和条件块已被正确清理，无隐私泄漏风险
- 确认_arXiv目录包含所有必需文件，未被误剔除被引用的资源

## 常见误区

- 未配置commands_to_delete列表导致私有命令和Todo注释保留在提交文件中
- 未将非标准if命令加入if_exceptions清单，导致合法命令被误判为条件块开头
- 运行前未检查_arXiv目录内容，被覆盖重建导致丢失需要保留的自定义文件
- 在无Ghostscript的Windows环境启用compress_pdf，该功能只在Linux/Mac可用

## 能力拆解

### 1. LaTeX隐私清理

把 LaTeX 论文工程中所有 .tex 文件里会泄漏私有笔记的内容（行内注释、整行注释、comment 环境、\iffalse/\iftrue/\if0 条件编译块、用户自定义命令、自定义环境、tikzpicture 源码、includesvg 调用、自定义快捷命令）删除或替换，输出不含可读私有痕迹的 .tex 文本。

证据：source-reviewed

输入

- 输入目录中的全部 .tex 和 .tikz 文件（含根目录和非根目录）。
- CLI 参数或 cleanerconfig.yaml：

输出

- 清理后的 .tex 文件，写入 <inputarXiv/ 目录。
- 这些 tex 文件供体积优化阶段的引用收敛使用（决定哪些文件和图片被保留）。

人工检查

- 用户必须确认 commandstodelete、commandsonlytodelete、environmentstodelete 列表，否则会误删论文正文命令或漏删私有命令。
- 用户必须确认 ifexceptions 是否覆盖所用宏包的非 primitive \if 命令。
- 用户必须确认 regex patternsandinsertions 不会误匹配正文。

### 2. arXiv提交体积优化

把 LaTeX 论文工程中未被引用的 tex 文件、未被引用的图片、辅助文件（.aux/.log/.out 等）和可选的 .bib 文件剔除，对被引用的图片按像素上限缩放、对被引用的 PDF 用 ghostscript 降分辨率压缩、对被引用的 PNG 按阈值转 JPG，并回写 tex 中的图片引用，输出一个体积适配 arXiv 50MB 限制的可打包 arXiv 目录。

证据：source-reviewed

输入

- 输入目录（LaTeX 工程）或 .zip 包。
- CLI 参数或 cleanerconfig.yaml：
- 来自隐私清理阶段的 texcontents（已清洗的 tex 全文，用于引用匹配）。

输出

- <inputarXiv/ 目录，内含：
- 该目录可直接 ZIP 后上传 arXiv。

人工检查

- 用户必须确认 imsize 和 pdfimresolution 是否满足目标体积和清晰度。
- 用户必须确认 imagesallowlist 中高分辨率图片的豁免是否正确。
- 用户必须确认 keepbib 是否需要保留 .bib（arXiv 有时要求）。

## 处理机制

1. **核心处理链**：一键清理LaTeX工程并优化体积，准备arXiv提交包

## 开始方式

1. 清理源文件：自动删除行内和整行注释、comment环境、条件编译块和自定义命令，保护隐私
2. 筛选文件：从根tex出发迭代收敛引用闭包，剔除未被引用的tex文件和辅助文件
3. 处理图片：按配置缩放图片最长边、用Ghostscript压缩PDF，可选将大PNG转JPG
4. 回写引用：图片格式变化后用正则策略回写tex中的includegraphics引用路径
5. 输出发布包：将全部清理产物输出到_arXiv目录，打包后即可上传arXiv系统

## 环境要求

- 准备完整的LaTeX论文工程目录，确保可正常编译
- 确认commands_to_delete等清理配置列表，防止误删正文命令
- 可选：安装Ghostscript以启用PDF压缩功能
- 确认输出目录_arXiv中无需要保留的内容（会被覆盖重建）

## 关键限制

- 未配置commands_to_delete列表导致私有命令和Todo注释保留在提交文件中
- 未将非标准if命令加入if_exceptions清单，导致合法命令被误判为条件块开头
- 运行前未检查_arXiv目录内容，被覆盖重建导致丢失需要保留的自定义文件
- 在无Ghostscript的Windows环境启用compress_pdf，该功能只在Linux/Mac可用

## 已核对范围

- 静态读取 README.md、arxivlatexcleaner/arxivlatexcleaner.py 全文
- 静态确认注释删除、条件块简化、命令删除、环境删除、TikZ/SVG/regex 替换的调用链
- 静态确认 testdata/ 提供预期输出快照
- 静态确认文件分流、引用收敛、图片缩放、PDF 压缩、PNG 转 JPG 及引用回写的调用链
- 静态确认 testdata/ 提供 texarXivtrue 和 texarXivpng2jpgtrue 预期输出快照

## 仍待核对

- 未安装依赖
- 未运行 CLI
- 未用真实论文工程验证 regex 边界
- 未测试嵌套命令删除和跨行注释的极端情况
- 未安装依赖（pillow、ghostscript）
- 未验证 ghostscript PDF 压缩的实际效果
- 未验证 PNG 转 JPG 后引用回写的正确性

## 许可边界

本页的能力分析、结构与说明由 bo老师整理，采用 CC BY 4.0。上游项目、代码、名称和商标继续遵循各自许可。这里不重新发布上游代码。
