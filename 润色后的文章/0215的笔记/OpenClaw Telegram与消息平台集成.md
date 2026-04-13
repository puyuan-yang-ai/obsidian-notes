# OpenClaw Telegram与消息平台集成

OpenClaw 支持多种消息平台集成，其中 Telegram 和 WhatsApp 是最常用的两个。本文介绍如何正确配置和使用这些平台。

## Telegram Bot 配置

### 配对机制

Telegram Bot 的 `/start` 指令核心功能是生成配对码（Pairing Code）建立连接。后续需要在终端 approve 这个配对码，将用户 ID 加入服务器的白名单（Allowlist）。

配置时会遇到 "Configure DM access policies now? (default: pairing)" 选项：

> [!info]
> **DM Policy 选择**
> - pairing（默认）：必须先通过配对码绑定，适合个人/私有服务器
> - 选择 No 即使用默认 pairing 模式

### 排查无响应问题

新建 Telegram Bot 并配置 token 后，如果在 Bot 对话页面输入 `/start` 没有反应：

```bash
# 在服务器执行以下命令排查
curl https://api.telegram.org/bot<你的TOKEN>/getWebhookInfo
```

99% 的情况是 OpenClaw 没有正确接收到 webhook，而非 Telegram 问题。

### 多服务器连接

如果手机 Telegram 已连接某服务器的 OpenClaw，想连接新服务器：

> [!tip]
> **推荐：多 Bot 隔离**
> 在 BotFather 中执行 `/newbot` 创建新 Bot，获取新 token 配置到新服务器。优点是完全隔离、无冲突、最安全。

## WhatsApp 集成

OpenClaw 连接 WhatsApp 的本质是**模拟网页版登录**。通过 `openclaw channels login` 扫码，扫码成功后，OpenClaw 相当于在你的手机上"登录"了 WhatsApp。

## 斜杠指令详解

OpenClaw 在 Telegram 中支持多种斜杠指令：

### /compact - 压缩上下文

**作用**：将之前对话总结成摘要，腾出上下文空间。

**使用场景**：不想完全重置会话（需要保留关键信息）但又想节省 Token 时使用。

### /verbose - 详细模式

```bash
/verbose on   # 开启
/verbose off  # 关闭
```

**作用**：开启后机器人会展示"思考过程"、调用的工具细节和执行日志。

**使用场景**：排查回答错误或了解后台执行了什么命令。

### /think - 思考深度控制

```bash
/think <level>
```

**注意**：此指令主要针对支持思维链（CoT）的模型，如 OpenAI o1/o3 或支持思考参数的模型，不是所有模型都支持。

### 模型切换

在聊天窗口用斜杠指令切换模型，**只能临时切换当前会话的模型**，无法修改默认模型。永久修改默认模型需要编辑 `openclaw.json` 中的 `primary` 字段。

## ChatGPT OAuth 选项

选择 "OpenAI Codex with ChatGPT OAuth" 选项的好处：

> [!success]
> **节省 API 费用**
> 不需要额外付 API 费用，可以利用已购买的 $20/月 ChatGPT Plus 订阅来消耗 Token，而不是走按量计费的 API 通道。

## Brave API 搜索能力

配置 Brave API 的作用是让机器人具备联网搜索能力。配置后**必须重启网关**：

```bash
openclaw gateway restart
```

这是为了让网关进程读取新的 API Key 并加载相应的搜索技能。

## 记忆系统

OpenClaw 有两种记忆文件：

| 文件 | 类型 | 用途 |
|------|------|------|
| MEMORY.md | 长期记忆 | 存储重要决策、偏好、关键项目背景 |
| memory/YYYY-MM-DD.md | 短期记忆 | 日记式上下文，可过期 |

## 相关链接

[[OpenClaw安装与基础配置]] [[OpenClaw Skills与高级功能]]
