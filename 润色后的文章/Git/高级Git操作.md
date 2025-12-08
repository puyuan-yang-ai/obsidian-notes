# 高级Git操作

掌握Git的高级操作能够让你更加精确地控制版本历史，处理复杂的协作场景，并保持项目的整洁性。这些高级技巧包括rebase、stash、reset、revert等操作。

## Rebase vs Merge：两种合并策略

### 理解Rebase和Merge的区别

在Git中，当需要整合两个分支的更改时，有两种主要策略：merge和rebase。

#### Merge操作
```bash
git pull                    # 等同于 git fetch + git merge
```

**Merge的工作原理：**
1. 从远程仓库获取最新内容
2. 将本地分支与远程分支进行合并
3. 创建一个新的合并提交（Merge commit）

**Merge的缺点：**
- 生成合并提交，使提交历史混乱
- 充满类似"Merge branch 'main' of ..."的信息
- 频繁推送时历史会变得复杂，难以追溯

#### Rebase操作
```bash
git pull --rebase           # 等同于 git fetch + git rebase
```

**Rebase的工作原理：**
```bash
# 假设场景：
# 远程：A -- B -- C (origin/main)
# 本地：A -- B -- D (你的提交)

# git pull --rebase 会：
# 1. fetch：获取远程最新提交C
# 2. rebase：暂时收起你的本地提交D，使分支回到A
# 3. 将本地分支快进到远程最新提交C
# 4. 再把提交D应用到C之上，生成新的提交D'
```

**Rebase的优势：**
- 保持线性历史，提交记录是一条干净直线
- 清晰易读，便于追踪和理解每个提交的变动
- 避免了多余的合并节点

### Rebase的核心思想

Rebase（变基）的核心思想是：**把自己的提交应用到远程分支的最新版本之上**。

### Rebase vs Merge的提交历史对比

**使用git pull（merge）的结果：**
```
A -- B -- C ---- M
           \     /
             D
```
其中M是新生成的合并提交，不包含任何功能性修改。

**使用git pull --rebase的结果：**
```
A -- B -- C -- D'
```
其中D'是你的提交被"移动"到C之后，内容与D相同但父提交由A变为C。

### 推荐使用Rebase的场景

> [!tip] 为什么一直推荐使用git pull --rebase？
> 因为它能保持提交历史线性、清晰，避免了大量不必要的"Merge branch..."提交记录，这在团队协作中能保持更加整洁的历史，便于追踪和理解每个提交的变动。

## Stash：临时保存工作进度

### 理解Stash的作用

`git stash`命令允许你临时保存工作区和暂存区的未提交代码，使工作区回到干净的状态。当你正在进行的工作尚未完成且需要切换到其他任务时，stash非常有用。

### Stash的基本用法

```bash
# 保存当前工作进度
git stash

# 保存并添加描述信息
git stash save "正在开发用户登录功能"
```

### Stash的数据流向

Stash会将以下内容保存到Git的一个独立stash区域：
- **工作目录中的修改**：所有未提交的文件更改
- **暂存区中的修改**：已经通过`git add`添加的更改

> [!note] Stash与暂存区的区别
> Stash Changes不会将更改存储到暂存区，而是存储到Git的一个独立的stash区域。暂存区和stash区域是两个不同的概念。

### Stash的使用场景

1. **紧急任务切换**：当前功能未完成，需要紧急修复bug
2. **分支同步**：在拉取远程更新前需要清理工作区
3. **实验性修改**：想尝试某些修改但不想提交

### Stash操作管理

```bash
# 查看所有stash列表
git stash list

# 恢复最近的stash
git stash pop

# 应用特定的stash（不删除）
git stash apply stash@{0}

# 删除特定的stash
git stash drop stash@{0}

# 清空所有stash
git stash clear
```

### 在Pull操作中使用Stash

在执行`git pull`前使用stash的作用是临时保存未完成的更改，减少合并冲突的风险。

```bash
# 完整的工作流程
git stash                    # 保存当前工作
git pull --rebase           # 拉取远程更新
git stash pop               # 恢复之前的工作
```

## Reset：重置提交历史

### Reset的三种模式

`git reset`命令有三种不同的模式，分别影响不同的区域：

1. **--soft**：只移动HEAD指针，保留工作区和暂存区的更改
2. **--mixed**（默认）：移动HEAD指针并清空暂存区，保留工作区更改
3. **--hard**：移动HEAD指针、清空暂存区、恢复工作区到指定状态

### Reset --hard：完全回退

```bash
git reset --hard HEAD~1
```

这个命令的作用是：
- **本地仓库回到上次提交之前的状态**
- **删除暂存区中的所有更改**
- **恢复工作区文件到上次提交的状态**

### Reset的使用场景

#### 撤销最近的提交

```bash
# 保留更改，只是撤销提交
git reset --soft HEAD~1

# 撤销提交和暂存，保留工作区更改
git reset --mixed HEAD~1

# 完全撤销提交和所有更改
git reset --hard HEAD~1
```

#### 以远程仓库为主重置本地

当你确定远程版本是"正确的"，想要完全放弃本地修改时：

