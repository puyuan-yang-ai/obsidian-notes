# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

你是中国人，默认用中文回复。

## 仓库概述

这是一个 Obsidian Vault 知识库仓库，结合了 AI 辅助开发工具链和自动化脚本。主要用途包括：
- 知识管理和笔记归档（Obsidian）
- 微信文章下载和 PDF 转换
- AI 辅助开发（Spec Workflow + SpecStory）
- 自动化工作流

## Python 脚本使用

### 文章下载工作流

仓库包含两步文章处理流程：

**Step 1: 批量下载微信文章为 HTML**
```bash
python step1_download_article.py
```
- 从 `2026-01-02.md` 读取链接列表（每行一个链接）
- 输出到 `1_downloaded_html/` 目录
- 自动清理 URL 参数（如 poc_token）
- 失败自动重试 3 次
- 跳过已存在的文件
- 生成日志文件 `log_download_*.txt`

**Step 2: 批量转换 HTML 为 PDF**
```bash
python step2_html_to_pdf.py
```
- 扫描 `1_downloaded_html/` 中的所有 HTML 文件
- 输出到 `2_transformed_pdf/` 目录
- 使用 Playwright 渲染（A4 格式，带背景）
- 等待所有网络请求完成（包括图片）

### 依赖要求

Python 脚本需要：
- `playwright` - HTML 转 PDF
- `playwright browsers install chromium` - 首次使用需安装浏览器

## 代码更新工作流

### 自定义命令：/commit-rebase-push

使用 `/commit-rebase-push` 技能进行代码提交：
1. 运行 git status 和 git diff，总结改动点
2. 生成符合 Conventional Commits 的 commit message
3. 精确 add 相关文件（不要 `add .`）
4. 执行 git commit
5. 执行 git pull --rebase（如有冲突提示用户）
6. push 到远程分支
7. 输出摘要

### 文件更新原则

**重要**：文件更新使用增量方式，不要覆盖已有内容。
- 修改文件前先用 `Read` 工具读取现有内容
- 使用 `Edit` 工具进行精确修改
- 保留文件中已有的其他内容

## AI 辅助开发工具

### Spec Workflow (.spec-workflow/)

软件开发规范管理系统，提供：
- **templates/** - 文档模板（requirements, design, tasks, product, tech, structure）
- **specs/** - 规范文档存储
- **steering/** - 指导文档（产品、技术、架构）
- **approvals/** - 审批请求管理

使用 MCP 工具：
- `mcp__spec-workflow__spec-workflow-guide` - 获取完整工作流指南
- `mcp__spec-workflow__spec-status` - 查看规范进度
- `mcp__spec-workflow__log-implementation` - 记录实现详情
- `mcp__spec-workflow__approvals` - 管理审批请求

### SpecStory (.specstory/)

AI 对话历史管理：
- **history/** - 自动保存的 AI 编码会话记录
- 通过 @ 引用可以捕获历史上下文
- 可搜索历史提示词和代码片段

搜索排除：`.specstory/*` 已在 `.cursorindexingignore` 中排除

## 目录结构

```
obsidian-notes/
├── .claude/               # Claude Code 配置
│   ├── commands/          # 自定义命令
│   └── settings.local.json
├── .spec-workflow/        # 规范工作流（已 gitignore）
├── .specstory/            # AI 对话历史（已 gitignore）
├── 润色后的文章/          # 知识库（按主题分类）
├── anki原始导出笔记/      # Anki 笔记备份
├── 1_downloaded_html/     # 文章下载临时目录（已 gitignore）
├── 2_transformed_pdf/     # PDF 转换临时目录（已 gitignore）
├── step1_download_article.py
├── step2_html_to_pdf.py
└── download_with_images.py
```

## 版本控制

### .gitignore 注意事项

以下内容不在版本控制中：
- Python 文件和虚拟环境
- PDF 和日志文件
- 下载目录（1_downloaded_html/, 2_transformed_pdf/）
- IDE 配置（.vscode/, .obsidian/）
- AI 工具目录（.spec-workflow/, .specstory/）

### Git 操作权限

Claude Code 已配置以下权限（`.claude/settings.local.json`）：
- 完整的读写、搜索权限
- Git 操作权限（commit, push, pull, stash）
