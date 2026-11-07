# Smart Agent

## Product overview

Smart Agent is a task-orchestration CLI built on top of the pi runtime. It can run a small task directly, or let a Manager design a workflow before Workers execute it. The product records decisions, tool activity, checkpoints, feedback, and final evidence so a long task can be resumed and reviewed.

Its main capabilities are workflow design, four execution shapes (Instant, Vertical, Horizontal, and Hybrid), a Spiral Engine for method evolution, Cruising Mode for pause and resume, optional sub-agent dispatch, and pi-native model and tool integration.

## The problem it solves

A direct Agent Loop is useful for a short request but becomes unreliable when work has dependencies, parallel branches, failed attempts, or a long pause between sessions. Important decisions disappear into chat history, completed branches are repeated, and a failed provider call can be confused with a failed task.

Smart Agent separates planning from execution, persists the method and graph state, keeps evidence by run, and exposes the difference between a task result, a provider error, and an incomplete run.

## Usage guide

Requirements are Node.js 22 or newer and a pi-supported provider.

```bash
npm install
node smart-agent.mjs --help
node smart-agent.mjs run --traditional --goal "Summarize the project notes" --workspace ./work
node smart-agent.mjs run --goal "Inspect the project and repair the issue" --workspace ./work
node smart-agent.mjs cruise start --goal "Continue a long task" --workspace ./work
node smart-agent.mjs cruise status <job-id>
```

Configure the provider, model, thinking level, and credential environment through Smart Agent config or pi's native configuration. Use `--attach` for text or image inputs. Run `smart-agent config doctor` before a first model call.

## Notes

Smart Agent controls task organization. The selected provider still determines model quality, context limits, tool availability, cost, and authentication. Spiral Engine stores method versions, feedback, promotion, and rollback for this product's runs. It does not claim that a model has acquired general intelligence. Traditional mode is the shortest path for a clear, low-risk task.

## Technical implementation

The CLI is implemented as Node.js ES modules. `src/cli.mjs` parses commands and config, `src/run.mjs` handles direct runs, `src/smart-run.mjs` coordinates Manager and Worker execution, and `src/cruise.mjs` manages resumable jobs. `core/` contains prompt lifecycle, graph orchestration, method contracts, extension contracts, and `SpiralStore`.

The Spiral Engine persists event history, feedback, strategy versions, promotion decisions, and checkpoints under the user configuration directory. `mcp/` provides the MCP-facing entry point. `tests/` and `fixtures/` cover runtime, control, comparison, cruise, and S-card behavior. Provider credentials are referenced through environment variables or pi authentication storage and are redacted from run evidence.

---

# Smart Agent

## 产品介绍

Smart Agent 是建立在 pi 运行时之上的任务编排 CLI。简单任务可以直接执行，需要依赖、并行、恢复或验收的任务先由 Manager 设计工作流，再由 Worker 执行。产品会记录设计、工具活动、checkpoint、反馈和最终证据，使长任务可以恢复和复查。

它的核心能力包括工作流设计、Instant/Vertical/Horizontal/Hybrid 四种执行形态、用于方法演化的 Spiral Engine、支持暂停恢复的 Cruising Mode、可选子智能体派发，以及 pi 原生模型和工具接入。

## 解决的痛点

普通 Agent Loop 适合短任务，但在任务存在依赖、并行分支、失败尝试或跨会话恢复时，关键决策容易丢失，完成分支可能被重复执行，供应商错误也可能被误认为任务失败。

Smart Agent 将计划和执行分开，保存方法与图状态，按运行记录证据，并区分任务结果、供应商错误和未完成运行。

## 使用指南

需要 Node.js 22 或更高版本，并准备 pi 支持的模型供应商。

```bash
npm install
node smart-agent.mjs --help
node smart-agent.mjs run --traditional --goal "总结项目说明" --workspace ./work
node smart-agent.mjs run --goal "检查项目并修复问题" --workspace ./work
node smart-agent.mjs cruise start --goal "持续处理一个长任务" --workspace ./work
node smart-agent.mjs cruise status <job-id>
```

通过 Smart Agent 配置或 pi 原生配置设置供应商、模型、思考档和凭证环境变量。文本或图片可以用 `--attach` 传入。首次调用模型前可以执行 `smart-agent config doctor`。

## 注意事项

Smart Agent 负责任务组织，模型质量、上下文、工具、费用和认证仍由所选供应商决定。Spiral Engine 保存本产品运行中的方法版本、反馈、晋升和回滚信息，不代表模型获得了通用智能。明确且低风险的任务可以直接使用 Traditional 模式。

## 技术实现

CLI 使用 Node.js ES modules。`src/cli.mjs` 处理命令和配置，`src/run.mjs` 处理直接运行，`src/smart-run.mjs` 协调 Manager 与 Worker，`src/cruise.mjs` 管理可恢复任务。`core/` 包含提示生命周期、图编排、方法契约、扩展契约和 `SpiralStore`。

Spiral Engine 将事件历史、反馈、策略版本、晋升决定和 checkpoint 保存在用户配置目录。`mcp/` 提供 MCP 入口，`tests/` 和 `fixtures/` 覆盖运行时、控制、对比、巡航和 S-card 行为。供应商凭证通过环境变量或 pi 认证存储引用，并在运行证据中脱敏。
