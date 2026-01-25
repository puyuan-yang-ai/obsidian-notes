# 容器化 AI 开发：从环境搭建到 Agent 自动化工作流

## 宿主机与容器的职责分工

> [!tip]
> **一句话总结**
>
> **代码管理在宿主机，代码运行在容器。**

| 职责 | 宿主机 | 容器 |
|------|--------|------|
| 代码编辑 | VS Code/Cursor | - |
| Git 操作 | git pull, git commit | - |
| 运行代码 | - | python xxx.py |
| 安装依赖 | - | pip install |
| 构建索引 | - | build-index |

## Volume Mount：共享代码目录

宿主机改代码为什么容器内立刻生效？因为 **Volume Mount（卷挂载）** 让宿主机和容器共享代码目录。

```
宿主机代码目录 ←→ 容器内代码目录（实时同步）
```

## Claude Code CLI：宿主机还是容器？

使用 Claude Code CLI 时，有两个部署选择：

> [!question]
> **Claude Code CLI 应该放在哪里运行？**
>
> 取决于使用场景和安全需求

### 方案一：宿主机（推荐默认）

适合场景：主要是人盯着它协作（会看它的 diff、命令、测试输出）

### 方案二：容器内

适合场景：
- 想开更高权限/更自动（比如跳过权限提示、无人值守跑较久任务）
- 担心 agent 误删文件、乱装依赖、改坏系统环境

> [!warning]
> **容器内运行的安全建议**
>
> 只挂载项目目录，别把整个 home、SSH 私钥目录、系统敏感路径都挂进去——越少越安全。

## 让 Claude Code 进入容器执行命令

如果 CC 在宿主机执行，但需要去容器里面执行任务，需要在项目根目录的 `CLAUDE.md` 里告诉 Claude Code：

```markdown
运行命令时请使用 docker exec -it my_container <command>
```

## Tmux：终端复用神器

### 进入复制模式

```bash
Ctrl+b 然后按 [
```

进入复制模式后：
- `PgUp` / `PgDn` 翻页
- `q` 或 `Esc` 退出复制模式

## Mini-SWE-Agent：串行优化工作流

> [!info]
> **Mini-SWE-Agent 的架构**
>
> 一个**单 Agent、多轮迭代**的优化 workflow，不是多 Agent 并行。

### 完整流程

```
Phase 1: 硬件分析 (rocminfo, rocm-smi)
    ↓
Phase 2: 代码分析 + MCP query 查询优化知识
    ↓
Phase 3: 跑 baseline benchmark
    ↓
Phase 4: 提出多个优化方向，按优先级排序
    ↓
Phase 5: 循环 {
    选择当前最高优先级的优化方向
    → 实现代码修改
    → 测试验证
    → 对比 baseline
    → 根据结果调整优先级或提出新方向
}
    ↓
Phase 6: 提交最终结果
```

Agent 会先分析问题，提出多个优化方向，然后按优先级依次尝试每个方向。每次只做一个优化，测试效果，再决定下一步。

## 参数优先级的业界标准

> [!success]
> **参数配置的黄金法则**
>
> **显式参数 > 配置文件 > 代码默认值**

| 优先级 | 来源 | 说明 |
|--------|------|------|
| 最高 | 显式参数 | 用户主动传入的值（命令行参数或函数调用） |
| 中等 | 配置文件 | YAML/TOML 等配置文件中定义的值 |
| 最低 | 代码默认值 | 函数签名中的默认参数 |

官方推荐的 `CLAUDE.md` 用途是写入：**进入容器以及运行程序的指令**。

## 依赖安装与索引构建

以下命令应该在**容器内**执行：

```bash
# 依赖要装到容器的 Python 环境里
pip install rank-bm25>=0.2.2

# 需要容器内的 sentence-transformers、faiss 等依赖来生成 embedding
amd-ai-devtool maintenance build-index --force
```

> [!tip]
> **build-index 的 --force 参数**
>
> 不需要提前手动删除缓存文件，`--force` 参数会自动覆盖所有索引文件。

## 容器化开发的核心优势

1. **环境隔离**：宿主机不受污染
2. **快速切换**：不同项目用不同容器
3. **版本一致**：团队环境统一
4. **安全可控**：限制 agent 的操作范围

## 相关文章

- [[RAG 知识库：从 Frontmatter 到 FAISS 索引的完整链路]]
- [[RAG 混合检索：从 RRF 融合到 Rerank 精排序]]
- [[Git 分支管理与代码回滚完全指南]]
