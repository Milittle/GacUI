# 单仓库项目 AI 协作流程最小落地手册

## 目的

这份文档给出一套可直接套用到现有单仓库项目的操作手册。
目标不是复制本仓库的 `.github/Agent` 实现，而是提取它背后真正高价值的方法，并用现成的 code agent 来驱动：

- Claude Code
- OpenCode
- Codex
- 其他具备读写文件、运行命令能力的代码代理

这套手册适合这样的项目：

- 已经有代码仓库
- 已经有基本构建方式
- 已经有测试入口，哪怕还不完整
- 你希望让 AI 的工作方式可预测、可恢复、可迭代

## 核心原则

先记住一句话：

`不要依赖一个会聊天的 AI，要依赖一套由文档、脚本和状态文件约束的工作流。`

这套方法最重要的不是 Portal、SDK 或多模型调度，而是下面五件事：

1. 请求必须路由到固定阶段
2. 每个阶段必须读写固定文件
3. 构建与测试必须走稳定脚本入口
4. 成功必须依赖外部证据，不依赖 AI 自述
5. 每个阶段都必须支持后续迭代更新

## 最小目录结构

建议先在你的项目里补出下面这组目录和文件：

```text
your-repo/
├─ AGENTS.md
├─ .github/
│  ├─ copilot-instructions.md
│  ├─ prompts/
│  │  ├─ 0-scrum.prompt.md
│  │  ├─ 1-design.prompt.md
│  │  ├─ 2-planning.prompt.md
│  │  ├─ 4-execution.prompt.md
│  │  ├─ 5-verifying.prompt.md
│  │  └─ review.prompt.md
│  ├─ KnowledgeBase/
│  │  ├─ Index.md
│  │  ├─ KB_Project_Architecture.md
│  │  └─ KB_Project_APIs.md
│  └─ TaskLogs/
│     ├─ Copilot_Scrum.md
│     ├─ Copilot_Task.md
│     ├─ Copilot_Planning.md
│     ├─ Copilot_Execution.md
│     ├─ Copilot_KB.md
│     └─ Copilot_Investigate.md
└─ scripts/
   ├─ prepare.sh
   ├─ build.sh
   ├─ test.sh
   └─ run.sh
```

如果你现在不想一下子建这么多，最低配可以只保留：

- `AGENTS.md`
- `.github/copilot-instructions.md`
- `.github/prompts/`
- `.github/TaskLogs/`
- `scripts/build.sh`
- `scripts/test.sh`

## 这套流程的阶段

建议先固定成下面六类阶段：

- `scrum`
- `design`
- `plan`
- `execute`
- `verify`
- `review`

你可以把它理解成一条标准链路：

`问题澄清 -> 方案设计 -> 实施计划 -> 修改代码 -> 验证结果 -> 复查质量`

### 各阶段职责

#### `scrum`

用来收敛问题定义、背景、约束、补充上下文。

产物建议写入：

- `Copilot_Scrum.md`

#### `design`

用来定义方案，不直接改代码。

产物建议写入：

- `Copilot_Task.md`

#### `plan`

用来把设计拆成执行步骤和测试步骤，不直接改代码。

产物建议写入：

- `Copilot_Planning.md`

#### `execute`

用来按设计和计划改代码、修复问题、记录执行过程。

主要依赖：

- `Copilot_Task.md`
- `Copilot_Planning.md`

过程记录建议写入：

- `Copilot_Execution.md`

#### `verify`

用来构建、测试、检查改动是否真的完成。

主要依赖：

- `Copilot_Execution.md`
- build/test 日志

#### `review`

用来做额外质量复查。没有多 agent 平台时，这一步也可以由另一个独立 agent 或你自己来执行。

## 为什么必须要有 `TaskLogs`

`TaskLogs` 是这套流程的核心，不是附属物。

它承担三件事：

1. 把 AI 的短期记忆外部化
2. 让不同阶段可以衔接，而不是每轮重讲一遍
3. 让流程可以恢复、检查、复盘

这意味着：

