# RAG 项目架构解析：mini-swe-agent 与混合检索系统

## 概述

在 RAG（检索增强生成）项目中，理解各组件之间的依赖关系和数据流转路径至关重要。本文将深入分析 `mini-swe-agent`、`langchain` 目录与 `amd-ai-devtool` 三者之间的关系，并详细解析从用户命令到检索结果的完整执行流程。

---

## 一、三组件关系与职责划分

### 1.1 核心关系

首先需要明确一个关键事实：**`mini-swe-agent` 仅仅使用了 `langchain` 目录里面的文件和功能，而 `amd-ai-devtool` 仅仅是作为知识库的提供者**。也就是说，`mini-swe-agent` 并没有使用或间接调用 `amd-ai-devtool` 里面定义的函数和 tool。

这三者之间的类比关系可以帮助理解：

| 组件 | 类比 | 说明 |
|------|------|------|
| `amd-ai-devtool` | 图书馆（提供书籍） | 数据源，包含 203 个 .md 知识文件 |
| `langchain/` | 索引系统设计师 | 设计卡片索引系统，负责构建索引 |
| `mini-swe-agent` | 读者 | 带着自己的卡片索引去查书 |

### 1.2 代码来源澄清

一个常见的混淆点是 `HybridRetriever` 和 `KnowledgeTools` 的来源。这两个类**并非**定义在 `langchain` 目录中，而是来自：

```python
mini-swe-agent/src/minisweagent/mcp_integration/langchain_retrieval.py
```

`langchain/` 目录中的 `HybridRetriever` 代码被复制到了 `mini-swe-agent` 内部，因此两者在运行时是独立的代码副本。

---

## 二、langchain 目录的角色

### 2.1 目录职责

`langchain/` 目录是一个纯粹的**检索库**，仅负责以下功能：

| 文件 | 作用 |
|------|------|
| `build_index.py` | 构建索引的工具 |
| `hybrid_retriever.py` | 测试用 |
| `test_search.py` | 测试用 |

> [!info]
> **langchain 目录定位**
> - 仅仅是构建索引的工具
> - **不是运行时依赖**
> - 代码被复制到 mini-swe-agent 中使用

### 2.2 删除影响

如果删除 `langchain/` 目录：

> [!success]
> **结论**：`mini-swe-agent` 仍可正常运行，前提是索引文件已存在。

这是因为 `mini-swe-agent` 运行的是自己复制的代码，只依赖共享的索引文件，不依赖 `langchain/` 目录的源代码。

---

## 三、数据层与缓存结构

### 3.1 缓存目录内容

所有索引数据存储在 `~/.cache/amd-ai-devtool/semantic-index/` 目录下：

```
~/.cache/amd-ai-devtool/semantic-index/
├── index.faiss          # FAISS 索引文件（二进制）
├── index.pkl            # FAISS 元数据（Pickle）
├── bm25_index.pkl       # BM25 模型（Pickle）
└── bm25_documents.pkl   # BM25 文档列表（Pickle）
```

### 3.2 数据流转

```
amd-ai-devtool/
└── knowledge-base/
    └── 203 个 .md 文件  ← 数据源
        ↓ (被读取)
~/.cache/amd-ai-devtool/semantic-index/
├── index.faiss         ← FAISS 索引数据
├── index.pkl           ← 元数据
└── bm25_*.pkl          ← BM25 索引数据
        ↓ (被读取)
mini-swe-agent 的 HybridRetriever.search()
```

> [!note]
> **数据层总结**
> - RAG 项目目前的数据层由 `amd-ai-devtool/` 的知识库和缓存目录的索引文件构成
> - `mini-swe-agent` 只读取索引文件，不访问原始 Markdown 文件

---

## 四、检索流程详解

### 4.1 用户命令执行链路

当用户执行以下命令时：

```bash
mini --mcp -t "优化 HIP 内核"
```

Agent 会生成检索命令：

```bash
@amd:query {"topic": "MI300X optimization"}
```

后续的完整执行流程如下：

```
1. MCPEnabledEnvironment.execute()
   └─ 检测到 "@amd:" 前缀
   └─ 调用 _execute_mcp("query {...}")

2. _execute_mcp()
   └─ 解析：tool_name="query", args={"topic": "..."}
   └─ 调用：self._tool_map["query"](args)
       ↓
   self._knowledge_tools.query(args)
   ⭐ KnowledgeTools 是 mini-swe-agent 自己定义的类

3. KnowledgeTools.query() [mini-swe-agent 的代码]
   └─ 调用：self.retriever.query(topic, ...)
       ↓
   self._retriever 是 HybridRetriever 的实例

4. HybridRetriever.query() [复制自 langchain/]
   └─ 并行检索：
      ├─ FAISS 检索（读取文件）
      │  └─ 读取：~/.cache/amd-ai-devtool/semantic-index/
      │     ├─ index.faiss  ⭐ 只是读取二进制文件
      │     └─ index.pkl    ⭐ 只是读取 pickle 文件
      │
      ├─ BM25 检索（读取文件）
      │  └─ 读取：~/.cache/amd-ai-devtool/semantic-index/
      │     └─ bm25_index.pkl  ⭐ 只是读取 pickle 文件
      │
      ├─ 合并去重
      │
      └─ BGE 重排序
```

### 4.2 混合检索管道

`langchain_retrieval.py` 中定义的检索管道采用四阶段处理：

> [!tip]
> **混合检索四阶段**
> - **Stage 1**: Embedding 检索 → top-k 候选
> - **Stage 2**: BM25 检索 → top-k 候选
> - **Stage 3**: 合并去重
> - **Stage 4**: BGE Reranker 重排序 → 最终结果

这种设计结合了语义检索和关键词检索的优势，并通过重排序提升结果质量。

---

## 五、总结

通过以上分析，我们可以得出以下关键结论：

1. **无运行时依赖**：`mini-swe-agent` 与 `langchain/` 目录之间没有运行时依赖关系
2. **代码复制模式**：`HybridRetriever` 代码被复制到 `mini-swe-agent` 内部使用
3. **数据隔离**：所有组件通过共享的索引文件进行数据交换
4. **职责清晰**：`amd-ai-devtool` 提供数据，`langchain/` 构建索引，`mini-swe-agent` 执行检索

> [!question]
> **思考题**：为什么采用代码复制而非直接导入模块的方式？这种设计有什么优缺点？

---

## 相关链接

- [[混合检索系统设计]]
- [[FAISS 索引原理]]
- [[BM25 算法详解]]
- [[BGE Reranker 应用]]
