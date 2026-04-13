# Skill 功能与命令系统

OpenClaw 的 Skill 系统是其核心扩展机制。本文介绍 Skill 的使用原则、语音处理能力以及斜杠命令的本质。

## Skill 使用原则：不是越多越好

一个常见的误区是"Skill 越多 Agent 越强"。实际情况恰恰相反：

> [!warning]
> Skill 越多 → Agent 决策空间越大 → 更慢 + 更乱 + 更贵

### 原因分析

Agent 每次思考都需要遍历所有 Skill：

```
Should I use skill A?
Should I use skill B?
Should I use skill C?
...
```

Skill 多 = Token 浪费 + 延迟增加

### 推荐做法

- 只安装真正需要的 Skill
- 定期清理不常用的 Skill
- 根据使用场景按需启用

## Skill 相关命令

```bash
# 查看已安装的 Skills（两个命令等效）
openclaw skills list
openclaw skills

# Skills 配置位置
~/.openclaw/openclaw.json  # 开关和参数配置
~/.openclaw/skills/        # 源代码目录
```

> [!note]
> 手动修改配置文件后，必须执行 `openclaw gateway restart` 才能生效。

## 常用 Skill 推荐

### searxng 技能

searxng 是一个强大的联网查询 Skill：

| 功能 | 说明 |
|------|------|
| 实时联网 | 查询最新新闻、天气、股票等即时信息 |
| 隐私保护 | 向搜索引擎发起请求时剔除 IP 等私人数据 |
| 无外部依赖 | 连接本地 SearXNG 实例，无需付费 API |

### Skill 安装注意事项

看到 Skill 目录下的 `SKILL.md` 文件不代表 Skill 已可用。需要：

```bash
cd ~/.openclaw/skills/github
npm install           # 安装依赖
openclaw gateway restart  # 重启加载
```

## 语音处理：TTS 与 STT

OpenClaw 支持语音交互，理解两个核心概念：

| 缩写 | 全称 | 中文 | 方向 | 本质 |
|------|------|------|------|------|
| **STT** | Speech To Text | 语音识别 | 声音 → 文本 | 听写 |
| **TTS** | Text To Speech | 语音合成 | 文本 → 声音 | 朗读 |

简单记忆：
- STT = 听 → 文字
- TTS = 文字 → 说

两者是方向相反的逆过程。

## 斜杠命令的本质

Slash Command（斜杠命令）不是魔法，本质是 **预设 Prompt 模板**：

```
/短命令 → 展开成一大段固定提示词
```

### Telegram 的特殊支持

Telegram 对 Slash Command 有原生支持：

- 输入 `/` 时自动弹出可用命令列表
- 这是 Telegram 自带的体验，无需额外开发

### 实际应用

斜杠命令适合：
- 常用的复杂 Prompt
- 标准化的工作流程
- 团队共享的指令模板

## systemd 守护进程

启用 systemd 守护进程的好处：

> [!success]
> 程序崩溃后会自动瞬间重启

这是生产环境必备的可靠性保障。

---

Skill 系统是 OpenClaw 灵活性的来源，但"少即是多"——选择真正需要的 Skill，保持配置精简，才能获得最佳体验。
