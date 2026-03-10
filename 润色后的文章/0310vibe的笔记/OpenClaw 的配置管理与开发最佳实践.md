# OpenClaw 的配置管理与开发最佳实践

在使用 OpenClaw 进行开发和运维时，合理的配置管理和规范约定能够大幅提升工作效率和系统稳定性。本文总结了 OpenClaw 在工作空间管理、配置文件组织、SSH 认证等方面的最佳实践。

## 工作空间的组织策略

OpenClaw 官方推荐：**一个 workspace，多个项目放在这个 workspace 下**。

建议只保留一个当前正在使用的 workspace，因为同一时间只有一个 workspace 会生效，同时存在多个 workspace 目录容易造成状态或认证混乱。

因此不要为每个项目分别创建独立的 `~/.openclaw/workspace`，而是在同一个 workspace 下通过子目录区分不同项目，例如：

```bash
/home/admin/.openclaw/workspace/my-project-1
/home/admin/.openclaw/workspace/my-project-2
```

这样既保持结构清晰，又避免环境冲突。

## TOOLS.md 与 MEMORY.md 的职责划分

在 OpenClaw 中，配置文件的职责划分非常明确，理解这一点有助于构建更加清晰的知识管理体系。

### TOOLS.md：环境与工具的使用约定

TOOLS.md 适合放：**环境/工具的使用约定、操作规则、"在这里应该怎么执行"**。

具体例子包括：
- 脚本要在 tmux 里跑
- 图片输出必须放某目录
- 用哪个 tmux socket
- SSH 主机别名
- 执行命令前需要加载的环境变量

例如，"每次运行要在 tmux 里跑脚本"这种开发过程约束，本质上是规定执行方式和运行环境，属于"工具如何使用、在什么环境下运行"的约定。将其放在 TOOLS.md 能让 AI 在做执行相关决策时优先读取并遵循这些规则，保证操作一致性和环境正确性。

### MEMORY.md：长期事实与决策记录

MEMORY.md 适合放：**长期事实、你的偏好、做过的决定、要"记住"的事**。

具体例子包括：
- 用户偏好（代码风格、命名习惯等）
- 项目重要决定（技术选型、架构调整等）
- 约定好的流程结论（评审通过的方案、已废弃的做法等）

> [!tip]
> **选择原则**：
> - 如果是"怎么执行、怎么操作"的规则 → TOOLS.md
> - 如果是"记住什么、偏好什么"的内容 → MEMORY.md

## AGENTS.md：系统核心运行规则

AGENTS.md 用来定义系统的核心运行规则与行为规范。所有与 Agent 相关的内容都会集中写在这里，包括：

- **执行流程**：任务如何启动与衔接
- **工作规范**：编码或操作标准
- **使用规则**：工具与权限边界
- **记忆管理方式**：如何读写与隔离记忆

简单来说，AGENTS.md 是整个 Agent 体系的"操作说明书"，负责统一约束和指导各个子会话与主流程的行为。

## SSH 认证配置的常见问题

在使用 Termux 通过用户名和密码进行 SSH 登录时，如果无法登录，通常是因为服务器 SSH 配置只允许密钥登录。解决方案如下：

### 步骤 1：修改 SSH 配置

在 `sshd_config` 里设置：

```bash
PasswordAuthentication yes
```

这将开启密码认证选项。

### 步骤 2：重启 SSH 服务

```bash
sudo systemctl restart sshd
```

### 步骤 3：检查用户密码状态

使用以下命令检查：

```bash
passwd -S admin
```

状态含义：
- **NP** = No password（没有密码）
- **LK** = Locked（锁定）
- **PS** = Password set（正常）

如果 admin 显示 NP（无密码），需要设置密码：

```bash
sudo passwd admin
```

## 查看服务器 IP 地理信息

在服务器里查看服务器对应的 IP 的相关信息（如城市、国家等），可以使用以下命令：

```bash
curl ipinfo.io
```

它会向 ipinfo.io 服务发送请求，返回当前公网 IP 的详细信息，格式为 JSON，包括国家、地区、城市、ISP 等信息。

---

通过遵循这些最佳实践，你可以构建更加清晰、稳定、易维护的 OpenClaw 开发环境，提升整体工作效率和系统可靠性。
