# SpecStory：AI对话历史的可追溯管理方案

## 什么是 SpecStory

在 AI 辅助开发日益普及的今天，开发者面临着一个普遍的痛点：与 AI 的对话历史往往散落在各个工具中，难以追溯和复用。上午用 Cursor 写界面，下午用 [[Claude Code]] 写逻辑，晚上想回查当时的决策思路时却发现无从下手。这就是典型的"工具碎片化"问题。

SpecStory 的核心作用正是解决这一问题——它能够自动在本地和云端保存对话历史，实现开发过程的**可追溯性**和**可移植性**。

> [!info]
> **核心价值**：
> - 对话历史自动保存为 Markdown 格式
> - 支持跨工具统一管理（Cursor + Claude Code）
> - 云端同步后可集中搜索、团队共享

## CLI 版本：透明封装的设计理念

SpecStory CLI 的本质是一个"包装器"（Wrapper）程序。当你运行 `specstory run claude` 时，SpecStory 会启动一个监听进程并在其内部运行 Claude Code，透明转发所有指令给 Claude Code，同时将输出实时反馈给你——使用体验与直接用 claude 几乎无异。

这种设计带来了一个重要的便利：**不需要额外开启 Auto Save**。CLI 本身就是为了自动化保存设计的，会自动监听并实时保存终端会话到 `.specstory/history/` 目录。

### 运行位置的重要性

```bash
# 在具体的项目根目录下运行
specstory run claude
```

运行位置直接决定了生成的 Markdown 会存放到哪个项目的 `.specstory/` 文件夹下。因此，如果一个项目同时使用了 Cursor 和 Claude Code，确保两者打开的是**同一个项目根目录**——这样它们才会共用一个 `.specstory` 数据库，最终呈现的是一份完整且连贯的项目开发日记。

### 安装与验证

将 SpecStory 可执行文件放入 `/usr/local/bin/` 的好处是确保命令在任何项目目录下都可用。安装完成后，可以通过以下命令验证：

```bash
specstory --version
```

看到版本号即说明安装成功。

> [!tip]
> **Windows 用户推荐使用 WSL**（Windows Subsystem for Linux）来运行 SpecStory CLI，以获得最佳兼容性。

## 历史同步：补救性导出与云端备份

### 批量导出历史对话

如果你在安装 SpecStory 之前就已经在使用 Claude Code，那些历史记录并不会丢失。使用以下命令可以批量导出：

```bash
specstory sync
```

该指令会扫描 Claude Code 的本地原始日志（通常是 JSONL 格式），并将旧对话补救性地转换为 Markdown 文件，存放在当前项目的 `.specstory/history/` 目录下。

### 云端同步的两大价值

`specstory sync` 不仅进行本地转换，还会将生成的 Markdown 上传到 SpecStory Cloud，带来两大核心好处：

**集中搜索**：同步后可以在云端通过关键词或语义搜索，跨项目查找以前的 AI 对话细节，方便回溯当时为什么要这样写代码。

**团队共享**：在团队环境中，通过 sync 指令可以将 AI 辅助开发路径共享给同事，方便团队成员协作和理解代码意图。

> [!warning]
> 如果 Cursor 和 Claude Code 打开的项目根目录不一致，它们的对话历史会分散存储到不同的 `.specstory` 文件夹中，无法实现统一管理。

## 恢复旧会话与持续监听

当你需要基于某个旧会话继续工作时，首先运行 `specstory run claude` 启动 SpecStory 包装的 Claude，进入界面后输入内部指令 `/resume`，然后在 Claude 交互菜单中选择旧会话。SpecStory 此时会开始记录后续的所有对话。

## 桌面端应用：BearClaude

除了 CLI 版本，SpecStory 还提供了名为 **BearClaude** 的官方 macOS 桌面应用。这是一款基于"Spec-first"（规格说明优先）开发理念的工具，旨在通过编写 Spec 文档来引导 AI 生成代码。

## 消除工具碎片化的实际意义

SpecStory 解决的不是单一工具的问题，而是整个 AI 辅助开发生态的碎片化困境。即便你上午用 Cursor 写界面，下午用 [[Claude Code]] 写逻辑，晚上打开 SpecStory，你看到的是一份完整且连贯的项目开发日记。

这种可追溯性对于个人知识管理、团队协作交接、以及代码决策的审计都有着不可替代的价值。
