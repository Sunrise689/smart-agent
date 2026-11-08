# Smart Agent

## 1. Product introduction

Smart Agent is a Node.js CLI and MCP-facing runtime for turning a goal into a durable workflow. A short, low-risk request can run through the normal pi Agent Loop. A task with dependencies, parallel branches, retries, or a long pause is first designed by a Manager and then executed by Worker nodes. Every run keeps its graph, node receipts, tool evidence, checkpoints, and terminal state so that a result can be inspected or resumed.

The runtime supports four execution shapes: **Instant** for one direct node, **Vertical** for a dependency chain, **Horizontal** for independent branches, and **Hybrid** for a graph that combines both. It also includes the Spiral Engine for recording feedback and strategy versions, Cruising Mode for pausing and waking long jobs, optional sub-agent dispatch, pi-native model/tool access, and text or image attachments.

## 2. Problems it solves

An ordinary agent loop is easy to start but becomes difficult to operate when a task has multiple pieces. A failed provider request can look like a failed task, a completed branch can be repeated after a restart, and important decisions can disappear in chat history. Teams also need to know which worker produced a result and whether the result has evidence behind it.

Smart Agent separates workflow design from execution, persists the graph and method state, skips completed branches after recovery, and records failure as an explicit state. The output is therefore a run with evidence and checkpoints, rather than an untraceable final paragraph.

## 3. Usage guide

Requirements: Node.js 22 or newer and a provider supported by pi. From the product directory:

```bash
npm install
npm run doctor
smart-agent --help
smart-agent run --traditional --goal "Summarize the project notes" --workspace ./work
smart-agent run --goal "Inspect the project and repair the issue" --workspace ./work
smart-agent cruise start --goal "Continue a long task" --workspace ./work
smart-agent cruise status <job-id>
smart-agent cruise pause <job-id>
smart-agent cruise resume <job-id>
```

`smart-agent doctor` checks the Node runtime, pi root, configuration path, provider and credential references, Spiral state directory, and sub-agent switch. Configure the provider, model, reasoning level, and credential environment through Smart Agent configuration or pi's native configuration. Attach text or images with repeated `--attach <file>` options. Enable the optional dispatcher with `smart-agent plugins enable sub-agents`; disable it when a direct run is sufficient.

For the MCP entry point, register `mcp/smart_agent_mcp_server.py` as a stdio server. Its tools expose S-card run, Cruise start, status, pause, resume, wake, and stop operations. The same run and job identifiers are used by the CLI and MCP paths.

## 4. Notes and boundaries

Smart Agent organizes work; the selected provider still determines model quality, context limits, tool availability, cost, and authentication. A successful process exit is not the same as a successful task: inspect the node receipt, evidence, and final status. Traditional mode is the shortest path for a clear task and deliberately avoids unnecessary Manager/Worker overhead.

Spiral state contains run metadata, feedback, strategy versions, promotion decisions, rollback events, and checkpoints. It does not grant a model general intelligence. Credentials are referenced through environment variables or pi authentication storage and are not written into run evidence. A Cruise job must have a valid goal, workspace, and provider reference before it can be resumed.

## 5. Technical implementation

The CLI is implemented as Node.js ES modules. `src/cli.mjs` parses commands and configuration; `src/run.mjs` handles direct runs; `src/smart-run.mjs` coordinates Manager design and Worker execution; `src/executors.mjs` runs pi nodes; and `src/cruise.mjs` implements pause, resume, wake, stop, and checkpoint recovery. `core/orchestration.mjs` stores graph design, dependency state, attempts, receipts, and checkpoint snapshots.

`core/spiral-store.mjs` is the event-backed Spiral Engine. It records events such as feedback, strategy version creation, promotion, rollback, checkpoint save, and checkpoint clear. `mcp/smart_agent_mcp_server.py` is a standard-library JSON-RPC MCP surface for S-card and Cruise lifecycle operations. `tests/` and `fixtures/` cover runtime, graph control, comparison, Cruise, and S-card behavior. Dispatch receipts use the `smart-agent-dispatch-receipt-v1` schema and include the requested and expected model, profile, CLI, effort, write scope, budget, exit code, and success flag.

---

# Smart Agent

## 1. 产品介绍

