# DVC

数据与模型版本管理工具，为机器学习项目提供可复现流水线

适合需要版本管理大规模数据和模型的机器学习和数据科学团队。它把Git的版本控制思维用于数据文件，结合流水线编排和实验跟踪，让项目从数据到模型的每一步都可复现、可比较、可共享。

- 官网详情：https://bolaoshi.cn/research-tools/dvc
- 原始来源：https://github.com/iterative/dvc
- 读取版本：f74c1c0e709de61f571905802bc0c75035dc6ef2
- 核对日期：2026-07-10
- GitHub Stars：15,739（2026-07-11）
- 上游许可：Apache-2.0
- 验证程度：verified
- 编辑判断：recommended

## 什么时候使用

- 数据文件太大不能被Git直接管理
- 需要追踪每次实验的参数、指标和产物并方便对比
- 多个数据处理和训练步骤需要增量重算而非每次全跑
- 团队需要共享同一份数据集并在各仓库间版本锁定引用

## 先准备什么

- 在项目根目录初始化Git仓库和DVC仓库
- 在dvc.yaml中声明数据处理和模型训练的各阶段及依赖关系
- 配置远端存储URL和凭证用于数据共享

## 它会怎样推进

1. **版本化数据**：用dvc add把数据文件纳入内容寻址缓存，Git只跟踪轻量元数据指针
2. **定义流水线**：在dvc.yaml中声明各处理阶段的输入、输出、命令和参数依赖
3. **增量复现**：运行dvc repro，只重跑上游依赖或参数发生变化的后继阶段
4. **实验迭代**：用dvc exp run跑多个参数组合，通过exp show对比指标并随时回滚
5. **远端同步**：用dvc push/pull把缓存对象与S3/GCS等远端双向同步

## 第一次这样开始

帮我安装这个库：https://github.com/iterative/dvc
安装前先告诉我需要什么环境，安装后帮我跑通一个最小示例。

## 结果出来后先看什么

- dvc.lock中记录的依赖哈希是否与当前文件一致
- dvc repro后metric文件中的数值是否在预期范围内
- dvc exp show的指标对比表是否清楚展示了实验间的差异
- 远端同步状态中missing项是否指向已被回收的对象

## 常见误区

- 把大文件直接提交到Git而不是只用DVC跟踪其.dvc元数据
- dvc exp apply前未保存当前工作区未提交改动导致覆盖
- 在多仓库间import数据后忘记用dvc update同步上游新版本

## 能力拆解

### 1. 实验跟踪

在不搭建中心化 ML 实验服务器的前提下，利用本地 Git 仓库跟踪大量实验运行（参数、指标、产物、队列执行），并支持按指标比较、筛选、回滚和共享实验，把单机或队列化的实验迭代纳入版本治理。

证据：source-reviewed

输入

- 现有 dvc.yaml pipeline（实验复用 pipeline 的 stage 定义）。
- 参数覆盖（--set-param，转 hydra overrides）或 hydra sweep 声明。
- baseline Git commit（实验基于哪个提交跑）。

输出

- 实验 Git ref（refs/exps/），可被 exp apply/branch/push/pull 引用。
- 实验产物（入 cache，可通过 exp push 共享）。
- metric 汇总表（exp show）。

人工检查

- exp apply 会覆盖 workspace 未提交改动，需人工确认。
- celery 并行实验的资源边界（CPU/内存/GPU）需人工配置。
- Studio 上传涉及外部服务和 token，授权边界需人工确认。

### 2. 数据导入与注册中心

从另一个 DVC/Git 仓库、远端 URL 或数据库拉取数据作为当前仓库的依赖，并以 frozen import 形式锁定版本，后续可用 dvc update 拉取上游新版本，从而把多个仓库组织成数据注册中心式的依赖网络。

证据：source-reviewed

输入

- 源仓库 URL（dvc import 的 url 参数，可以是 GitHub repo、本地路径）。
- 源仓库的 rev/branch/tag（--rev）。
- 源仓库内的数据路径（path）。

输出

- 本地工作区数据文件（import/import-url）。
- import stage 记录（入 dvc.yaml/.dvc，frozen）。
- 依赖哈希（入 dvc.lock，供 update 比对）。

人工检查

- 源仓库的授权和访问边界需人工确认（尤其私有仓库）。
- import 锁定的 rev 是否为预期版本需人工复核。
- dvc update 拉取的新版本可能改变下游结果，需人工确认是否触发 repro。

### 3. 数据版本管理

把工作目录里的数据文件或模型文件纳入版本控制，让 Git 只跟踪轻量 .dvc 元数据，而真实大文件存放在内容寻址缓存里，从而实现“Git 跟踪版本、缓存存内容”的数据版本管理。

证据：source-reviewed

输入

- 工作目录中的一个或多个数据文件、目录或模型文件（如 images/、model.p）。
- Git 仓库（可选 core.noscm，但数据版本管理默认依赖 Git 跟踪 .dvc 元文件）。
- .dvc/config 中的 cache 类型、link 类型配置（reflink/hardlink/symlink/copy）。

输出

- <name.dvc 元数据文件（入 Git）。
- .dvc/cache 下的内容寻址对象。
- .gitignore 更新（排除真实数据文件）。

人工检查

- link 类型选择影响磁盘和跨平台行为，生产环境需确认文件系统支持。
- 共享 cache 目录（sitecachedir）的多用户隔离边界需人工确认。

### 4. 流水线编排与repro

