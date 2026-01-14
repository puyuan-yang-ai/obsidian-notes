# CLAUDE.md 配置与项目文档体系

## 概述

CLAUDE.md 是 Claude Code 中最重要的配置文件之一，它充当了"给 AI 看的项目规范手册"。理解 CLAUDE.md 的定位、内容组织、加载机制，以及它与项目文档体系的关系，是构建高效 AI 辅助开发工作流的关键。本文将深入解析 CLAUDE.md 的最佳实践，以及如何构建清晰的项目文档结构。

## CLAUDE.md 的正确定位

### 官方定义

根据 Claude Code 官方文档，CLAUDE.md 的定位是包含以下内容：

- **项目规范**（Project conventions）
- **工作流**（Workflows）
- **规则**（Rules）
- **结构**（Structure）
- **命令**（Commands）

> [!info]
> **核心原则**
> CLAUDE.md 应该保持**短小、稳定、低频修改**。它只包含"长期可复用"的工程原则、禁忌、Git workflow、build/test 命令、目录结构与约定。

### 常见误区

一个常见的错误提示词是：

> "总结我们之前对话中的核心决策、代码架构和**未完成的任务**，并更新到 CLAUDE.md 中。"

这种做法的问题在于：它会把 CLAUDE.md 变成长期堆积的任务清单（会过期）和噪声上下文（每次都加载，污染决策）。

> [!danger]
> **禁忌**
> 不要把未完成任务或临时 TODO 写入 CLAUDE.md。这会导致上下文膨胀，每次会话都加载过期的信息。

## 正确的 CLAUDE.md 更新方式

在会话结束时，应该使用类似以下的指令：

```markdown
请回顾本次会话或改动，提炼 3–7 条"长期可复用"的项目规则或约定（包括项目结构的更新、架构决策、新的编码规范、新的工作流、常用命令、测试或提交流程、踩坑提示）。将它们以简短 bullet 合并到 CLAUDE.md 的合适小节中。
不要记录未完成任务或临时 TODO；请把未完成项单独输出为一个清单（不写入 CLAUDE.md）。
```

> [!tip]
> **提炼标准**
> 问自己：这条信息在三个月后是否仍然有用？如果是，才应该写入 CLAUDE.md。

## CLAUDE.md 的加载机制

### 多文件合并加载

Claude Code 启动新会话时，会从当前工作目录开始，**向上逐级查找 CLAUDE.md**，直到文件系统根目录。同时也会加载用户级 memory 文件。

查找路径示例：
```
项目根目录/CLAUDE.md
当前目录/CLAUDE.md
~/.claude/CLAUDE.md（用户级）
```

所有找到的 memory 文件会按"**就近优先**"原则合并：当前目录 > 父目录 > 更上层目录 > 用户级。冲突时后加载（更近）的规则覆盖前面的，同类规则以"最后出现"为准。

> [!note]
> **重要细节**
> Claude Code **不会递归扫描子目录**，只会从当前目录开始向父目录查找。

### 会话启动时的加载行为

一个常见的误解是："Claude Code 会在新 session 开始时把 CLAUDE.md 放进 system prompt 里并永久显示。"

更准确的说法是：Claude Code 会在新 session 开始时"**自动载入 CLAUDE.md 作为上下文**"。它被加载进 Claude 的 context window，而不是形成单一的"超级 system prompt"。

### @ 导入机制

CLAUDE.md 支持使用 `@path/to/file` 语法导入额外文件：

```markdown
See @README for project overview
Git workflow @docs/git-instructions.md
```

> [!warning]
> **加载规则**
> - 只是在文本里"提到"（如"查阅 docs/plans/index.md"）：**不会**自动加载
- 使用 `@` 语法（如 `@docs/plans/index.md`）：**会**加载文件内容

此外，`@` 是**递归显式加载**。如果导入的文件中又包含 `@` 引用，这些被引用的文件也会被加载。例如：

```markdown
# 在提示词中输入
@docs/plans/index.md

# index.md 内容为
See details in @docs/plans/plan-001.md
```

这种情况下，`docs/plans/index.md` 和 `docs/plans/plan-001.md` **都会被加载**。

