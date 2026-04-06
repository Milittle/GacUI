# AI 协作方法论新项目初始化模板包

## 目的

这份文档提供一套最小初始化模板，用于在新项目里快速落地仓库级 AI 协作机制。
目标不是复制一个成熟系统，而是先搭出最小闭环。

## 最小目录

建议先创建如下目录结构：

```text
your-repo/
├─ AGENTS.md
├─ .github/
│  ├─ copilot-instructions.md
│  ├─ prompts/
│  │  ├─ code.prompt.md
│  │  ├─ 2-planning.prompt.md
│  │  ├─ 4-execution.prompt.md
│  │  └─ 5-verifying.prompt.md
│  ├─ KnowledgeBase/
│  │  ├─ Index.md
│  │  ├─ KB_Project_Architecture.md
│  │  └─ KB_Project_APIs.md
│  └─ TaskLogs/
│     ├─ Copilot_Task.md
│     ├─ Copilot_Planning.md
│     └─ Copilot_Execution.md
```

## 初始化顺序

建议按下面顺序完成：

1. 先写 `AGENTS.md`
2. 再写 `copilot-instructions.md`
3. 再写默认 `code.prompt.md`
4. 再加 `planning` 与 `execution` prompt
5. 最后补知识库和 TaskLogs

## 模板文件 1：`AGENTS.md`

这个文件负责请求分流。
下面是一份最小模板：

```md
# AGENTS Instructions

- Read `REPO-ROOT/.github/copilot-instructions.md` before doing any work.

## Request Routing

- If the first word of the latest user message is `plan`, follow `REPO-ROOT/.github/prompts/2-planning.prompt.md`.
- If the first word of the latest user message is `execute`, follow `REPO-ROOT/.github/prompts/4-execution.prompt.md`.
- If the first word of the latest user message is `verify`, follow `REPO-ROOT/.github/prompts/5-verifying.prompt.md`.
- Otherwise, follow `REPO-ROOT/.github/prompts/code.prompt.md`.

## Notes

- Treat the remaining user message as the actual request body.
- Start working immediately after applying the routing rule.
```

这个版本非常小，但已经足够建立工作流入口。

## 模板文件 2：`.github/copilot-instructions.md`

这个文件负责仓库级规则。
下面是一份最小模板：

```md
# General Instructions

- `REPO-ROOT` means the root directory of the repository.

## Before Working

- Read `REPO-ROOT/.github/KnowledgeBase/Index.md` before making design or coding decisions.
- Respect generated files and protected folders.

## Allowed Changes

- You may modify source files under `src/` and `tests/`.
- Do not modify generated files under `generated/`.
- Do not modify third-party code under `vendor/`.

## Build and Test

- Build command: `<PUT YOUR BUILD COMMAND HERE>`
- Test command: `<PUT YOUR TEST COMMAND HERE>`
- After code changes, build and test are required.

## Working Rules

- Do not invent project rules.
- Prefer existing patterns in the codebase.
- If a change touches architecture decisions, consult the knowledge base first.
```

这里最重要的是把 build/test 和禁改目录写清楚。

## 模板文件 3：`.github/prompts/code.prompt.md`

这是默认编码流程。
最小模板如下：

```md
# Code Task

## Goal

- Follow the latest user request.
- Implement the requested change.
- Ensure the code builds.
- Ensure tests pass.

## Workflow

1. Read repository instructions.
2. Read relevant knowledge base documents.
3. Inspect related code.
4. Implement the change.
5. Build the project.
6. Run tests.
7. Fix failures until the task is complete.
```

这个模板不要写得过长。
重点是强制流程顺序。

## 模板文件 4：`.github/prompts/2-planning.prompt.md`

这是先写计划的模板。

```md
# Planning Task

## Goal

- Produce a planning document in `REPO-ROOT/.github/TaskLogs/Copilot_Planning.md`.
- Do not change source code in this step.

## Workflow

1. Read repository instructions.
2. Read relevant knowledge base documents.
3. Understand the user request.
4. Identify affected modules and files.
5. Write a step-by-step implementation plan.
6. List build and test actions needed for verification.
```

这个 Prompt 的重点是禁止在 planning 阶段直接改代码。