把多个数据处理或模型训练步骤描述成 dvc.yaml 里的有向无环图（DAG）阶段，当依赖或代码变化时只重跑受影响的后继阶段（dvc repro），从而实现增量、可复现的 ML 流水线。

证据：source-reviewed

输入

- dvc.yaml：声明 stages，每个 stage 有 cmd、deps（依赖）、outs（输出）、params（参数依赖）、wdir、foreach/matrix（参数化）。
- params.yaml：参数文件，作为 stage 的依赖被哈希跟踪。
- 工作目录的依赖文件（代码、数据）和上一 stage 的输出。

输出

- dvc.lock 更新（每个 stage 的新 deps/outs 哈希）。
- stage 的输出文件（入 DVC cache 和工作区）。
- metric 文件（-M，供 dvc metrics show 读取）。

人工检查

- onerror=keep-going 时失败 stage 的后继被跳过，需人工确认部分结果是否可信。
- run cache 复用历史 outs 前提是 cmd 和环境未变，跨机器复用需确认环境一致。
- parametrized stage 展开后的命名冲突需人工复核。

### 5. 远端存储同步

把本地 .dvc/cache 里的内容寻址对象与一个或多个远端对象存储（S3、GCS、Azure、SSH、OSS、HTTP、本地路径等）双向同步，实现数据缓存的共享、备份和团队协作，同时保持 Git 元数据的轻量。

证据：source-reviewed

输入

- 本地 .dvc/cache 的对象集合（由 usedobjs 收集当前仓库实际引用的对象）。
- .dvc/config 的 [remote] 段（远端 name → url）和 [core] remote（默认远端）。
- 远端凭证（环境变量、~/.aws/credentials、服务账号 JSON、SSH key 等，由对应 dvc-<scheme 包处理）。

输出

- 远端 odb 上的对象集合（push）。
- 本地 cache 对象 + 工作区文件（pull/fetch）。
- 同步状态表（status：ok/missing/new/deleted）。

人工检查

- 云凭证和 bucket 权限边界需人工配置。
- --all-commits/--all-experiments 会收集大量历史对象，成本和耗时需人工评估。
- worktree 远端的版本保留策略和成本需人工确认。

## 处理机制

1. **内容寻址缓存与对象数据库**：DVC 的数据版本管理、流水线复现、实验跟踪、远端同步和数据导入五张能力卡，都依赖同一个横向机制：内容寻址缓存（content-addressable cache）。这个机制解释了为什么 DVC 能用轻量 Git 元数据管理大文件、为什么 push/pull 是云无关的、以及为什么多个能力能共享同一份本地存储。

## 开始方式

1. 版本化数据：用dvc add把数据文件纳入内容寻址缓存，Git只跟踪轻量元数据指针
2. 定义流水线：在dvc.yaml中声明各处理阶段的输入、输出、命令和参数依赖
3. 增量复现：运行dvc repro，只重跑上游依赖或参数发生变化的后继阶段
4. 实验迭代：用dvc exp run跑多个参数组合，通过exp show对比指标并随时回滚
5. 远端同步：用dvc push/pull把缓存对象与S3/GCS等远端双向同步

## 环境要求

- 在项目根目录初始化Git仓库和DVC仓库
- 在dvc.yaml中声明数据处理和模型训练的各阶段及依赖关系
- 配置远端存储URL和凭证用于数据共享

## 关键限制

- 把大文件直接提交到Git而不是只用DVC跟踪其.dvc元数据
- dvc exp apply前未保存当前工作区未提交改动导致覆盖
- 在多仓库间import数据后忘记用dvc update同步上游新版本

## 已核对范围

- 只读上游快照 本地只读快照，commit f74c1c0e709de61f571905802bc0c75035dc6ef2
- README.rst、dvc/repo/experiments/init.py、dvc/repo/experiments/run.py、dvc/repo/experiments/queue/base.py、dvc/repo/metrics/show.py
- dvc/commands/experiments/init.py（exp 子命令清单）、dvc/repo/experiments/refs.py、dvc/repo/experiments/stash.py
- README.rst、dvc/repo/imp.py、dvc/repo/impurl.py、dvc/repo/impdb.py、dvc/repo/get.py、dvc/repo/geturl.py、dvc/repo/update.py、dvc/repo/datasets.py
- dvc/stage/imports.py（syncimport/updateimport）、dvc/repo/openrepo.py
- README.rst、dvc/repo/add.py、dvc/repo/init.py、dvc/dvcfile.py、dvc/cachemgr.py
- tests/unit/testdvcfile.py、tests/func/testadd.py 的存在与命名
- README.rst、dvc/repo/reproduce.py、dvc/stage/init.py、dvc/stage/loader.py、dvc/stage/run.py、dvc/dvcfile.py

## 仍待核对

- 未运行 dvc exp run/dvc exp show/dvc exp apply
- 未验证 workspace/tempdir/celery 三种 queue 的实际隔离与并发
- 未验证 hydra sweep 的实际展开
- 未验证 Studio live metrics 的上传链路
- 未验证 exp ref（refs/exps/...）与 Git 提交的交互
- 未运行 dvc import/import-url/get/update
- 未验证跨仓库数据注册中心的实际拉取与版本锁定
- 未验证 dvc update 拉取上游新版本的行为

## 许可边界

本页的能力分析、结构与说明由 bo老师整理，采用 CC BY 4.0。上游项目、代码、名称和商标继续遵循各自许可。这里不重新发布上游代码。
