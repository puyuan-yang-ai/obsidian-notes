# OpenClaw 的进程管理与 Gateway 架构解析

OpenClaw 的进程管理架构采用了父子进程分离的设计，通过 Gateway 作为常驻调度中心，统一管理所有子进程的生命周期，实现了高效、可控的任务执行机制。

## Gateway 的角色定位与职责边界

Gateway 本质上是一个常驻运行的父进程服务，负责监听 Telegram、HTTP、CLI 等入口，接收命令、调度任务并管理进程生命周期，但它本身**不执行**具体的 AI 推理或脚本逻辑。

真正干活的是 Gateway spawn 出来的子进程，这些子进程才会调用 claude、运行脚本、执行 skill 或调用 shell 命令。Gateway 的职责仅限于：创建子进程、轮询运行状态、收集输出结果，并在必要时发送终止信号进行清理。

整体结构是：启动 OpenClaw → Gateway 常驻运行 → Gateway 按需创建多个子进程 → 所有具体任务都在这些子进程中完成。

> [!tip]
> **进程层级关系总结**：
> - Gateway 是常驻进程
> - 所有任务在子进程中跑
> - Gateway 负责调度和生命周期管理
> - 子进程负责实际 AI 推理或执行

## 环境变量的继承机制

OpenClaw 使用 `bash -c "xxx"` 执行命令时，涉及三层进程关系：

```bash
Gateway (父进程, PID=1034)
  └─ bash (子进程, 由 `bash -c "xxx"` 启动)
      └─ "xxx" 命令在这个 bash 子进程中执行
```

需要注意的是，`bash -c "xxx"` 启动的是 **non-login non-interactive shell**，因此不会读取 `.bashrc` 等配置文件。即使设置了 BASH_ENV，也无法在脚本中使用，因为脚本启动时 BASH_ENV 还未设置。

解决方案是在 **gateway.env** 文件中设置环境变量。Gateway 中的环境变量会被 bash 子进程继承，bash 子进程中的命令可以使用这些环境变量。环境变量的继承链是：Gateway（有环境变量）→ bash 子进程（继承 Gateway 的环境变量）→ "xxx"命令（可使用 bash 子进程的环境变量）。

## 进程管理的命名机制与轮询操作

OpenClaw 为每个子进程自动分配一个随机名称，例如 **neat-canyon** 或 **cool-shoal**。这个名称不是 Linux 系统进程名，也不是系统 PID，而是 OpenClaw 内部生成的 **process label**，用来做查询、轮询和终止等管理操作。

这种"形容词+名词"的随机命名方式和 Docker 容器的随机名类似，主要目的是让名称更易读、避免直接暴露系统 PID，同时减少命名冲突。名字自动随机生成，每个进程唯一，任务结束后释放。

当你看到以下输出时，它们都是 OpenClaw 在"内部进程管理层"做的操作：

- **Process: list**：列出当前由 OpenClaw 管理的所有子进程
- **Process: poll · neat-canyon**：对名为 neat-canyon 的子进程进行一次状态查询，判断它是否运行完成、是否仍在执行、是否有新的输出，作用类似于 `ps`，但只作用于 OpenClaw 自己维护的进程池
- **Process: kill · cool-shoal**：终止名为 cool-shoal 的子进程，通常由 OpenClaw 主进程向该子进程发送 SIGTERM 或 SIGKILL 信号强制结束

> [!info]
> **poll 操作不会消耗 LLM token**：
> poll 只是 OpenClaw Gateway 与本地子进程之间的一次状态检查操作，本质是查询该子进程是否已结束、是否有新的 stdout 输出、exit code 是什么等运行状态信息。整个过程发生在本地进程管理层，不会调用 Claude API、OpenAI API 或任何大模型接口，因此既不会产生 token 消耗，也不会产生 API 费用。

## 与其他运行环境的对比

在不同的运行环境下，父进程关系存在差异：

- **OpenClaw**：父进程是 gateway
- **直接 ssh 跑 cc**：父进程是 ssh session
- **tmux 跑 cc**：父进程是 tmux server

这种差异决定了进程的生命周期管理方式。在 OpenClaw 中，Gateway 作为统一的调度中心，能够精确控制每个子进程的启动与终止，实现更加稳定和可控的任务管理。

## Cron 定时调度机制

OpenClaw 内置的 Cron 机制指的是 Gateway 内置的定时任务与调度能力，它的作用是在指定时间或按固定周期主动"唤醒"智能体并执行任务，例如提醒事项、每日简报或周期性检查等。

Cron 运行在 **Gateway 进程层**，不是模型内部，属于系统级调度能力。相关任务持久化在 **~/.openclaw/cron/**（如 jobs.json），重启也不会丢失。支持定义"何时执行"（一次性 at 或周期性的 every/cron 表达式）和"执行什么内容"，实现时间条件与任务行为的组合调度。

## 驱动 Claude Code 的正确方式

用 OpenClaw 驱动 Claude Code 做编程任务，官方推荐的方式**不是** sessions_spawn，而是 **"coding-agent skill + bash/process"**。

当你对 OpenClaw 说"用 Claude Code 在 xx 项目里做 xxx"时，不需要手动写 `pty:true` 或 `background:true`。OpenClaw 的智能体会自动判断可以调用 coding-agent 这个 skill，并按照 skill 内定义的流程去执行：通过 bash 工具运行命令，自动附带 `pty:true`（长任务则加上 `background:true`），如果是后台运行，还会使用 process 的 list/poll/log 等能力进行监控。

sessions_spawn 本质是"主 agent → 子 agent"的调度机制。子 agent 若需要运行 Claude Code，仍然是通过 bash 或 process 等工具去执行，真正驱动 Claude Code 的是这些底层工具，而不是 spawn 本身。

通过这种清晰的职责分离和进程管理机制，OpenClaw 实现了高效、稳定、可控的任务调度能力，为复杂的 AI 辅助开发提供了坚实的基础架构。
