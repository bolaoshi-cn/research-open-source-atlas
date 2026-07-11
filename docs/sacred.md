# Sacred

自动记录每次计算实验的配置、种子、依赖和结果，让实验可追溯、可复现。

适合每天跑大量实验变体、需要回溯每次运行用了什么参数和依赖的研究者。用装饰器标注实验入口，Sacred自动捕获环境和配置变更，存到文件或数据库。

- 官网详情：https://bolaoshi.cn/research-tools/sacred
- 原始来源：https://github.com/IDSIA/sacred
- 读取版本：source-reviewed
- 核对日期：2026-07-11
- GitHub Stars：4,367（2026-07-11）
- 上游许可：MIT
- 验证程度：catalogued
- 编辑判断：recommended

## 什么时候使用

- 每天跑几十次参数变体实验，需要一个系统记录每次运行的全貌
- 论文修改阶段需要回溯某张图用的是哪次实验的哪个参数
- 需要保证实验结果可复现——包括配置、种子、依赖和源码版本
- 多个实验想在同一套参数基础上叠加变体，需要层级化配置管理

## 先准备什么

- 实验主函数及其参数，Sacred用装饰器@ex.automain标注入口
- 实验用到的Python环境和包版本
- 目标存储后端选择（默认文件，可切MongoDB/SQL/S3等）

## 它会怎样推进

1. **装饰实验函数**：用@ex.config定义默认参数，@ex.automain标注入口，captured函数自动注入配置
2. **运行实验**：命令行或程序化运行，支持with key=val覆盖参数、命名配置变体和队列模式
3. **自动采集**：运行期间自动收集源码、依赖版本、host信息、git状态和stdout输出
4. **持久化记录**：配置、种子、指标、结果写入文件或数据库，每次run有唯一ID
5. **查询实验**：用sacredboard、omniboard或incense跨run检索、对比和可视化实验记录

## 第一次这样开始

帮我安装这个库：https://github.com/IDSIA/sacred
安装前先告诉我需要什么环境，安装后帮我跑通一个最小示例。

## 结果出来后先看什么

- 每run的config.json是否完整记录了这次运行的全部参数
- run.json中的seed值是否唯一且可复现
- 源码和依赖版本是否与预期一致（审稿前尤其要核对）
- metrics.json的步数和值是否覆盖了整个训练过程

## 常见误区

- 不设置seed就运行——每次seed随机，导致实验不可复现
- 以为Sacred会自动seed所有库——它只处理Python/NumPy/TF/PyTorch四个库
- jsonpickle序列化把tuple变成list后，跨类型比较行为会改变
- QueueObserver在外部服务中断时无最终失败声明，可能无限重试而研究者不知情

## 能力拆解

### 1. 依赖与实验信息自动收集

在不要求研究者手写任何"环境清单"的前提下，自动捕获"这次 run 用了哪些源码、哪些包、什么版本、在什么机器上、git 状态如何"。上游是被 import 的模块与运行进程，下游是 experimentinfo（sources/dependencies/repositories）与 hostinfo，随 run 持久化用于复现审计。这是 sacred"零开销可追溯"的关键支撑。

证据：source-reviewed

输入

- 被 experiment 模块 import 的 Python 模块（用于 inspect 自动发现 source）。
- 运行进程的已安装包（pkgresources 提取 name==version）。
- experiment 所在的 git 仓库（url/commit/dirty）。

输出

- experimentinfo（随 startedevent/queuedevent 持久化）。
- hostinfo（随 startedevent 持久化）。
- 源码快照（FileStorageObserver 存 sources/，MongoObserver 存 GridFS）。

人工检查

- 哪些环境变量进 CAPTUREDENV 由研究者决定（避免泄露敏感值）。
- auto-discovery 漏掉的源码/依赖需人工 addsourcefile/addpackagedependency。
- additionalhostinfo 的自定义 gatherer 正确性由研究者负责。

### 2. 实验信息序列化与前端查询

