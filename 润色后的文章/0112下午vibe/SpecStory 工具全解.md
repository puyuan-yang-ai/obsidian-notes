# SpecStory 工具全解

## 概述

SpecStory 是一个专门为 Claude Code 等 AI 编程助手设计的会话记录工具。它能够实时捕获、保存和管理你的 AI 对话历史，让知识沉淀变得自动化。本文将全面介绍 SpecStory 的安装、工作原理和使用技巧。

## 什么是 SpecStory？

> [!question] **npm 上有 specstory 包吗？**
> - 根本不存在
> - SpecStory 不是 npm 包，无法通过 `npm install -g specstory` 安装
> - 正确的安装方式取决于你的操作系统

SpecStory CLI 的核心功能是以**父进程方式**启动 Claude，监听 stdin/stdout，将完整对话实时保存为 markdown 文件。这意味着：

- 所有对话内容被完整记录，不会丢失
- 以标准 markdown 格式存储，便于后续查阅和分享
- 支持多个 AI agent（Claude、Codex、Gemini 等）

## 安装指南

### macOS 安装

在 macOS 上，Homebrew 几乎是默认存在的包管理器，安装非常简单：

```bash
brew install specstory
```

### Linux/WSL 安装

在 Linux 环境（包括 WSL）上，Homebrew 默认不存在，需要手动下载 GitHub Release：

```bash
# 1. 下载 Linux (x86) 版本
wget https://github.com/.../specstory-linux-x86_64.zip

# 2. 解压并移动到系统路径
sudo unzip specstory-linux-x86_64.zip
sudo mv specstory /usr/local/bin
sudo chmod +x /usr/local/bin/specstory
```

> [!warning] **常见错误：wget 下载 HTML 页面**
> - 直接用 wget 访问 `github.com` 只会下载到 HTML 页面
> - 必须使用完整的 GitHub Release 链接
> - 确保下载的是实际文件的 URL（以 `.zip` 或 `.tar.gz` 结尾）

> [!danger] **缺少 unzip 命令**
> - 如果出现 `zsh: command not found: unzip` 错误
> - 说明系统根本没有安装 unzip 工具
> - 解决方法：
> ```bash
> sudo apt update
> sudo apt install -y unzip
> ```

> [!tip] **Linux 通用排查原则**
> - 遇到 `command not found` 错误时，90% 的原因是工具根本没有安装
> - 先尝试安装，而非怀疑系统配置问题

### 平台支持

| 平台 | 支持状态 |
|------|---------|
| macOS | ✅ 原生支持 |
| Linux | ✅ 原生支持 |
| Windows (Win32/PowerShell/CMD) | ❌ 不支持 |
| Windows + WSL | ✅ 完全支持 |

## 工作原理

SpecStory 的核心设计理念是**以父进程方式启动 AI agent**：

```
终端 → SpecStory (父进程) → Claude/其他 agent (子进程)
                  ↓
            监听 stdin/stdout
                  ↓
            实时保存为 markdown
```

### 记录粒度

> [!info] **重要概念：agent 级别 vs 终端级别**
> - SpecStory 的记录粒度是 **agent 级别**，而非终端级别
> - 它不是记录终端里发生的一切
> - 而是记录某个被它启动的 agent（如 cc、Gemini、codex）的输入输出

这意味着：

- ✅ 使用 `specstory run claude` 启动的会话会被记录
- ❌ 直接用 `claude` 命令启动的会话不会被记录

## 基本使用

### 启动会话

```bash
# 启动 Claude Code 并记录
specstory run claude

# 启动 Codex CLI 并记录
specstory run codex
```

### 创建命令别名

如果觉得每次输入 `specstory run claude` 太长，可以创建 alias：

```bash
# 添加到 ~/.zshrc
echo "alias cc='specstory run claude'" >> ~/.zshrc
source ~/.zshrc

# 之后只需输入
cc
```

> [!warning] **Alias 的作用域限制**
> - Alias 只在**交互式 shell** 里生效
> - 在脚本里使用 alias 可能找不到命令
> - 这是 shell 的设计特性，需要区分交互和脚本环境

### 高级别名配置

如果需要启动带参数的 Claude（如跳过权限检查）：

```bash
# 在 ~/.zshrc 中添加
alias ccy='specstory run claude -c "claude --dangerously-skip-permissions"'
```

## 同步历史会话

SpecStory 提供了 `sync` 命令来同步旧对话：

```bash
specstory sync
```

### 同步机制

SpecStory 的同步模型有三层状态：

1. **本地 session**：Claude Code 原始会话（jsonl 文件）
2. **本地 markdown**：`.specstory/history/*.md` 文件
3. **SpecStory Cloud**：云端备份（需显式触发）

> [!warning] **sync 命令注意事项**
> - 需要在项目根目录下运行
> - SpecStory 只会查看当前目录，不会跨目录搜索
> - 如果本地 jsonl 文件不存在，则同步不到任何内容

## Resume 功能

Claude Code 的 resume 会话功能与 SpecStory 的启动方式**无关**：

- Resume 是 Claude Code 自身提供的能力
- 无论是否通过 SpecStory 启动，resume 功能都可以正常使用
- SpecStory 只是负责记录会话内容，不干扰 Claude 自身功能

## 总结

SpecStory 是 AI 辅助开发的知识管理利器：

1. **自动记录**：以父进程监听方式实时捕获对话
2. **多平台支持**：macOS/Linux 原生，Windows 通过 WSL
3. **灵活扩展**：支持 Claude、Codex、Gemini 等多种 agent
4. **知识沉淀**：所有会话保存为 markdown，便于检索和复用

掌握 SpecStory，让你的每一次 AI 对话都成为可复用的知识资产。

---

**相关文章**：[[Claude Code 基础使用指南]] | [[CLAUDE.md 规范管理]] | [[WSL/Linux 环境配置]]
