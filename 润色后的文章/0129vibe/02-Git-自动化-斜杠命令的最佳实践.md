# Git 自动化：斜杠命令的最佳实践

在 AI 辅助编程中，Git 操作是一个非常适合自动化的领域。尤其是 **Commit Message 的生成**，AI 可以大大减少心智负担和操作成本——它能够自动总结 diff，按 Conventional Commits 规范生成，写清 scope/reason/breaking change，并保持语言统一。

## /commit 命令：只负责"写信息 + 执行提交"

> [!tip]
> **/commit 的设计原则**
> - 只负责"写提交信息 + 执行 commit"
> - 默认基于 staged（已暂存的文件）
> - 不包含 add 操作——更安全，需要自己先决定要提交哪些文件

一个精心设计的 `/commit` 命令通常包含以下步骤：

```bash
# /commit 执行流程
# ① 读取 staged diff
# ② 生成 commit message
# ③ 执行 git commit
```

### 为什么不包含 add 操作？

如果 `/commit` 包含了 `add` 操作，容易把不想提交的文件也带进去——比如密钥、配置文件、临时测试文件、debug 改动等。**开发者应该自己先决定要提交哪些文件**，然后再让 AI 帮忙生成规范的提交信息。

这种设计遵循了 **"人是决策者，AI 是执行者"** 的原则：AI 最适合的是基于你已经做出的决策（选择哪些文件要提交）来生成规范的描述，而不是替你做决策。

### 为什么不包含 pull rebase 和 push？

同样，`/commit` 也不包含 `pull rebase` 和 `push` 操作，原因在于：

1. **pull rebase 有时需要人为介入解决冲突**——这不是 AI 可以自动处理的
2. `pull rebase` 和 `push` 指令非常简单，手动输入不费事

将 Git 操作拆分成多个独立的命令，让每个命令只做一件事，是 Unix 哲学的体现，也更符合"**原子性操作**"的最佳实践。

## /ship 命令：企业级的发布流程

如果说 `/commit` 是日常开发的利器，那么 `/ship` 就是企业级发布的标准流程。

> [!info]
> **/ship 的完整流程**
> - git status 检查
> - git fetch
> - 判断 ahead/behind
> - 跑 tests
> - push

一个完善的 `/ship` 命令会：

1. 检查工作区状态（git status）
2. 获取远程最新状态（git fetch）
3. 判断本地分支与远程分支的关系（ahead/behind）
4. 运行测试套件确保质量
5. 推送到远程仓库

> [!warning]
> **如果 /ship 只是做 git push**
> 如果仅仅是只做 git push，不值得用 /ship。只有当它包含了完整的发布前检查流程时，才能真正发挥价值。

## 主流做法：命令的职责边界

基于社区的实践，斜杠命令的职责边界可以这样划分：

| 命令 | 职责范围 | 不包含的原因 |
|------|----------|--------------|
| `/commit` | 写信息 + 执行 commit | add 操作需要人工决策 |
| `/ship` | 检查 + 测试 + push | pull rebase 可能需要冲突解决 |
| `git add` | 手动选择文件 | 避免误提交敏感文件 |
| `git pull rebase` | 手动执行 | 冲突解决需要人工介入 |

> [!success]
> **AI 在 Git 操作中的最佳角色**
> - 生成规范的 Commit Message ✅
> - 总结 diff 变更 ✅
> - 执行原子性的 git 操作 ✅
> - 替代人工决策 ❌
> - 自动处理冲突 ❌

## 实战示例

假设你已经完成了代码修改，并使用 `git add` 选择了要提交的文件：

```bash
# 手动选择要提交的文件
git add src/components/Button.tsx
git add tests/Button.test.ts

# 执行 /commit 命令
# AI 会自动生成类似如下的 commit message：
# feat(button): add loading state prop
#
# - Add isLoading prop to Button component
# - Update styles to show loading indicator
# - Add unit tests for loading state
```

当你准备好发布时：

```bash
# 执行 /ship 命令
# 它会自动完成以下检查：
# 1. git status 检查
# 2. git fetch 获取最新
# 3. 运行测试
# 4. push 到远程
```

## 总结

Git 自动化的精髓在于：**让 AI 做它擅长的事（生成规范的文本、执行原子操作），让人保留决策权（选择文件、解决冲突）**。

一个设计良好的斜杠命令系统，应该遵循 Unix 哲学——每个命令只做一件事，并做好。`/commit` 专注于提交信息的生成，`/ship` 专注于发布流程的自动化，而不是试图把所有 Git 操作都塞进一个命令里。

## 相关文章

- [[从想法到代码：Plan Mode 的正确打开方式]]
- [[上下文管理：如何让 AI 更懂你的项目]]