把分散采集的 config/seed/依赖/host/输出/结果/info/artifacts 序列化成稳定结构（run.json 或 Mongo 文档），并提供跨 run 检索、对比与回读的途径。上游是 Run 对象与 observer 事件，下游是可被前端（sacredboard/omniboard/incense/TinyDbReader）查询与可视化的持久记录。这是 sacred"记录后能用"的出口：没有这一层，记录只是死数据。

证据：source-reviewed

输入

- Run 对象的 config/configmodifications/experimentinfo/hostinfo/info/metainfo/result/status/times/capturedout。
- resource/artifact 文件。
- metrics（run.logscalar 系列）。

输出

- 结构化 run 记录（文件/DB/云）。
- metrics 时序数据。
- 源码/artifact/resource 文件（去重存储）。

人工检查

- observer 与前端选择由研究者/团队决定。
- info dict 内容（如 numpy 数组）序列化后的可读性需人工确认。
- 跨版本数据兼容性需人工核对 format 字段。

### 3. 实验运行与观察者记录

把一次实验执行从"启动→运行→停止"的完整生命周期，连同配置/种子/依赖/host/输出/结果/失败栈，结构化地记录到可插拔的后端。上游是 captured main function 的执行，下游是持久化的 run 记录（DB/文件/云/通知）。这是 sacred"记录每一次实验"目标的主载体，也是区别于"只跑代码不记上下文"的核心。

证据：source-reviewed

输入

- captured main function（@ex.automain/@ex.main）或 command（@ex.command）。
- 已 finalize 的 config（含 seed）。
- experimentinfo / hostinfo（createrun 时采集）。

输出

- 持久化 run 记录（DB/文件/云）：config/experiment/host/info/meta/result/status/times/capturedout/artifacts/resources。
- 通知（Slack/Telegram/Neptune）：停止时消息。
- 源码/资源/产物文件（去重存储）。

人工检查

- observer 选择与配置（DB 连接、认证、bucket）由研究者/运维决定。
- QueueObserver 的无限重试需人工监控。
- DEAD run 判定需人工看 heartbeattime 与 status。

### 4. 配置捕获与注入

把一次实验的"参数集"从硬编码常量升级成可记录、可覆盖、可注入、可校验的配置对象。上游是用户写的 Python 函数体 / dict / 配置文件，下游是一个被自动收集、按 dotted path 嵌套、注入到任意 captured function、随 run 一起持久化的 config dict。这是 sacred 最核心的能力：种子、依赖、host、结果都附在 config 之上。

证据：source-reviewed

输入

- @ex.config 装饰的函数体（局部变量即配置项，可含动态计算与条件分支，无 return/yield）。
- ex.addconfig({...}) 或 ex.addconfig(foo=42, bar='baz')（纯字典，必须 JSON 可序列化）。
- 配置文件：ex.addconfig('conf.json'|'conf.yaml'|'conf.pickle')（YAML 需 PyYAML）。

输出

- 全局 config dict（随 run 持久化到 observer）。
- configmodifications（added/modified/typechanged），用于 printconfig 与可复现审计。
- 注入到所有 captured function 的参数值。

人工检查

- 配置项命名规范、named config 组织方式由研究者自定义（无内建约束）。
- 类型变更告警需人工判断是否为有意行为。
- config scope 内的复杂逻辑（条件、循环）可读性由研究者负责。

### 5. 随机种子层级化管理

让一次实验的所有随机性都可被单一 root-seed 控制、可复现，且各随机调用彼此独立（调用顺序变化不改变结果）。上游是自动生成的 root-seed（或用户指定 seed），下游是全局 PRNG 状态 + 每个 captured function 独立派生的 seed/rnd。这是 sacred 区别于"手动 seed 一次"的关键机制：层级化派生带来抗改动性。

证据：source-reviewed

输入

- 自动生成的 root-seed（getseed 用 random.randint(1, int(1e9))）。
- 用户指定：ex.run(configupdates={'seed': 123}) 或 CLI with seed=123。
- 全局 PRNG 库可用性：numpy（可选）、tensorflow（检测 moduleincache）、pytorch（检测）。

