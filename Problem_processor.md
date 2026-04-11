# Problem Processor

这个文件当前只说明 Task 的处理过程，也就是 `design` 指令如何驱动 `Copilot_Task.md`。

## 入口机制

Task 过程由首词 `design` 触发，对应读取：

- `.github/prompts/1-design.prompt.md`

然后根据请求的第二个词决定生成的标题。
在当前仓库规则下，Task 流程稳定可用的入口是：

- `design problem`
- `design update`

它们会把消息重写为：

- `design problem ...` -> `# Problem`
- `design update ...` -> `# Update`

而 `1-design.prompt.md` 会根据这个标题决定进入哪个分支。

## Task 流程图

```text
用户输入 design ...
        |
        v
读取 .github/prompts/1-design.prompt.md
        |
        v
检查重写后的最新消息标题
        |
        +-----------------------------+
        |                             |
        v                             v
     # Problem                     # Update
        |                             |
        v                             v
创建或刷新当前任务文档            更新当前任务文档
Copilot_Task.md                   Copilot_Task.md
        |                             |
        |                             |- 把更新追加到 # UPDATES
        |                             |- 按更新调整设计内容
        |
        |- 执行 copilotPrepare.ps1
        |- 清理 Task / Planning / Execution
        |- 解析当前任务来源
        |
        +-----------------------------+
        |                             |
        v                             v
   Problem = Next               Problem = Complete task No.X
        |                             |
        v                             v
从 Copilot_Scrum.md 取第一个        从 Copilot_Scrum.md 取指定任务
未完成任务
        |                             |
        +-------------+---------------+
                      |
                      v
             写入 Copilot_Task.md
                      |
                      |- # PROBLEM DESCRIPTION
                      |- # UPDATES
                      |- # INSIGHTS AND REASONING
                      |- # AFFECTED PROJECTS
                      |- # !!!FINISHED!!!
                      |
                      v
                当前 Task 生效
                      |
                      v
         后续交给 plan / execute / verify
                      |
                      v
         下一次 design problem 到来时
         Copilot_Task.md 被新任务刷新
```

## `design problem` 的过程

典型输入：

```md
design problem
Next
```

或：

```md
design problem
Complete task No.3
```

处理过程：

1. 进入 `1-design.prompt.md` 的 `# Problem` 分支。
2. 执行 `copilotPrepare.ps1`。
3. 清理当前任务工作区文档：
   - `Copilot_Task.md`
   - `Copilot_Planning.md`
   - `Copilot_Execution.md`
4. 解析 `# Problem` 的正文：
   - `Next`：从 `Copilot_Scrum.md` 选择第一个未完成任务。
   - `Complete task No.X`：从 `Copilot_Scrum.md` 选择指定任务。
   - 其他文本：直接把文本本身作为当前任务。
5. 如果任务来自 `Copilot_Scrum.md`，把对应任务标记为 `[x]`。
6. 把任务详细内容写入 `Copilot_Task.md`。
7. 完成高层设计分析，补全 `# INSIGHTS AND REASONING` 与 `# AFFECTED PROJECTS`。
8. 在末尾加上 `# !!!FINISHED!!!`。

## `design update` 的过程

典型输入：

```md
design update
补充新的约束或修改意见
```

处理过程：

1. 进入 `1-design.prompt.md` 的 `# Update` 分支。
2. 把更新原文追加到 `Copilot_Task.md` 的 `# UPDATES`。
3. 根据这个更新调整 `Copilot_Task.md` 的设计内容。
4. 不切换到新任务。
5. 不修改源码。

## `Copilot_Task.md` 的职责

`Copilot_Task.md` 只服务于当前一个任务，结构固定为：

- `# !!!TASK!!!`
- `# PROBLEM DESCRIPTION`
- `# UPDATES`
- `# INSIGHTS AND REASONING`
- `# AFFECTED PROJECTS`
- `# !!!FINISHED!!!`

其中重点是：

- `# INSIGHTS AND REASONING`
  - 记录这个任务准备怎么做。
  - 解释为什么这样做。
  - 引用源码和知识库证据。
- `# AFFECTED PROJECTS`
  - 说明后续需要 build / run 的 solution 或 project。
  - 顺序应明确，因为后续步骤会按这个顺序执行。

## Task 的生命周期

Task 不是长期并行维护多份的文档，而是当前激活任务的单槽位文档。

```text
design problem
        |
        v
创建或刷新 Copilot_Task.md
        |
        v
当前任务成为唯一激活任务
        |
        +--> design update
        |        |
        |        v
        |   在当前任务上追加更新
        |
        v
进入后续 plan / execute / verify
        |
        v
下一次 design problem
        |
        v
Copilot_Task.md 被新任务替换
```

因此可以这样理解：

- `Copilot_Scrum.md` 管全部任务。
- `Copilot_Task.md` 只管当前任务。
- 当处理下一个任务时，`Copilot_Task.md` 会被刷新，而不是为每个任务长期保留一份独立副本。

## 最常用的 Task 输入模板

选择下一个任务：

```md
design problem
Next
```

选择指定任务：

```md
design problem
Complete task No.X
```

给当前任务补充要求：

```md
design update
这里写补充说明
```