Smart Agent 是一个 Node.js CLI 和 MCP 运行时，用来把一个目标变成可持续执行的工作流。短小、低风险的任务可以直接走 pi 的 Agent Loop；有依赖、并行分支、重试或长时间暂停的任务，会先由 Manager 设计工作流，再由 Worker 节点执行。每次运行都会保存图结构、节点回执、工具证据、checkpoint 和终态，便于检查与恢复。

它支持四种执行形态：适合单节点的 **Instant**、依赖链式执行的 **Vertical**、独立分支并行的 **Horizontal**，以及混合依赖关系的 **Hybrid**。产品还包含记录反馈和策略版本的 Spiral Engine、可暂停和唤醒长任务的 Cruising Mode、可选子智能体派发、pi 原生模型与工具接入，以及文本或图片附件。

## 2. 解决的痛点

普通 Agent Loop 启动简单，但遇到多步骤任务就容易失控：供应商请求失败可能被误判为任务失败，重启后已经完成的分支可能重复执行，关键决策也容易消失在聊天记录里。使用者还需要知道结果由哪个 Worker 产生，以及结果有没有对应证据。

Smart Agent 将工作流设计和执行分开，保存图状态与方法状态，恢复时跳过已完成分支，并把失败写成明确状态。因此交付的是带证据和 checkpoint 的运行结果，而不是无法追溯的最终文字。

## 3. 使用指南

需要 Node.js 22 或更高版本，并在 pi 中配置可用模型供应商。进入产品目录后执行：

```bash
npm install
npm run doctor
smart-agent --help
smart-agent run --traditional --goal "总结项目说明" --workspace ./work
smart-agent run --goal "检查项目并修复问题" --workspace ./work
smart-agent cruise start --goal "持续处理一个长任务" --workspace ./work
smart-agent cruise status <job-id>
smart-agent cruise pause <job-id>
smart-agent cruise resume <job-id>
```

`smart-agent doctor` 会检查 Node 运行时、pi 根目录、配置路径、供应商和凭证引用、Spiral 状态目录以及子智能体开关。供应商、模型、思考档和凭证环境变量可以通过 Smart Agent 配置或 pi 原生配置设置。文本或图片使用重复的 `--attach <file>` 传入。需要派发能力时执行 `smart-agent plugins enable sub-agents`，简单任务可以保持关闭。

需要接入 MCP 时，把 `mcp/smart_agent_mcp_server.py` 注册为 stdio 服务。它提供 S-card 运行、Cruise 启动、状态查看、暂停、恢复、唤醒和停止工具。CLI 与 MCP 使用同一套运行 ID 和任务 ID。

## 4. 注意事项

Smart Agent 负责组织任务，模型质量、上下文、工具、费用和认证仍由所选供应商决定。进程正常退出不等于任务完成，必须同时检查节点回执、证据和最终状态。目标明确的简单任务可以使用 Traditional 模式，避免引入不必要的 Manager/Worker 开销。

Spiral 状态保存运行元数据、反馈、策略版本、晋升决定、回滚事件和 checkpoint，不代表模型获得了通用智能。凭证只通过环境变量或 pi 的认证存储引用，不写入运行证据。Cruise 只有在目标、工作区和供应商引用完整时才能恢复。

## 5. 技术实现

CLI 使用 Node.js ES modules。 `src/cli.mjs` 解析命令和配置，`src/run.mjs` 处理直接运行，`src/smart-run.mjs` 协调 Manager 设计和 Worker 执行，`src/executors.mjs` 运行 pi 节点，`src/cruise.mjs` 实现暂停、恢复、唤醒、停止和 checkpoint 恢复。 `core/orchestration.mjs` 保存图设计、依赖状态、尝试次数、回执和 checkpoint 快照。

`core/spiral-store.mjs` 是事件驱动的 Spiral Engine，记录反馈、策略版本创建、晋升、回滚、checkpoint 保存和清理等事件。 `mcp/smart_agent_mcp_server.py` 使用标准库实现 S-card 与 Cruise 生命周期的 JSON-RPC MCP 接口。 `tests/` 和 `fixtures/` 覆盖运行时、图控制、对比、Cruise 和 S-card 行为。派卡回执使用 `smart-agent-dispatch-receipt-v1`，记录请求模型、预期模型、profile、CLI、推理档、写入范围、预算、退出码和成功标志。