```bash
git fetch origin                    # 获取远程最新状态
git reset --hard origin/main        # 强制重置为远程状态
```

> [!warning] Reset的注意事项
> `git reset --hard`会丢失所有未提交的修改，使用前请确保已经备份或确认不需要这些修改。

## Revert：安全的撤销操作

### Revert vs Reset的区别

**Reset**：重置历史，删除某些提交，改变提交历史
**Revert**：创建新的提交来撤销之前的更改，保留完整历史

### Revert的优势

在团队合作中使用`git revert`而不是`git push --force`的优势：
- **避免重写公共历史**：不改变已有的提交记录
- **保留历史记录**：所有修改都被记录，便于追踪
- **安全性更高**：不会影响其他人的工作

### Revert的使用

```bash
# 撤销最近的提交
git revert HEAD

# 撤销特定的提交
git revert <提交哈希值>

# 撤销多个提交
git revert <起始提交>..<结束提交>
```

### 团队协作中安全处理错误的提交

在团队合作中，当发现错误的提交时，推荐使用：

```bash
git revert <错误提交的哈希值>
git push origin <分支名>
```

而不是使用强制推送。

## 版本管理与历史操作

### 查看提交信息

```bash
# 查看当前分支的提交哈希值
git rev-parse --short HEAD
git log -1 --oneline
git branch -v
```

这些命令的作用：
- **git rev-parse --short HEAD**：显示当前提交的简短哈希值
- **git log -1 --oneline**：显示最近一条提交的简洁信息
- **git branch -v**：显示分支及其最新提交信息

### 简化命令别名

可以设置Git别名来简化常用命令：

```bash
# 设置查看提交哈希的别名
git config --global alias.hash "rev-parse --short HEAD"

# 之后可以直接使用
git hash
```

### 历史记录比较

```bash
# 比较两个提交之间的差异
git diff <提交1> <提交2>

# 比较当前工作区与指定提交的差异
git diff HEAD
```

## 冲突解决策略

### Rebase时的冲突处理

在执行`git pull --rebase`时遇到冲突，需要注意：

```bash
# --ours：保留当前rebase基准上的代码（原分支的内容）
# --theirs：保留当前正在应用的提交（你的本地提交）
```

> [!note] 为什么rebase时--ours表示远程分支？
> 因为rebase的过程是先切到远程分支作为新的基础（因此是"ours"），再把你的本地提交逐个应用上去（因此成了"theirs"）。这可能看起来反直觉，但理解rebase的工作原理后就很清楚了。

### 解决rebase冲突的步骤

1. **识别冲突文件**：Git会标记出冲突
2. **手动解决冲突**：编辑文件，选择保留的内容
3. **标记已解决**：使用`git add`标记冲突已解决
4. **继续rebase**：使用`git rebase --continue`
5. **处理下一个冲突**：如果还有冲突，重复上述步骤

## 高级工作流程

### 以远程为主的本地重置

当你确定GitHub上的版本是最新正确版本，而本地分支有问题时：

```bash
git fetch origin
git reset --hard origin/你的分支名
```

**为什么使用这两个指令？**
1. `git fetch origin`：更新本地对远程分支的最新引用，确保基于的是GitHub最新版本
2. `git reset --hard`：直接将本地重置为远程状态，避免合并本地的错误内容

> [!warning] 不能使用pull的情况
> 如果想以远程分支为主、本地分支不要了，不应该使用pull，因为pull会把远程修改合并到本地，应该使用reset，因为reset会直接将本地重置为远程状态。

### 多设备同步的最佳实践

在使用两台设备操作同一个仓库时：

1. **理想状态**：两边分支的哈希值一致，表示代码版本完全同步
2. **同步检查**：使用`git fetch origin && git status`确认状态
3. **哈希一致的含义**：两个仓库的代码内容就100%同步

```bash
# 快速比较远程仓库与本地仓库内容是否一致
git fetch origin && git status
```

## 高级操作的最佳实践

### 版本控制的基本原则

1. **版本创建时机**：只有在`git commit`时才会产生新的版本；`git add`、`git push`、`git pull`都不会创建版本
2. **历史保护**：在团队协作中避免重写公共历史
3. **操作前检查**：执行高级操作前先用`git status`检查当前状态

### 安全的高级操作流程

1. **备份当前状态**：在执行危险操作前考虑使用stash备份
2. **确认操作影响**：理解每个命令对不同区域的影响
3. **逐步执行**：复杂操作分解为多个步骤，便于回滚

### 选择合适的操作方式

| 场景 | 推荐操作 | 原因 |
|------|----------|------|
| 撤销个人提交 | `git reset` | 影响范围小，不影响他人 |
| 撤销公共提交 | `git revert` | 不重写历史，保留记录 |
| 同步远程更新 | `git pull --rebase` | 保持线性历史 |
| 临时切换任务 | `git stash` | 保存工作进度，保持工作区干净 |

掌握这些高级Git操作后，你就能更自信地处理复杂的版本控制场景，保持项目的整洁性和团队协作的效率。这些技巧在[[团队协作最佳实践]]中将发挥重要作用。