- 不要只在聊天里说清需求，要写入文档
- 不要只在脑子里记住计划，要写入文档
- 不要只在终端里修了几轮 bug 就结束，要写入文档

没有 `TaskLogs`，你的流程会退化成普通问答。

## `TaskLogs` 的建议分工

### `Copilot_Scrum.md`

记录：

- 问题背景
- 当前目标
- 范围边界
- 特殊约束
- 新增补充信息

### `Copilot_Task.md`

记录：

- 设计方案
- 影响范围
- 模块边界
- 风险点
- 为什么这样做

### `Copilot_Planning.md`

记录：

- 拟改哪些模块和文件
- 实施顺序
- 验证顺序
- 回归测试项

### `Copilot_Execution.md`

记录：

- 当前执行步骤
- 已完成项
- 编译修复尝试
- 测试修复尝试
- 遇到的问题和处理

### `Copilot_KB.md`

记录当前任务中发现的、值得沉淀的知识。
不是每次都必须写，但当你发现“以后还会反复遇到”的信息时，应该沉淀进去。

### `Copilot_Investigate.md`

用于调查类问题，例如：

- 非确定性 bug
- 构建异常
- 运行时不稳定
- 未知崩溃原因

## 现有 code agent 如何驱动这个流程

没有 `.github/Agent` 调度器时，最简单的方式是把 agent 当作“严格遵守仓库工作流的执行者”。

核心做法如下：

1. 每轮开始时，让 agent 先读 `AGENTS.md`
2. 再读 `.github/copilot-instructions.md`
3. 根据请求首词，读对应的 prompt
4. 再读相关 `TaskLogs`
5. 按 prompt 规定推进该阶段
6. 通过 `scripts/build.sh` 和 `scripts/test.sh` 做验证
7. 把结果写回 `TaskLogs`

也就是说，即使没有 Portal，流程仍然是：

`请求 -> 阶段路由 -> 读规则 -> 读状态文件 -> 执行 -> 验证 -> 写回状态文件`

### 对 Claude Code / OpenCode / Codex 的统一要求

无论你用哪个 agent，都建议强制以下规则：

- 开始任何工作前先读 `AGENTS.md`
- 任何设计和编码决策前先读知识库
- 禁止绕过 `scripts/build.sh` 和 `scripts/test.sh`
- 禁止把“我觉得完成了”当作完成标准
- 必须把阶段结果写入对应 `TaskLogs`

## 推荐的仓库级文件

### `AGENTS.md`

这是入口文件，职责是请求路由，不是承载所有项目知识。

建议至少包含：

- 先读哪些文件
- 按首词路由到哪个 prompt
- 如果不是已知首词，默认走哪个 prompt
- 如何处理剩余文本

最小模板：

```md
# AGENTS Instructions

- Read `REPO-ROOT/.github/copilot-instructions.md` before doing any work.

## Request Routing

- If the first word is `scrum`, follow `REPO-ROOT/.github/prompts/0-scrum.prompt.md`.
- If the first word is `design`, follow `REPO-ROOT/.github/prompts/1-design.prompt.md`.
- If the first word is `plan`, follow `REPO-ROOT/.github/prompts/2-planning.prompt.md`.
- If the first word is `execute`, follow `REPO-ROOT/.github/prompts/4-execution.prompt.md`.
- If the first word is `verify`, follow `REPO-ROOT/.github/prompts/5-verifying.prompt.md`.
- If the first word is `review`, follow `REPO-ROOT/.github/prompts/review.prompt.md`.
- Otherwise, follow `REPO-ROOT/.github/prompts/code.prompt.md`.

## Notes

- Treat the rest of the user message as the actual request body.
- Start working immediately after applying the routing rule.
```

### `.github/copilot-instructions.md`

这是仓库全局规则。

建议至少写清：

- 哪些目录允许改
- 哪些目录禁止改
- 构建命令是什么
- 测试命令是什么
- 哪些文档必须先读
- 何时必须查知识库
- 何时必须跑测试

最小模板：