> [!info]
> **限制**
> `@` 一般是"文件级导入"，**不支持目录递归导入**。如果需要"导入整个目录"的效果，应该用索引文件（如 `docs/plans/index.md`）管理列表。

## 项目文档体系的语义分层

### 文档类型分类

一个健康的文档体系应该清晰地分层：

| 目录 | 语义 | 内容类型 |
|------|------|----------|
| `.claude/` | 给工具看的配置 | settings.local.json、自定义命令 |
| `docs/` | 给人看的项目知识 | architecture.md、api.md、deployment.md |
| `docs/tasks/` | 过程文档（临时） | task.md、plan.md、notes.md |

> [!success]
> **核心原则**
> - 给人看的文档 → `docs/`
> - 给工具看的文档 → `.claude/`

### task.md 和 plan.md 的真实定位

许多用户混淆了文档的定位，这里用土木工程的例子类比：

- **task.md 和 plan.md** = "施工图纸和施工记录"（临时性、过程性）
- **项目文档**（architecture.md、design.md）= "房屋结构说明书"（稳定性、长期性）
- **CLAUDE.md** = "给机器人看的施工规范"（工具性、指令性）

从 task.md 和 plan.md 中提炼稳定知识后，应写入 architecture.md、design.md 或 CLAUDE.md。这个过程叫 **Knowledge Distillation**（工程知识沉淀）。

### 推荐的目录结构

```
docs/
├── plans/              # 功能计划（当前活跃）
│   ├── index.md        # 计划索引
│   ├── 2024-12-feature-login.md
│   └── 2025-01-rag-refactor.md
├── archive/            # 历史计划（已归档）
├── tasks/              # 任务过程文档
│   └── feature-xxx/
│       ├── task.md     # 业务背景和目标（需求）
│       ├── plan.md     # 执行计划（方案）
│       └── notes.md    # 过程记录或实验结果
├── architecture.md     # 架构文档
├── api.md             # API 文档
└── deployment.md      # 部署文档
```

> [!tip]
> **命名规范**
> 计划文档使用日期+功能名的命名方式，如 `2024-12-feature-login.md`，便于按时间排序和归档。

## ultrathink 关键词的工作原理

### 作用范围

与 Claude Code 对话时加入 `ultrathink` 关键词，**只对"当前这一轮请求（这一条消息）"生效**，不是整个会话。

> [!note]
> ultrathink 的真实作用范围是**只影响当前这一条请求**。

### 模型兼容性

`ultrathink` **不是通用的模型能力**，只对 Anthropic 的 Claude 系列模型和 Claude Code 工具链有效。对其他模型（如 GLM、GPT、Qwen、LLaMA 等）通常无效或只会被当作普通文本。

这是因为 Claude Code 工具内置了关键词识别机制——当输入 `ultrathink` 时，系统会识别这个词并分配更大的"思考预算"（更多 tokens、更多内部中间推理等）。这种机制不是模型本身训练出来自动决定的，而是工程上做了显式关键词检测加预算分配逻辑。

## 配置文件的版本控制策略

对于 `.claude/` 目录的版本控制，建议如下：

- **`.claude/settings.local.json`** → 添加到 `.gitignore`（本地个性化设置，不共享）
- **`.claude/commands/`** → 不忽略（团队共享的自定义命令）
- **`.claude/` 其他文件** → 不忽略（团队共享规范和规则）

核心原则是：**只忽略不应共享的"本地或个性化"配置，不忽略"团队共享"规范和规则文件**。

---

## 总结

构建清晰的 CLAUDE.md 配置和项目文档体系，是建立高效 AI 辅助开发工作流的基础。核心要点包括：

1. **CLAUDE.md 定位**：短小、稳定、只包含长期可复用的规则和约定
2. **加载机制**：向上逐级查找、就近优先、支持 `@` 导入
3. **文档分层**：配置文件、项目文档、过程文档各归其位
4. **知识沉淀**：从临时文档中提炼稳定知识，写入适当位置
5. **版本控制**：个性化设置忽略，团队规范共享

掌握这些原则后，你可以构建一个既能服务人类开发者、又能被 AI 工具高效理解的文档体系。

**相关文章**：[[Claude Code 工作空间与文件访问管理]] | [[运行模式与权限控制机制]]
