---
description: 智能 git commit - 分析暂存更改并生成符合规范的 commit message
---

# Smart Git Commit

请帮我完成一次智能 git commit，按以下步骤执行：

## Step 1: 检查 Git 状态

执行 `git status --porcelain` 并分析输出：

- 如果输出为空：告诉我 "没有任何更改需要提交"，然后**停止**
- 如果没有 staged 文件（没有以 `A`/`M`/`D`/`R`/`C` 开头的行）：
  - 列出未暂存的修改和未跟踪的文件
  - 提示我需要先使用 `git add <file>` 暂存文件
  - **不要自动执行 git add**，然后**停止**
- 如果有 staged 文件：继续下一步

## Step 2: 获取 Staged 内容

执行 `git diff --staged` 获取已暂存的更改详情。

## Step 3: 生成 Commit Message

根据 diff 内容，生成符合 **Conventional Commits** 规范的 commit message：

格式：`<type>(<scope>): <description>`

type 类型：
- `feat`: 新功能
- `fix`: 修复 bug  
- `docs`: 文档更新
- `style`: 代码格式（不影响功能）
- `refactor`: 重构
- `perf`: 性能优化
- `test`: 测试相关
- `chore`: 构建/工具/依赖相关

规则：
- description 简洁明了，使用英文
- 首字母小写，不加句号
- scope 可选，表示影响的模块/目录

## Step 4: 执行 Commit

**直接执行**，无需确认：

```bash
git commit -m "<生成的message>"
```

展示 commit 结果（包含 commit hash、分支、修改文件数、commit msg等信息）。

$ARGUMENTS