```md
# General Instructions

- `REPO-ROOT` means the root directory of the repository.

## Before Working

- Read `REPO-ROOT/.github/KnowledgeBase/Index.md` before making design or coding decisions.

## Allowed Changes

- You may modify source files under `src/` and `tests/`.
- Do not modify generated files under `generated/`.
- Do not modify third-party files under `vendor/`.

## Build and Test

- Build command: `REPO-ROOT/scripts/build.sh`
- Test command: `REPO-ROOT/scripts/test.sh`
- After code changes, both build and test are required.

## Working Rules

- Do not invent repository rules.
- Prefer existing project patterns.
- Use task log documents as working state, not chat memory only.
```

## 推荐的 Prompt 设计

### `0-scrum.prompt.md`

职责：

- 澄清问题
- 补充背景
- 收敛范围
- 写 `Copilot_Scrum.md`

### `1-design.prompt.md`

职责：

- 输出设计方案
- 识别影响范围
- 不改代码
- 写 `Copilot_Task.md`

### `2-planning.prompt.md`

职责：

- 把设计拆成执行步骤和验证步骤
- 不改代码
- 写 `Copilot_Planning.md`

### `4-execution.prompt.md`

职责：

- 按计划改代码
- 写执行过程
- 构建并修复编译问题
- 运行测试并修复测试问题
- 更新 `Copilot_Execution.md`

### `5-verifying.prompt.md`

职责：

- 独立验证改动是否完成
- 检查 build 与 test
- 记录验证结论

## 脚本入口必须怎么设计

你真正要控制的不是 AI 说什么，而是 AI 怎么运行项目。

建议至少准备这几个脚本：

### `scripts/prepare.sh`

作用：

- 初始化 `TaskLogs`
- 清理旧的临时日志
- 为新任务准备状态文件

### `scripts/build.sh`

作用：

- 用唯一允许的方式构建项目
- 输出标准日志

### `scripts/test.sh`

作用：

- 用唯一允许的方式执行测试
- 输出标准日志

### `scripts/run.sh`

可选。
如果项目有运行入口，建议也统一脚本化。

## 一轮标准流程怎么跑

假设你要让现有 agent 完成一个功能或 bugfix，推荐顺序如下：

1. `scrum <问题描述>`
2. `design`
3. `plan`
4. `execute`
5. `verify`
6. `review`

### 每一步应做什么

#### 第一步：`scrum`

目的：

- 把需求说清楚
- 把边界说清楚
- 把额外约束记下来

结果：

- `Copilot_Scrum.md` 成为后续阶段的输入

#### 第二步：`design`

目的：

- 明确方案
- 明确影响范围
- 明确为何这样做

结果：

- `Copilot_Task.md` 成为后续计划和执行的设计依据

#### 第三步：`plan`

目的：

- 细化实施步骤
- 细化验证步骤

结果：

- `Copilot_Planning.md` 成为执行阶段的任务清单

#### 第四步：`execute`

目的：

- 实际改代码
- 修复 build / test 问题
- 记录执行过程

结果：

- 代码完成修改
- `Copilot_Execution.md` 记录了执行和修复轨迹

#### 第五步：`verify`

目的：

- 独立确认任务是否完成
- build 是否通过
- test 是否通过
- 是否还有明显遗漏

结果：

- 给出明确的验证结论

#### 第六步：`review`

目的：

- 做第二视角的质量检查
- 看行为回归、边界缺陷、遗漏测试

结果：

- 发现 execute / verify 阶段可能漏掉的问题

## 某个阶段需要反复迭代时怎么办

这套方法必须支持 `update`。

也就是说，每个阶段都应支持两种模式：

- 第一次创建
- 后续更新

例如：

- `design 新增导出功能`
- `design update 增加分页约束和错误处理要求`
- `execute`
- `execute update 修复导出结果顺序不稳定的问题`
- `verify`
- `verify update 增加空输入与超长输入的验证`

### 正确的迭代原则

如果上游阶段发生变化，下游阶段通常也应该刷新。

例如：

- 设计变了，计划要更新
- 计划变了，执行要更新
- 执行变了，验证要重跑

所以一条合理的刷新链通常是：

`design update -> plan update -> execute update -> verify update`

### 不要怎么做

