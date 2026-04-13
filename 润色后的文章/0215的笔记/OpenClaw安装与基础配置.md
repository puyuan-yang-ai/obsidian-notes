# OpenClaw安装与基础配置

OpenClaw 是一款强大的 AI Agent 运行平台，支持多种消息平台集成和丰富的扩展能力。本文介绍在 Linux VPS 上安装和配置 OpenClaw 的完整流程。

## 安装步骤

在 Linux VPS 上安装 OpenClaw 只需一条命令：

```bash
curl -fsSL https://openclaw.ai/install.sh | bash
```

安装完成后，使用 `openclaw --version` 确认安装成功。接下来执行配置向导：

```bash
openclaw onboard
# 或推荐版本（同时安装守护进程）
openclaw onboard --install-daemon
```

这会启动一个交互式的终端引导界面（TUI），逐步完成各项配置。

> [!tip]
> **Linuxbrew 支持**
> Linux VPS 完全可以安装 Homebrew（官方长期支持，叫 Linuxbrew）。同一套 brew 同时支持 macOS 和 Linux，不是 hack 方案。

## 配置向导选择

配置向导中会遇到几个关键选择：

### Config handling（配置处理方式）

- **Use existing values**：复用现有配置，适合已配好只需重启/升级的情况
- **Update values**（推荐）：逐步进入配置流程填写 provider/key/model 等，适合新服务器首次配置
- **Reset**（危险）：删除 `~/.openclaw/` 全部配置恢复出厂，会清除所有认证状态和 pairing 数据

### Hatch 方式选择

- **Hatch in TUI**（推荐）：终端界面启动，最稳定、最省资源，VPS/SSH 环境首选
- **Open the Web UI**：浏览器控制台，VPS 不方便使用（需要开端口和隧道）
- **Do this later**：跳过启动，没有实际意义

### Gateway 运行位置

选择 **Local (this machine)** 表示在当前服务器启动"大脑"和"神经中枢"进程。Local 并不意味着断网或局域网限制，VPS 有公网 IP，运行在 Local 模式下依然可以正常连接 Telegram、WhatsApp 和 AI 模型接口。

## 人格设置建议

> [!warning]
> **避免 Chatty personality**
> 在 GitHub、Discord、Reddit 等社区讨论中，几乎一致认为 Chatty personality 人格设置是灾难。原因是：回复变长 → 上下文爆炸 → Token 成本增加 → 决策变慢。

社区高手的共识是：**不要把 OpenClaw 变成"有性格的聊天机器人"，要把它变成"冷血执行引擎"**。

原因分析：
- personality 越强 → token 越多
- 越像人 → 越容易闲聊
- 越 creative → 越容易产生幻觉（hallucinate）

生产环境追求的是：可预测、可重复、少废话、优先调用工具、快。

## 打造高效工作机器

要将 OpenClaw 打造成提升效率的工作机器，需要修改以下文件：

| 文件 | 作用 | 生产力方向 |
|------|------|-----------|
| SOUL.md | 行为准则/输出风格/思维模式 | 控制废话、啰嗦、情绪 |
| IDENTITY.md | 角色定位/专业边界 | 限制乱发挥 |
| USER.md | 用户偏好 | 决定输出语言、简洁度 |

> [!info]
> **AGENT.md 不需要修改**
> 大多数情况下 AGENT.md 不需要修改，它是系统级框架（Agent 的基础运行规则）。主要改的是 SOUL.md、IDENTITY.md、USER.md。

## 配置生效机制

修改身份和灵魂文件后，**不需要重启 gateway**。因为 SOUL.md、IDENTITY.md、USER.md 是在每个新会话开始时读取的，Gateway 是服务组件不直接读取这些配置，新会话会自动读取新配置。

但如果修改了 `openclaw.json`、`.env` 文件或 skill，则需要执行：

```bash
openclaw gateway restart
```

原因是网关程序（Gateway Daemon）只在启动时读取硬盘上的配置文件，运行期间修改不会自动感知，必须重启才能重新加载。

## 相关链接

[[SSH配置与服务器连接]] [[OpenClaw Telegram与消息平台集成]] [[OpenClaw Skills与高级功能]]
