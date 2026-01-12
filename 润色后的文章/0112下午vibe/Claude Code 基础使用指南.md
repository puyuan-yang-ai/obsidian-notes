# Claude Code 基础使用指南

## 概述

Claude Code 是 Anthropic 官方推出的 CLI 工具，为开发者提供了强大的 AI 辅助编码能力。本文将系统介绍 Claude Code 的核心功能，包括斜杠命令系统、代码提交流程以及与终端的高效集成方式。

## 斜杠命令系统

Claude Code 内置了一套斜杠命令系统，类似于 GitHub CLI 的交互方式，让开发者能够快速执行常见操作。

### 自定义命令创建

> [!info] **斜杠命令定义位置**
> - 所有自定义斜杠命令都需要在 `.claude/commands/` 目录下定义
> - 每个命令对应一个独立的 `.md` 文件
> - 文件名即为命令名（如 `terminal.md` 对应 `/terminal`）

需要注意的是，Claude Code **没有内置** `/terminal` 命令，但提供了 `/terminal-setup` 用于安装终端快捷键配置（如 Shift+Enter 换行等）。如果需要自定义命令，只需在命令文件中编写相应的指令逻辑即可。

### 代码提交工作流

在日常开发中，代码提交是最频繁的操作之一。Claude Code 提供了两种提交方式：

#### 方式一：自定义斜杠命令（推荐）

使用 `/commit-rebase-push` 命令，这是一个封装完整工作流的智能命令：

```bash
/commit-rebase-push
```

该命令会自动执行以下流程：

1. 运行 `git status` 查看当前状态
2. 运行 `git diff` 分析改动内容
3. 根据改动生成符合 [[Conventional Commits]] 规范的提交信息
4. 精确执行 `git add`（**不会**使用 `add .` 避免误提交）
5. 执行 `git commit`
6. 执行 `git pull --rebase`（如有冲突会提示用户）
7. 推送到远程分支

> [!tip] **为什么推荐自定义命令？**
> - 自动生成规范的 commit message
> - 精确控制文件添加范围
> - 自动处理 rebase 流程
> - 减少人为错误

#### 方式二：手动 Git 操作

在正常终端手动输入传统 git 指令：

```bash
git status
git add <files>
git commit -m "message"
git pull --rebase
git push
```

这种方式灵活性更高，但需要开发者手动处理所有步骤。

## Git 插件增强

如果你使用 [[oh-my-zsh]]，其内置的 git 插件提供了大量实用别名。例如：

```bash
# glog 等价于完整命令
glog
# 展开后实际执行：
git log --oneline --decorate --graph
```

> [!note] **常用 git 别名**
> - `gst` → `git status`
> - `gaa` → `git add --all`
> - `gcmsg` → `git commit -m`
> - `gp` → `git push`

合理使用这些别名可以显著提升终端操作效率。

## 终端集成与快捷键

Claude Code 可以与系统终端深度集成。通过 `/terminal-setup` 命令，可以配置各种快捷键，例如：

- **Shift+Enter**：在输入中换行（避免直接发送）
- **Ctrl+C**：中断当前生成
- **上下箭头**：浏览历史命令

这些快捷键让 CLI 交互体验更接近传统终端工具，提升操作流畅度。

## 总结

Claude Code 的斜杠命令系统是其核心优势之一：

1. **可扩展性**：通过 `.md` 文件轻松定义自定义命令
2. **自动化工作流**：如 `/commit-rebase-push` 封装复杂操作
3. **终端集成**：支持快捷键和插件增强
4. **规范化输出**：自动生成符合规范的 commit message

掌握这些基础功能，能够让你的 AI 辅助开发体验更加高效流畅。

---

**相关文章**：[[SpecStory 工具全解]] | [[CLAUDE.md 规范管理]] | [[WSL/Linux 环境配置]]