不要在设计已经变化后，仍然拿旧计划直接继续执行。

不要在执行已经改动后，只凭感觉说“应该没问题”，而不重新验证。

## 没有多 agent 平台时，怎么做 `review`

即使没有 `.github/Agent` 的多模型调度，你仍然可以保留 review 步骤。

有三种简单做法：

### 方案一：同一个 agent 二次审视

先 `execute` 与 `verify`，然后单独发 `review` 请求，明确要求它用 code review 视角重查：

- 逻辑错误
- 行为回归
- 边界条件
- 缺失测试

### 方案二：第二个 agent 独立 review

例如：

- Claude Code 负责 `design/plan/execute`
- Codex 或 OpenCode 负责 `review`

或者反过来。

### 方案三：人工 review

如果暂时只有一个 agent，也可以由你自己根据 `Copilot_Execution.md` 和最终 diff 做一次人工复查。

## 第一周落地顺序

如果你准备在一个现有单仓库项目里落地，建议按下面顺序推进。

### 第一天

创建：

- `AGENTS.md`
- `.github/copilot-instructions.md`
- `scripts/build.sh`
- `scripts/test.sh`

目标：

- 先把规则和构建验证入口固定下来

### 第二天

创建：

- `.github/prompts/0-scrum.prompt.md`
- `.github/prompts/1-design.prompt.md`
- `.github/prompts/2-planning.prompt.md`
- `.github/prompts/4-execution.prompt.md`
- `.github/prompts/5-verifying.prompt.md`

目标：

- 把流程从“临场发挥”变成“固定阶段”

### 第三天

创建：

- `.github/TaskLogs/`
- 所有基础状态文件

目标：

- 让阶段结果能落盘、恢复、复查

### 第四天

创建：

- `.github/KnowledgeBase/Index.md`
- 2 到 3 篇高价值知识库文档

建议优先写：

- 项目架构
- 模块职责
- API 选型规则

### 第五天

开始用真实任务跑一轮：

- `scrum`
- `design`
- `plan`
- `execute`
- `verify`

然后根据实际卡点再补文档和脚本。

## 常见失败方式

### 失败方式 1：只写一个超长总 Prompt

后果：

- 难维护
- 难迭代
- 不同阶段职责混乱

正确做法：

- 用路由文件 + 分阶段 prompt

### 失败方式 2：没有 `TaskLogs`

后果：

- 多轮工作容易丢上下文
- 计划、执行、验证混在聊天里
- 难恢复、难审查

正确做法：

- 把阶段性成果写进固定文档

### 失败方式 3：让 AI 自己发明 build/test 命令

后果：

- 不同 agent 跑法不一致
- 验证不可重复

正确做法：

- 统一用 `scripts/build.sh` 和 `scripts/test.sh`

### 失败方式 4：没有 `update` 机制

后果：

- 一旦中途变更，流程就变乱
- 只能从头重来或在聊天里硬补

正确做法：

- 所有阶段都支持 update

### 失败方式 5：把“AI 说完成”当成完成

后果：

- 容易留下隐藏回归和遗漏测试

正确做法：

- 只接受外部证据
- 必须 build
- 必须 test
- 必要时必须 review

## 你真正要复制的是什么

如果你从本仓库只带走一个结论，那应该是：

你不需要先复制 `.github/Agent` 的 portal、session、jobs、live polling。

你真正应该先复制的是这四层：

1. `AGENTS.md` 的请求路由
2. `prompts/` 的阶段流程
3. `TaskLogs/` 的外部状态
4. `scripts/` 的统一执行入口

这四层一旦建起来，Claude Code、OpenCode、Codex 这类现成 agent 就已经可以驱动一套稳定的项目级 AI 协作流程。

`.github/Agent` 这种平台化能力，是第二阶段优化，不是第一阶段前提。

## 最后一条建议

不要一开始追求“自动化编排得多漂亮”。
先追求下面这件事：

`同一个任务，换一个 agent 来做，流程仍然一致、状态仍然可恢复、验证仍然可信。`

如果你做到了这一点，这套方法已经落地成功。
