# OpenClaw Skills与高级功能

OpenClaw 的 Skills 系统是其核心扩展机制，允许用户为机器人添加各种能力。本文介绍 Skills 的分类、安装方式以及高级使用技巧。

## Skills 分类

OpenClaw Skills 分为四类：

| 类型 | 位置 | 说明 |
|------|------|------|
| 内置 skills | openclaw-bundled | 随安装提供，如 `coding-agent` |
| home 目录托管 | ~/.openclaw/skills/ | 用户自行安装的技能 |
| 工作区 skills | 工作区 /skills 目录 | 项目级别的技能 |
| clawhub registry | 社区注册表 | 社区贡献，通过 clawhub 安装 |

## 查看已安装 Skills

```bash
openclaw skill list
# 或简写
openclaw skill
```

## 安装第三方 Skills

### 方式一：ClawHub CLI 安装（推荐）

```bash
npx clawhub@latest install coding-agent
npx clawhub@latest install notion
```

这会从 SkilHub 注册表下载技能到 `~/.openclaw/skills/` 目录。

### 方式二：手动克隆

```bash
cd ~/.openclaw/skills/
git clone https://github.com/<username>/<skill-name>.git
```

### 方式三：直接复制目录

将下载好的 skill 目录直接放进 `~/.openclaw/skills/`，OpenClaw 启动时会自动加载。

> [!warning]
> **不支持 `openclaw skills install xxx`**
> `openclaw skills` 命令只能显示已安装/未安装的 skill，不能用于安装。安装必须通过 npx clawhub 或克隆到 skills 目录。

## coding-agent 详解

`coding-agent` 是内置 skill，**不需要安装**。如果显示 `missing` 状态：

> [!info]
> **missing 状态原因**
> coding-agent 需要底层 CLI 工具（claude/codex/opencode/pi）作为执行引擎。安装任一 CLI 工具（如 Claude Code 或 Codex）后，skill 会自动变为可用。

### 高手的标准流程

网上高手的共识是：**不要直接利用 OpenClaw 生成代码**，而是通过 coding-agent 调用专业工具：

```
Telegram → OpenClaw → coding-agent → Claude Code → 执行 → 回传
```

而不是：
```
Telegram → OpenClaw → 模型直接生成代码 ❌
```

## 配置生效机制

修改 skill 后需要执行：

```bash
openclaw gateway restart
```

因为网关程序只在启动时读取硬盘上的配置文件，运行期间修改不会自动感知。

## /verbose 指令详解

```bash
/verbose on   # 开启详细模式
/verbose off  # 关闭
```

**开启后展示**：
- 思考步骤
- 工具调用细节
- 具体执行日志

**使用场景**：想知道机器人为什么回答错误，或者它在后台偷偷运行了什么命令。

## /reasoning 和 /think 指令

| 指令 | 作用 | 注意事项 |
|------|------|----------|
| /verbose | 开关详细模式 | 全局适用 |
| /reasoning | 开关推理内容显示 | 部分版本支持 |
| /think <level> | 控制思考深度 | 仅针对支持思维链的模型（如 o1/o3） |

## 高效使用建议

结合 [[OpenClaw安装与基础配置]] 中的人格设置建议，配合 Skills 系统使用：

1. **选择合适的 skill**：根据任务选择 coding-agent、notion 等专业 skill
2. **开启 verbose 调试**：遇到问题时开启详细模式排查
3. **善用 compact**：长对话时压缩上下文节省 token
4. **配置搜索能力**：通过 Brave API 让机器人具备联网搜索能力

## 相关链接

[[OpenClaw安装与基础配置]] [[OpenClaw Telegram与消息平台集成]] [[Shell类型与环境变量机制]]