## 模板文件 5：`.github/prompts/4-execution.prompt.md`

这是执行模板。

```md
# Execution Task

## Goal

- Execute the approved plan in `REPO-ROOT/.github/TaskLogs/Copilot_Execution.md`.

## Workflow

1. Read repository instructions.
2. Read relevant knowledge base documents.
3. Apply the planned changes to source code.
4. Record progress in the execution document.
5. Build the project.
6. Fix compile errors.
7. Run tests.
8. Fix test failures.
```

## 模板文件 6：`.github/prompts/5-verifying.prompt.md`

这是验证模板。

```md
# Verification Task

## Goal

- Verify that the requested change is complete and correct.

## Workflow

1. Read repository instructions.
2. Check what files were changed.
3. Build the project.
4. Run relevant tests.
5. Report whether the task is verified, partially verified, or failed verification.
```

## 模板文件 7：`.github/KnowledgeBase/Index.md`

这是知识库入口。

```md
# Knowledge Base

## Project Overview

- This project provides `<ONE-SENTENCE DESCRIPTION>`.

## Module Map

- `src/core`: core business logic
- `src/api`: external API layer
- `src/ui`: user interface
- `tests/`: test suites

## Rules

- Read architecture notes before changing cross-module behavior.
- Read API notes before introducing new utility layers.

## Documents

- [Architecture](./KB_Project_Architecture.md)
- [API Choices](./KB_Project_APIs.md)
```

## 模板文件 8：`.github/KnowledgeBase/KB_Project_Architecture.md`

```md
# Project Architecture

## Entry Points

- Describe the main runtime entry points here.

## Main Components

- Describe each major component and its responsibility.

## Important Flows

- Describe the main cross-module flows here.

## Boundaries

- Document what should not be bypassed directly.
```

## 模板文件 9：`.github/KnowledgeBase/KB_Project_APIs.md`

```md
# Project API Choices

## Common Tasks

- Use `<MODULE OR CLASS>` for `<COMMON TASK>`.
- Use `<MODULE OR CLASS>` for `<COMMON TASK>`.

## Avoid

- Do not call `<LOW-LEVEL MODULE>` directly from `<HIGH-LEVEL AREA>`.
- Do not duplicate logic already provided by `<SHARED MODULE>`.
```

## 模板文件 10：TaskLogs

这三个文件先放最小头部即可：

### `Copilot_Task.md`

```md
# !!!TASK!!!
```

### `Copilot_Planning.md`

```md
# !!!PLANNING!!!
```

### `Copilot_Execution.md`

```md
# !!!EXECUTION!!!
```

## 第一周建议

新项目落地时，第一周只做这些事情：

### 第 1 天

- 写 `AGENTS.md`
- 写 `copilot-instructions.md`
- 写 `code.prompt.md`

### 第 2 天

- 写 `2-planning.prompt.md`
- 写 `4-execution.prompt.md`
- 建 TaskLogs

### 第 3 天

- 写知识库首页
- 补一篇架构说明
- 补一篇 API 选型说明

### 第 4 到 5 天

- 用真实任务试跑
- 记录 AI 失败点
- 回补 Prompt 和知识库

## 试跑建议

第一次试跑时，选下面这类任务：

- 影响范围中等
- 需要读一点项目知识
- 但不涉及特别危险的架构重构

例如：

- 加一个中等复杂度接口
- 补一组测试
- 做一次有限范围的模块重构

不要一上来就用这套机制做最大最复杂的任务。

## 最小成功标准

如果一个新项目做到了下面这些，就算第一版成功：

- AI 会先读仓库规则
- AI 会根据请求类型走不同流程
- AI 会查询知识库而不是纯猜
- AI 会把中间结果写到任务文档
- AI 会在改代码后做构建和测试

## 后续扩展方向

等第一版稳定后，再考虑：

- review prompt
- investigate prompt
- knowledge-base drafting prompt
- TaskLogs 初始化与备份脚本
- 结构化 prompt 变量和条件系统

## 最后的建议

新项目初始化时，不要追求“像成熟体系一样完整”。
真正重要的是先搭出一个足够小、但能约束行为的版本。

先让 AI 的行为稳定下来，再逐步增强功能。