输出

- config 中的 seed（持久化，复现关键）。
- 全局 PRNG 状态（运行期）。
- 每个 captured function 的独立 seed/rnd。

人工检查

- 第三方库随机性是否需手动 seed 由研究者判断。
- cuDNN 等硬件级非确定性 sacred 无法控制，需研究者知晓。
- 跨 numpy 版本复现时是否锁定 legacy API 需人工决定。

## 处理机制

1. **观察者事件生命周期**：解释 sacred 五类能力（配置捕获、种子、依赖采集、运行记录、序列化）的共同架构支柱：RunObserver 基类定义的事件合同 + priority 排序 + 首 observer 决定 id + Run 状态机驱动事件分发。这个横切结构决定了为什么 File/Mongo/TinyDB/SQL/S3/GCS/Slack/Telegram/Neptune 九类后端能共用同一套运行记录逻辑，以及为什么 run 记录、id、status、heartbeat、artifacts 在所有后端表现一致。

## 开始方式

1. 装饰实验函数：用@ex.config定义默认参数，@ex.automain标注入口，captured函数自动注入配置
2. 运行实验：命令行或程序化运行，支持with key=val覆盖参数、命名配置变体和队列模式
3. 自动采集：运行期间自动收集源码、依赖版本、host信息、git状态和stdout输出
4. 持久化记录：配置、种子、指标、结果写入文件或数据库，每次run有唯一ID
5. 查询实验：用sacredboard、omniboard或incense跨run检索、对比和可视化实验记录

## 环境要求

- 实验主函数及其参数，Sacred用装饰器@ex.automain标注入口
- 实验用到的Python环境和包版本
- 目标存储后端选择（默认文件，可切MongoDB/SQL/S3等）

## 关键限制

- 不设置seed就运行——每次seed随机，导致实验不可复现
- 以为Sacred会自动seed所有库——它只处理Python/NumPy/TF/PyTorch四个库
- jsonpickle序列化把tuple变成list后，跨类型比较行为会改变
- QueueObserver在外部服务中断时无最终失败声明，可能无限重试而研究者不知情

## 已核对范围

- 静态确认 gathersourcesanddependencies 通过模块 inspect + pkgresources 自动发现源码与依赖（dependencies.py:726）
- 静态确认 experimentinfo 含 name/basedir/sources/dependencies/repositories（ingredient.py getexperimentinfo）
- 静态确认 collectrepositories 从 sources 提取 git url/commit/dirty
- 静态确认 hostinfo 含 cpu/os/hostname/pythonversion/gpu/ENV，可扩展 hostinfogatherer
- 静态确认 run.json/Mongo runs 文档字段合同（config/experiment/host/info/meta/result/status/times/capturedout/artifacts/resources）
- 静态确认 FileStorageObserver 目录结构（runxxx/{config.json,cout.txt,info.json,run.json,metrics.json} + sources/ + resources/）
- 静态确认 MongoObserver 三集合（runs/fs.files/fs.chunks）+ metrics 集合
- 静态确认前端生态（sacredboard/omniboard/incense/TinyDbReader/Neptune）查询路径

## 仍待核对

- 未运行验证 auto-discovery 在 95% 用例的覆盖率声明
- 未验证 gpu 信息采集在不同硬件下的实际输出
- 未验证 MODULEBLACKLIST 过滤的完整性
- 未运行验证 Mongo 查询的 filter 语法覆盖范围
- 未验证 TinyDbReader 三种检索（indices/expname/query）的实际行为
- 未验证 jsonpickle 序列化对 numpy/pandas 对象的实际输出形态
- 未运行验证 observer 端到端记录
- 未验证 QueueObserver 在外部服务中断时的重试与最终失败边界

## 许可边界

本页的能力分析、结构与说明由 bo老师整理，采用 CC BY 4.0。上游项目、代码、名称和商标继续遵循各自许可。这里不重新发布上游代码。
