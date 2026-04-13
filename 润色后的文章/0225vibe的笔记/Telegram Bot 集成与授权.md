# Telegram Bot 集成与授权

将 OpenClaw 与 Telegram Bot 集成是最推荐的远程交互方案。本文详细介绍从 Bot 创建到完成授权的完整流程。

## 创建 Telegram Bot

在 Telegram 中创建 Bot 非常简单：

1. 搜索 **@BotFather**（Telegram 官方的 Bot 管理工具）
2. 发送 `/newbot` 命令
3. 按提示设置 Bot 名称和 username（username 必须以 `bot` 结尾且全网唯一）
4. 获得一串 Token（格式：`123456:ABC...`）

> [!warning]
> Token 是控制你机器人的钥匙，务必妥善保管，不要泄露给他人。

## 理解 Token 格式

Telegram Bot Token 的标准格式是：

```
<bot_id>:<secret_hash>
```

- **冒号前**（数字）：机器人唯一 ID
- **冒号后**（字符串）：认证密钥
- **整串**：API 调用时必须完整使用

## 验证 Token 有效性

拿到 Token 后，在服务器上执行以下命令验证：

```bash
curl -s "https://api.telegram.org/bot<完整token>/getMe"
```

- 返回 `{"ok":true,...}` → Token 有效
- 返回 `401 Unauthorized` → Token 错误/有空格/复制错误

## OpenClaw 与 Telegram 的两层配置

通过 Telegram 使用 OpenClaw 需要完成两个阶段的配置：

### Layer 1：Bot 认证（系统层）

将 Bot Token 输入到 OpenClaw，解决"OpenClaw 是否有资格控制这个 Bot"的问题。

### Layer 2：用户授权（个人层）

解决"谁可以使用这个 Bot"的问题，流程如下：

```
Telegram 输入 /start
        ↓
OpenClaw 生成配对码（如 PVX8A3T）
        ↓
配对码发送到 Telegram
        ↓
终端执行：openclaw pairing approve telegram PVX8A3T
        ↓
你的 user_id 加入 allowlist
        ↓
之后永久允许访问
```

## 检查与验证

配置过程中常用的检查命令：

```bash
# 检查 Channel 配置状态
openclaw channels status --probe

# 重启网关使配置生效
openclaw gateway restart
```

> [!info]
> **重要时序**：必须先执行 `openclaw channels status --probe` 看到 Telegram 条目后，再去 Telegram 发送 `/start`。

## 私聊权限策略选择

配置时会询问 "Configure DM access policies now"，这是设置 Telegram 私聊权限控制策略。

三种模式对比：

| 模式 | 特点 | 适用场景 |
|------|------|----------|
| **pairing** ⭐ | 安全、只允许自己 | 个人使用（强烈推荐） |
| allowlist | 需手动填 user id | 企业/多人使用 |
| public | 任何人都能聊天 | ⚠️ 非常危险，不推荐 |

> [!danger]
> **绝对不要选择 public 模式**！这等于将你的 Agent 暴露在公网，可能被恶意利用执行危险命令，或被刷爆 API。

## 为什么选择 Telegram

全球 Dev/Agent/量化/运维圈首选 Telegram 的原因：

- ✅ 消息数量近乎无限
- ✅ 服务稳定可靠
- ✅ 配置简单
- ✅ Webhook 支持完善
- ✅ 完全免费
- ✅ Linux 服务器友好

常见使用场景包括：量化交易通知、服务器运维监控、GPU 训练进度、cron 任务报警等。
