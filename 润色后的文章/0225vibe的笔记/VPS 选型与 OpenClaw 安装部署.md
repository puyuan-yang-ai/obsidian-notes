# VPS 选型与 OpenClaw 安装部署

部署 OpenClaw 的第一步是选择合适的 VPS。本文将介绍 VPS 选型建议、安装流程以及配置向导的关键选项。

## VPS 选型：为什么不用国内服务器

### 国内 VPS 的三大限制

| 限制类型 | 具体问题 |
|----------|----------|
| **安装困难** | 一键脚本依赖 GitHub，依赖包需要访问 NPM/Homebrew 国际源，经常超时失败 |
| **AI 模型** | 无法直接访问 Anthropic (Claude)、OpenAI 等 API，会 Connection refused |
| **聊天软件** | Telegram、WhatsApp、Discord 均被屏蔽，无法建立连接 |

### 为什么不推荐香港 VPS

虽然香港理论上网络更开放，但 **OpenAI、Gemini、Claude 三大模型在香港仍无法使用**。

### 推荐选择：新加坡 VPS

新加坡是最佳选择：
- 🌏 距离中国近，延迟低
- 🔓 无限制使用三大模型
- 💰 价格合理

### 配置建议

- **内存**：4GB（避免安装时内存溢出）
- **系统**：Ubuntu 22.04

## 一键安装脚本

OpenClaw 提供了极简的安装方式：

```bash
curl -fsSL https://openclaw.ai/install.sh | bash
```

### 脚本背后的流程

1. 从 OpenClaw 官方服务器下载安装脚本
2. 自动检测系统类型（Ubuntu/Debian 等）
3. 自动安装 Node.js 22+（运行环境）
4. 下载核心程序并安装到系统路径

### 验证安装

```bash
openclaw --version
```

## 配置阶段（Onboarding）

安装完成后进入配置阶段，这是让机器人"活"过来的关键步骤：

```bash
openclaw onboard --install-daemon
```

> [!tip]
> `--install-daemon` 参数非常重要！它会将 OpenClaw 设置为后台服务，关闭 SSH 窗口后机器人不会掉线。

## 配置向导选项

### 必须配置的项目

| 配置项 | 推荐选择 | 说明 |
|--------|----------|------|
| Onboarding Mode | QuickStart | 快速开始模式 |
| Gateway Configuration | Local | 本地运行 |

### 可后续配置的项目

| 配置项 | 推荐选择 | 说明 |
|--------|----------|------|
| AI Provider | OpenAI codex OAuth | 免费登录 |
| Messaging Channel | Telegram | 最稳定、配置最简单 |
| Skills | github, model-usage | 常用技能 |

## `onboard` vs `configure` 两个命令的区别

| 命令 | 用途 | 场景 |
|------|------|------|
| `openclaw onboard` | 首次安装后的初始化引导 | 全新安装 |
| `openclaw configure` | 后期调整已有配置 | 修改 API Key、更换模型等 |

**简单记忆**：
- 🆕 全新安装 → 用 `onboard`
- 🔁 因 429/模型问题重配 → 用 `configure`

## 国内 VPS 的替代方案

如果必须使用国内 VPS，需要接受以下限制：

- **AI 模型**：仅限国内模型（Kimi、Qwen）
- **聊天软件**：仅限飞书/钉钉

## 配置文件位置

所有配置存储在：

```
~/.openclaw/openclaw.json
```

可以直接编辑此文件修改配置，但修改后需要重启网关：

```bash
openclaw gateway restart
```

## 安装后检查

完成配置后，建议执行以下检查：

```bash
# 检查服务状态
openclaw gateway status

# 检查 Channel 连接
openclaw channels status --probe
```

---

部署 OpenClaw 的核心是选择合适的 VPS 和正确完成配置向导。按照本文的推荐，你可以快速搭建一个稳定运行的 AI Agent 服务。
