# Spark2Piece

<p align="center">
  <img src="assets/logo/spark2piece-logo.svg" width="220" alt="Spark2Piece">
</p>

<p align="center"><strong>From an idea to a complete, shippable product.</strong><br>从灵感到可交付成果。</p>

<p align="center">
  <a href="README.md">中文</a>
  ·
  <a href="LICENSE">MIT License</a>
  ·
  <a href="https://github.com/Spark2Piece">GitHub Organization</a>
</p>

Spark2Piece is meant to be an **Agent platform**, not another single-session chat agent.

The organization works along four tracks. This repository lands the foundations of the first two: Platform Use To Agent (PU2A) and Workflow Design To Agent (WD2A).

- [Organization tracks](#organization-tracks)
- [Platform Use To Agent (PU2A)](#platform-use-to-agent-pu2a)
- [Workflow Design To Agent (WD2A)](#workflow-design-to-agent-wd2a)
- [License](#license)

## Organization tracks

**Platform Use To Agent (PU2A)** addresses Agent abstraction (Agent Prompt and Skill) and the Agent runtime, as well as how Agents are invoked and how permissions work. Invocation covers standalone use, Group collaboration, and Agents calling one another as Subagents. Additional capabilities include Workspace hierarchy, Issue / automation, and Skill management and sharing.

**Workflow Design To Agent (WD2A)** addresses workflow abstraction and design, plus workflow execution and monitoring.

**Architecture Language To Agent (AL2A)** addresses how an Agent reads, parses, and executes the architecture design language of a mid-to-large project, instead of grepping the codebase on every turn.

**Memory Management To Agent (MM2A)** addresses Agent memory management, including performance, recall, and data privacy.

---

# Platform Use To Agent (PU2A)

PU2A separates *what an Agent is* from the runtime, so the same Runtime can run different kinds of Agents. A clear invocation protocol and permission model support standalone use, Group collaboration, Subagent calls, and ChildAgent. Issue / Task is the cross-session unit of work; Workspace provides hierarchical boundaries. Sandbox, external connectivity, and the Observatory are first-class platform capabilities, not afterthoughts.

## Foundations

### Agent configuration

Take “what this Agent is” out of the runtime. There are at least two layers:

- **Prompt**: role, goals, constraints, and output conventions
- **Skill**: a reusable capability pack (tool bindings, operating handbook, input/output contracts) that can be installed, shared, and versioned

### Runtime

The Runtime only runs things. Session loops, tool calls, event intake, concurrency, and failure recovery are not bound to any one Agent definition. The same Runtime can run different types of Agents.

### Agent Protocol

- **Standalone**: one Agent completes the task on its own
- **Group**: multiple Agents form a collaboration group with a single outward interface
- **Subagent calls**: Agents can invoke one another as subagents
- **ChildAgent**: a child instance derived from a parent Agent, with a lifecycle constrained by that parent

### Context carriers

Issue / Task is the unit of work: it carries the goal, status, artifacts, discussion, and follow-on events. Workspace provides hierarchical boundaries (personal / project / team).

### Additional capabilities

Workspace hierarchy, Issue / automation, and Skill management and sharing.

### Permissions

Data-access permissions (what can be seen) and action permissions (what can be changed, which tools can be called, and in whose identity other Agents may be invoked) are modeled separately, with least privilege and auditability.

## External dependencies

- **External platforms**: discover, pull, and reference external Skills / Agents / Workflows / Workspaces / Issues / Tasks.
- **Sandbox**: sandbox-as-a-service, ready to use. Tool and code execution are isolated by default, with an environment assigned per task or Agent. This covers Code Interpreter, Browser Use, and planned Computer / Mobile Use.
- **External connectivity**: keep IM, source-control platforms, email, and internal systems connected and available (reconnect, rate limiting, degradation, health checks).
- **Observatory**: analyze a Coding Agent’s execution quality, failure modes, and whether the engineering loop actually closed. Token usage and raw logs are inputs only.

## Ready-to-use Sandbox

Sandbox is PU2A’s default execution environment. The platform provides it as sandbox-as-a-service, so developers do not have to build, patch, or manage dependencies themselves. Agent code execution, browser automation, and later GUI operations all run in this isolated environment.

### Sandbox as a Service

- **Code Interpreter**: safely execute Python, Node.js, Java, and similar languages, with file and session management.
- **Browser Use**: a stable browser automation environment over CDP over WebSocket, compatible with Puppeteer / Playwright.
- **Computer / Mobile Use** (planned / extension): GUI operations, remote desktop, and other advanced actions.

### Enterprise isolation

- Multi-level isolation on secure containers (MicroVMs): request-level, instance-level, and session-level.
- Storage isolation, with mounts such as OSS / NAS.
- The sandbox environment is maintained and upgraded by the platform. Developers do not build, patch, or manage dependencies themselves.

### Serverless Sandbox performance

- **Shallow sleep, millisecond wake**: briefly idle instances enter shallow sleep and can wake in milliseconds when a request arrives, with almost no perceived cold start.
- **Deep sleep, second-level wake**: instances with no traffic for a longer period enter a more efficient deep sleep, and can still resume within seconds, balancing cost and performance.
- **Million-template concurrency**: the substrate can run sandbox templates (function-level) at million-scale concurrency, so many kinds of sandboxes can run on one platform at once—suited to multi-team, multi-line enterprise use.

## Agent Observatory

The Observatory is a first-class PU2A capability, at the same level as Agent abstraction, Runtime, and Sandbox. It consumes sessions, tasks, and timelines to answer three questions: did this Coding Agent do a good job, what kind of failure was it, and did the work go from Issue to a mergeable result. Tokens and raw logs are inputs only.

WD2A monitoring answers where a workflow has run and why it stopped. The Observatory answers coding-execution quality, failure modes, and whether the engineering loop closed.

- **What to watch**: Coding Agent sessions across providers (Claude Code, Codex, and others), subtasks, timelines, and execution status; queues that need attention (approvals, errors, unread items). Usage (tokens, models, cache, quotas) is kept, but as input, not as the conclusion.
- **Execution quality**: against the Issue / Task goal, is the result reviewable, or is it unfinished (code changed but no PR, CI failing, not merged, Issue left open).
- **Failure modes**: permission denied, tools spinning without progress, context exhausted, declaring done without verification, repeated retries, sub-agent context bleed, Sandbox failures, and similar—each should be attributable.
- **Engineering loop**: Issue → session → code change → review / CI → merge or close, with breakpoints marked.
- **Diagnosis and reports**: each session gets an outcome, timeline, diagnosis, and execution recap. Intelligent analysis and per-turn summaries are off by default and started by a person. Reports cite the timeline, artifacts, and status. Missing telemetry is marked unavailable, never recorded as zero. Usage stats keep only counts, model, working directory, and session identifiers; prompts and outputs are not treated as statistical raw material.
- **Status**: running, waiting for approval, failed, archived. Exceptions should be locatable, and can be retried or handed off.

## Core problems to solve

- When abstraction is coupled to the Runtime, Agents cannot be reused across environments or evolved on their own.
- When invocation is unclear, Group / Subagent / ChildAgent instances race for replies, bleed context, and amplify permissions.
- Without a stable Issue / Task carrier, people can only switch Sessions by hand, and continuity breaks at the session boundary.
- Once Agents touch real systems and data, permissions are no longer optional—they are the prerequisite for operating at scale.
- When Sandbox and external connectivity are weak, teams either withhold permissions or grant them and take on security and stability risk.
- A sandbox has to satisfy isolation strength, wake experience, and scale cost at the same time: request / instance / session isolation and storage isolation must be present by default; brief idle should wake in milliseconds, long idle should deep-sleep and resume in seconds. The platform maintains the environment so each team does not build, patch, and manage dependencies on its own.
- If Skills / Agents / Workflows cannot be discovered and shared, the platform collapses into a single-machine tool instead of a composable Agent platform.
- Token counts and provider-local logs alone cannot show execution quality, failure type, or where the engineering loop broke, and cannot inform how Runtime, permissions, and Skills should change.

## Optional open-source references

| Area | References | Notes |
| --- | --- | --- |
| Agent abstraction + Runtime | [LangGraph](https://github.com/langchain-ai/langgraph), [AutoGen / AG2](https://github.com/ag2ai/ag2), [CrewAI](https://github.com/crewAIInc/crewAI), [OpenHands](https://github.com/All-Hands-AI/OpenHands) | Harness / Runtime, graph scheduling, role collaboration, coding-agent sandbox runtimes |
| A2A invocation | [A2A Protocol](https://github.com/a2aproject/A2A) | Discovery, delegation, and inter-agent calls; MCP is more about an Agent calling tools |
| Tools and external connectivity | [MCP](https://modelcontextprotocol.io/) | Uniform access to external platforms and tools, to complete connectivity |
| Sandbox | [E2B](https://github.com/e2b-dev/e2b), [Daytona](https://github.com/daytonaio/daytona), isolation approaches such as OpenHands Docker / gVisor / Firecracker; for browser automation, CDP / [Playwright](https://github.com/microsoft/playwright) / Puppeteer | Isolated execution per task; sandbox-as-a-service, MicroVM multi-level isolation, and serverless sleep/wake can be extended from this class of systems |
| Skill management and sharing | Anthropic / community Agent Skill shapes, and Skill marketplace practice across platforms | Separating Prompt from reusable Skills, plus install and distribution |
| Context carriers | Issue / Task workflows (GitHub / Linear and similar) + event-driven sessions (for example DeepSeek Harness Super-agent practice) | Use Issue / Task / Thread as the cross-session carrier, instead of exposing Session to the user |
| Observatory | [Paseo Agent Observatory](https://paseo.sh/zh-CN/plugins/zhjiang22/agent-observatory) (zhjiang22/agent-observatory) | Outcome-first boards, timelines, subtasks, diagnosis, execution recap, unread and approvals, scheduled runs, local token stats, optional intelligent analysis. That plugin stops at session observation and recap; this platform also needs quality judgment, failure modes, and the engineering loop |

---

# Workflow Design To Agent (WD2A)

WD2A should fully cover traditional workflows and support replacing them with Agent-native execution. It abstracts patterns such as Chat, Race (Pick First), Batch, Flow, and Pipeline into unified objects, with a proactive Super Agent as the core scheduler: it analyzes and solves problems on its own, and holds all permissions of that user’s role on the platform (equivalent to the host user).

The point is not to invent another process engine. It is to turn these abstractions into first-class objects that an Agent can design, execute, and monitor.

## Foundations

- **Traditional workflow coverage**: existing approval flows, CI, batch jobs, human tasks, retries, timeouts, and compensation should move onto Agent-native execution, not onto a second, incompatible process.
- **Workflow abstractions**, at least:
  - **Chat**: progress through conversation, revising the plan as you go; suited to goals that are still unsettled
  - **Race (Pick First)**: try several paths in parallel; the first complete and usable result wins, and the rest are canceled or discarded
  - **Batch**: run the same steps over many records or tasks, concurrently or in waves, with partial failure supported
  - **Flow**: a directed process with branching, joins, loops, and human confirmation
  - **Pipeline**: advance by stages in order, with each stage’s artifacts becoming the next stage’s inputs
- **Super Agent as core scheduler**: proactively analyze and solve problems, and uniformly schedule the workflows above. Permissions are equivalent to the host user, with the full rights of that role on the platform.
- **Design**: visual or code-first orchestration, reusable subflows, parameterization, and versioning.
- **Execution**: start, pause, resume, cancel, retry, idempotency, and human checkpoints (continue only after confirmation).
- **Monitoring**: run status, current node, duration, failure reason, replay, and alerts—enough to answer “where did it get to, and why did it stop.”

## Super Agent

The Super Agent is WD2A’s proactive core scheduler, not just another execution Agent. The five workflow abstractions (Chat / Race / Batch / Flow / Pipeline) feed into it; it then asks the execution engine to start, resume, and retry, and inserts human confirmation when needed.

- **Proactive**: keep watching signals from Issues, Tasks, conversations, and workflow state; analyze the problem and decide whether to act, instead of waiting for step-by-step instructions. Act when it is worthwhile; stay quiet when intervention is not needed.
- **Scheduling**: choose or compose workflows by the shape of the problem. Use Chat when the conversational goal is still unsettled; Race (Pick First) when several paths should be tried; Batch for many records or tasks; Flow when there are branches, joins, loops, and checkpoints; Pipeline when stage artifacts are clear. After Chat converges on a plan, an executable Workflow can be generated once the user confirms.
- **Permissions equivalent to the host user**: the Super Agent has all permissions of that user’s role on the platform. Data-access and action permissions match the host, and it can schedule platform capabilities and advance work on the user’s behalf. This is “equal to the host,” not global rights beyond that role. Because the privilege is high, proactive actions must be auditable and traceable; critical decisions can still land through a “user confirmation” checkpoint.
- **Division of labor with the execution engine**: the Super Agent judges, selects, and coordinates; the execution engine keeps the run solid (pause, resume, cancel, retry, idempotency). Design and monitoring are visible to it, so it can answer “where did it get to, and why did it stop,” and step in when a run fails.

## Core problems to solve

- Traditional workflows are strong on determinism and observability, and weak on flexible reasoning. Pure Agents are strong on adaptation, and weak on predictability, recoverability, and audit. WD2A should let the two substitute for each other, not replace one another and lose the other half of the capability.
- If Chat / Race / Batch / Flow / Pipeline are not a unified abstraction, each pattern grows its own executor and state model, and later composition becomes impossible (for example, Race nested in a Flow, or a Pipeline generated after Chat confirmation).
- Without execution and monitoring standards, long-running work cannot recover from a crash, failures cannot be located, and production SLAs cannot be met.
- “Generate / execute the workflow only after user confirmation” should be a first-class capability, not a human switch added afterwards.
- Once Super Agent permissions are equivalent to the host, the bounds of proactive action, audit, and rollback of mistakes must be designed at the same time. Otherwise “it can do everything for you” becomes an uncontrolled risk.

## Optional open-source references

| Area | References | Notes |
| --- | --- | --- |
| Deterministic / long-running orchestration | [Temporal](https://github.com/temporalio/temporal), [Inngest](https://github.com/inngest/inngest), [Restate](https://github.com/restatedev/restate) | Durable execution, retries, waits, and crash recovery; Temporal also has a LangGraph plugin for running Agent graphs on durable workflows |
| Agent workflow graphs | [LangGraph](https://github.com/langchain-ai/langgraph) | State graphs, parallelism, interrupts, and checkpoints—suited to Flow / Pipeline / human confirmation |
| Visual orchestration and replacing traditional automation | [Dify](https://github.com/langgenius/dify), [n8n](https://github.com/n8n-io/n8n) | Visual orchestration, Chatflow / Workflow, batching, and integrations; a useful comparison when migrating traditional automation |
| Traditional workflow baseline | Airflow, Prefect, Camunda, and similar | Use these to check the capability list traditional workflows must still cover (scheduling, dependencies, retries, audit)—not as an Agent Runtime directly |

---

## License

This project is released under the [MIT License](LICENSE).
