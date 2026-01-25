# Git 分支管理与代码回滚完全指南

## 分支切换的正确姿势

在 Git 操作中，切换分支是一个高频动作。但很多人在使用 `git switch` 时容易踩坑。

> [!danger]
> **分支名中的斜杠不是路径分隔符**
>
> 分支名 `feature/bge-chunker-extended-kb` 中的 `/` 是分支名的一部分，让分支在视觉上有层级感，但 Git 并不会把它当成目录结构。

正确的切换命令是使用**完整的分支名**：

```bash
# ✅ 正确 - 使用完整分支名
gsw feature/bge-chunker-extended-kb

# ❌ 错误 - 会报错找不到分支
gsw bge-chunker-extended-kb
```

对于远程分支，Git 会自动匹配并创建本地跟踪分支。当看到 `remotes/origin/rag_summarize_original` 时，直接使用简写即可：

```bash
# 直接使用分支名，Git 会自动处理
gsw rag_summarize_original
```

## 终端与 IDE 的分支显示差异

开发中常会遇到一个困惑：终端显示的分支和 VS Code 底部显示的分支不一样。

> [!info]
> **显示差异的本质**
>
> - **终端显示的分支**：当前真实分支
> - **VS Code 底部显示**：只是 UI 显示，可能过期未刷新

以终端显示的分支为准，因为终端是直接和 Git 仓库交互的，显示的是实时状态。VS Code 底部的分支显示只是个 UI 问题，不影响实际操作。

## Atomic Commits：原子提交的艺术

> [!tip]
> **Atomic Commits 的核心原则**
>
> 一个 commit = 一个逻辑变更

每个 commit 应该具备以下特性：
- **独立完整**：能单独回滚
- **可运行**：每个 commit 后代码应该能正常运行
- **可审查**：别人能单独理解这个 commit 做了什么

常见误区是"每个小功能一个 commit"，正确的理解是"每个**逻辑完整**的变更一个 commit"。

## 浅克隆：节省空间与时间

> [!note]
> **浅克隆（Shallow Clone）**
>
> 只下载最新版本，不下载完整历史记录

```bash
# 完整克隆（默认）
git clone git@github.com:xxx.git

# 浅克隆 - 只下载最新 1 次提交
git clone --depth 1 git@github.com:xxx.git
```

浅克隆特别适合 CI/CD 场景或临时拉取代码进行测试的情况，能大幅减少下载时间和磁盘占用。

## 代码回滚的三层境界

当需要撤销修改时，Git 提供了不同层级的 `restore` 指令。

### 境界一：恢复单个文件

```bash
# 只撤销 config.py 的修改，其他保留
git restore config.py

# 撤销整个目录的修改
git restore src/utils/
```

### 境界二：恢复所有修改

```bash
# 恢复工作区所有文件到最新 commit 状态
git restore .
```

### 境界三：同时清空暂存区和恢复工作区

> [!warning]
> **理解 restore 各参数的作用**
>
> | 指令 | 暂存区 | 工作区 |
> |------|--------|--------|
> | `git restore .` | 不变 | 恢复 |
> | `git restore --staged .` | 清空 | 不变 |
> | `git restore --staged --worktree .` | 清空 | 恢复 |

如果执行 `git restore --staged .`：
- ✅ 暂存区被清空（撤销 `git add`）
- ❌ 工作区修改**仍然保留**

要同时清空暂存区 + 恢复工作区，需要：

```bash
# 方式一：两条指令
git restore --staged .  # 清空暂存区
git restore .           # 恢复工作区

# 方式二：合并写法
git restore --staged --worktree .
```

## 相关文章

- [[RAG 知识库：从 Frontmatter 到 FAISS 索引的完整链路]]
- [[RAG 混合检索：从 RRF 融合到 Rerank 精排序]]
- [[容器化 AI 开发：从环境搭建到 Agent 自动化工作流]]
