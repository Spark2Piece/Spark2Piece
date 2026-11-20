# Spark2Piece

<p align="center">
  <img src="assets/logo/spark2piece-logo.svg" width="220" alt="Spark2Piece">
</p>

<p align="center"><strong>从灵感到可交付成果。</strong><br>From an idea to a complete, shippable product.</p>

<p align="center">
  <a href="README.en.md">English</a>
  ·
  <a href="LICENSE">MIT License</a>
  ·
  <a href="https://github.com/Spark2Piece">GitHub Organization</a>
</p>

Spark2Piece 要做成一个 **Agent 平台**，而不是再做一个单会话聊天 Agent。

组织沿四个方向推进。本仓库先落地前两个方向的基础能力：Platform Use To Agent（PU2A）和 Workflow Design To Agent（WD2A）。

- [组织方向](#组织方向)
- [Platform Use To Agent（PU2A）](#platform-use-to-agentpu2a)
- [Workflow Design To Agent（WD2A）](#workflow-design-to-agentwd2a)
- [许可证](#许可证)

## 组织方向

**Platform Use To Agent（PU2A）** 解决 Agent 抽象（分为 Agent Prompt 与 Skill）和 Agent 运行时，以及 Agent 的调用方式与权限。调用分为单独使用、组成 Group 使用，以及 Agent 之间相互作为 Subagent 调用。附加能力包括 Workspace 层级、Issue / 自动化、Skill 管理与分享等。

**Workflow Design To Agent（WD2A）** 解决 Workflow 的抽象与设计，以及 Workflow 的执行和监控。

**Architecture Language To Agent（AL2A）** 解决 Agent 如何读取中大型项目的架构设计语言，并进行解析和执行，而不是目前这种每次都 grep 代码的方式。

**Memory Management To Agent（MM2A）** 解决 Agent 的记忆管理，覆盖性能、召回率以及数据隐私。

---

# Platform Use To Agent（PU2A）

PU2A 把「是什么 Agent」从运行时拆出来，让同一套 Runtime 能跑不同类型的 Agent，并以清晰的调用协议和权限模型支撑单独使用、Group 协作、Subagent 互调与 ChildAgent。Issue / Task 作为跨会话的工作单元，Workspace 提供层级边界。Sandbox、外部连接和观测台是平台的一等能力，而不是事后补丁。

## 基础功能

### Agent 配置

把「是什么 Agent」从运行时拆出来，至少分两层：

- **Prompt**：角色、目标、约束、输出规范
- **Skill**：可复用的能力包（工具绑定、操作手册、输入输出约定），可安装、分享、版本化

### Runtime

Runtime 只负责跑。会话循环、工具调用、事件接入、并发、失败恢复，不和某个具体 Agent 定义绑死。同一 Runtime 能跑不同类型的 Agent。

### Agent Protocol

- **单独使用**：一个 Agent 独立完成任务
- **Group 使用**：多个 Agent 组成协作组，统一对外
- **相互作为 Subagent 调用**：Agent 之间可互为子代理
- **ChildAgent**：由父 Agent 派生、生命周期受父级约束的子实例

### 上下文载体

Issue / Task 作为工作单元，承载目标、状态、产物、讨论和后续事件。Workspace 提供层级边界（个人 / 项目 / 团队）。

### 附加能力

Workspace 层级、Issue / 自动化、Skill 管理与分享。

### 权限

数据访问权限（能看什么）和操作权限（能改什么、能调什么工具、能以谁的身份调用其他 Agent）分开建模，支持最小权限和可审计。

## 外部依赖

- **外部平台**：能发现、拉取、引用外部的 Skill / Agent / Workflow / Workspace / Issue / Task。
- **Sandbox**：开箱即用的沙箱即服务。工具和代码执行默认隔离，按任务或 Agent 分配执行环境；覆盖 Code Interpreter、Browser Use，以及规划中的 Computer / Mobile Use。
- **外部连接**：对 IM、代码平台、邮件、内部系统等保持连接性与可用性（重连、限流、降级、健康检查）。
- **观测台**：分析 Coding Agent 的执行质量、失败模式，以及工程流程是否闭环。Token 用量和原始日志只作为原料。

## 开箱即用的 Sandbox

Sandbox 是 PU2A 的默认执行环境，由平台以沙箱即服务的方式统一提供，开发者无需自建、打补丁和依赖管理。Agent 的代码执行、浏览器自动化和后续的 GUI 操作都跑在这套隔离环境里。

### 沙箱即服务（Sandbox as a Service）

- **Code Interpreter**：安全执行 Python / Node.js / Java 等代码，支持文件管理和会话管理。
- **Browser Use**：通过 CDP over WebSocket 提供稳定的浏览器自动化环境，兼容 Puppeteer / Playwright。
- **Computer / Mobile Use**（规划与扩展方向）：支持 GUI 操作、远程桌面等高级操作。

### 企业级安全隔离

- 基于安全容器（MicroVM）的多级隔离：请求级、实例级、会话级。
- 存储隔离，支持挂载 OSS / NAS 等存储。
- 沙箱环境由平台统一维护与升级，开发者无需自建、打补丁和依赖管理。

### Serverless Sandbox 性能

- **浅休眠，毫秒级唤醒**：短暂空闲的实例自动进入浅休眠，有请求到达时可在毫秒级完成唤醒，几乎无冷启动感知。
- **深休眠，秒级唤醒**：长时间无流量的实例进入更节能的深休眠，仍可在秒级范围内快速恢复，兼顾成本与性能。
- **百万沙箱模板并发**：底座支持百万级沙箱模板（函数级别）并发运行，可支撑大规模、多类型沙箱在同一平台上同时工作，适合多团队、多业务线共享的企业级场景。

## 观测台（Agent Observatory）

观测台是 PU2A 的一等能力，与 Agent 抽象、Runtime、Sandbox 同级。它消费会话、任务和时间线，回答三件事：这次 Coding Agent 做得好不好、失败属于哪一类、工程有没有从 Issue 走到可合并的结果。Token 和原始日志只作为原料。

WD2A 的监控回答工作流跑到哪、为什么停；观测台回答编码执行的质量、失败模式和工程闭环。

- **看什么**：多 Provider 的 Coding Agent 会话（Claude Code、Codex 等）、子任务、时间线、执行状态；需要处理的队列（审批、报错、未读）。用量（Token、模型、缓存、额度）保留，但是输入而不是结论。
- **执行质量**：对照 Issue / Task 目标，看结果是否可审查、是否半成品（只改代码、未开 PR、CI 未过、未合并、Issue 未关）。
- **失败模式**：权限拒绝、工具空转、上下文耗尽、未经验证就宣称完成、重复重试、子 Agent 串台、Sandbox 失败等，要能归因。
- **工程闭环**：Issue → 会话 → 代码变更 → 评审 / CI → 合并或关闭，标出断点。
- **诊断与报告**：每个会话给出结果、时间线、诊断和执行复盘；智能分析和每轮小结默认关闭，由人发起。报告引用时间线、产物和状态；缺遥测标为不可用，不记成 0。用量统计只保留计数、模型、工作目录和会话标识，不把提示词和输出当成统计原料。
- **状态**：运行中、等待审批、失败、已归档；异常可定位，并可重试或移交。

## 要解决的核心问题

- 抽象和 Runtime 耦合后，Agent 无法跨环境复用，也无法单独演进。
- 调用关系不清晰时，Group / Subagent / ChildAgent 会抢回复、上下文串台、权限被放大。
- 没有稳定的 Issue / Task 载体，多任务只能靠人手切换 Session，连续性断在会话上。
- Agent 接入真实系统和数据后，权限不再是可选能力，而是能不能规模化的前提。
- Sandbox 和外部连接不足时，要么不敢给权限，要么一给就有安全与稳定性风险。
- 沙箱要同时满足隔离强度、唤醒体验和规模成本：请求 / 实例 / 会话多级隔离与存储隔离必须默认具备；短暂空闲要能毫秒级唤醒，长空闲要能深休眠秒级恢复。环境由平台统一维护，避免每个团队自建、打补丁和管依赖。
- Skill / Agent / Workflow 不能被发现和分享时，平台会变成单机工具，而不是可组合的 Agent 平台。
- 只有 Token 用量和分散在各 Provider 的本机日志时，看不出执行质量、失败类型和工程闭环断在哪，也无法反推 Runtime、权限和 Skill 该怎么改。

## 可选的开源参考方案

| 方向 | 可参考 | 说明 |
| --- | --- | --- |
| Agent 抽象 + Runtime | [LangGraph](https://github.com/langchain-ai/langgraph)、[AutoGen / AG2](https://github.com/ag2ai/ag2)、[CrewAI](https://github.com/crewAIInc/crewAI)、[OpenHands](https://github.com/All-Hands-AI/OpenHands) | Harness / Runtime、图调度、角色协作、编码 Agent 沙箱运行时 |
| A2A 调用 | [A2A Protocol](https://github.com/a2aproject/A2A) | Agent 之间的发现、委托与互调协议；MCP 更偏 Agent 调工具 |
| 工具与外部连接 | [MCP](https://modelcontextprotocol.io/) | 统一接入外部平台和工具，补齐连接性 |
| Sandbox | [E2B](https://github.com/e2b-dev/e2b)、[Daytona](https://github.com/daytonaio/daytona)、OpenHands Docker / gVisor / Firecracker 一类隔离方案；浏览器自动化可参考 CDP / [Playwright](https://github.com/microsoft/playwright) / Puppeteer | 按任务隔离执行环境；沙箱即服务、MicroVM 多级隔离与 Serverless 休眠唤醒可在此类方案上扩展 |
| Skill 管理分享 | Anthropic / 社区的 Agent Skill 形态，以及各平台的 Skill 市场实践 | Prompt 与可复用 Skill 分离、安装和分发 |
| 上下文载体 | Issue / Task 工作流（GitHub / Linear 等）+ 事件驱动会话（如 DeepSeek Harness Super-agent 实践） | 用 Issue / Task / Thread 做跨会话载体，而不是把 Session 暴露给用户 |
| 观测台 | [Paseo Agent Observatory](https://paseo.sh/zh-CN/plugins/zhjiang22/agent-observatory)（zhjiang22/agent-observatory） | 结果优先看板、时间线、子任务、诊断、执行复盘、未读与审批、定时运行、本机 Token 统计、可选智能分析。该插件停在会话观测和复盘；本平台还要质量判断、失败模式和工程闭环 |

---

# Workflow Design To Agent（WD2A）

WD2A 要完整覆盖传统工作流，并支持将其平替为 Agent 化执行。它把 Chat、Race（Pick First）、Batch、Flow、Pipeline 等模式抽象为统一对象，以主动式 Super Agent 作为核心调度：主动分析、解决问题，并具备平台中该用户角色的全部权限（等效宿主用户）。

关键不是再发明一种流程引擎，而是把这些抽象变成 Agent 可设计、可执行、可监控的一等对象。

## 基础功能

- **传统工作流覆盖**：已有的审批流、CI、批处理、人工节点、重试、超时、补偿，要能平移到 Agent 化执行，而不是另起一套互不兼容的流程。
- **工作流抽象**，至少覆盖：
  - **Chat**：以对话推进，边聊边改计划，适合目标还不稳的任务
  - **Race（Pick First）**：多路并行尝试，先完成且可用的结果胜出，其余取消或丢弃
  - **Batch**：同一套步骤对多条数据 / 多任务并发或分批执行，支持部分失败
  - **Flow**：有分支、汇合、循环、人工确认的有向流程
  - **Pipeline**：按阶段顺序推进，上一阶段产物作为下一阶段输入
- **Super Agent 核心调度**：主动分析与解决问题，统一调度上述工作流；权限等效宿主用户，具备平台中该角色的全部权限。
- **设计能力**：可视化或代码化编排、复用子流程、参数化、版本管理。
- **执行能力**：启动、暂停、恢复、取消、重试、幂等、人工卡点（确认后再继续）。
- **监控能力**：运行状态、当前节点、耗时、失败原因、重放、告警，能回答「跑到哪了、为什么停」。

## Super Agent

Super Agent 是 WD2A 的主动式核心调度，不是又一个普通执行 Agent。五种工作流抽象（Chat / Race / Batch / Flow / Pipeline）汇入它，再由它调度执行引擎去启动、恢复、重试，并在需要时加上人工确认。

- **主动**：持续观察 Issue、Task、对话和工作流状态等信号，主动分析问题并决定是否出手，而不再等用户逐步下达指令。有价值再行动；不需要介入时保持安静。
- **调度**：按问题形态选择或组合工作流。对话目标还不稳时走 Chat；需要多路试探时走 Race（Pick First）；多条数据 / 多任务时走 Batch；有分支、汇合、循环和卡点时走 Flow；阶段产物明确时走 Pipeline。Chat 收敛出计划后，可在用户确认后再生成可执行 Workflow。
- **权限等效宿主用户**：具备平台中该用户角色的全部权限，数据访问与操作权限与宿主一致，可代表用户调度平台能力并推进任务。这是「与宿主同等」，不是超出该角色的全局权限。因为权限高，主动操作需要可审计、可追溯；关键决策仍可通过「用户确认」卡点落地。
- **和执行引擎的分工**：Super Agent 负责判断、选择和协调；执行引擎负责跑得住（暂停、恢复、取消、重试、幂等）。设计与监控对它可见，便于回答「跑到哪了、为什么停」，并在失败时主动介入。

## 要解决的核心问题

- 传统工作流强在确定性和可监控，弱在灵活推理；纯 Agent 强在应变，弱在可预期、可恢复、可审计。WD2A 要让两者能互相平替，而不是互相替换后丢失另一半能力。
- Chat / Race / Batch / Flow / Pipeline 若不做成统一抽象，每种模式都会各写一套执行器和状态模型，后续无法组合（例如 Flow 里嵌 Race，或 Chat 确认后生成 Pipeline）。
- 缺少执行与监控标准时，长任务崩溃不可恢复，失败不可定位，也无法做生产级 SLA。
- 「用户确认后再生成 / 执行 workflow」应是一等能力，而不是事后补的人工开关。
- Super Agent 权限与宿主等效后，主动行动的边界、审计和误操作回滚必须同时设计，否则「能替你做所有事」会变成不可控风险。

## 可选的开源参考方案

| 方向 | 可参考 | 说明 |
| --- | --- | --- |
| 确定性 / 长任务编排 | [Temporal](https://github.com/temporalio/temporal)、[Inngest](https://github.com/inngest/inngest)、[Restate](https://github.com/restatedev/restate) | 持久执行、重试、等待、崩溃恢复；Temporal 也有 LangGraph 插件可把 Agent 图跑在耐久工作流上 |
| Agent 工作流图 | [LangGraph](https://github.com/langchain-ai/langgraph) | 状态图、并行、中断与 checkpoint，适合 Flow / Pipeline / 人工确认 |
| 可视化与平替传统自动化 | [Dify](https://github.com/langgenius/dify)、[n8n](https://github.com/n8n-io/n8n) | 可视化编排、Chatflow / Workflow、批处理与集成；迁传统自动化时可对照 |
| 传统工作流基线 | Airflow、Prefect、Camunda 等 | 用来对照「传统工作流要被覆盖的能力清单」（调度、依赖、重试、审计），而不是直接当 Agent Runtime |

---

## 许可证

本项目采用 [MIT License](LICENSE) 开源。
