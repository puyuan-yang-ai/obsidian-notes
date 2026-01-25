# RAG 知识库：从 Frontmatter 到 FAISS 索引的完整链路

## Frontmatter：Markdown 的元数据心脏

> [!info]
> **Frontmatter 定义**
>
> Frontmatter 就是 Markdown 文件顶部用 `---` 分隔的元数据块，使用 YAML 格式编写。

```yaml
---
layer: "optimization"
category: "performance"
tags: ["bf16", "vectorization", "hip"]
rocm_version: "6.0"
vendor: "amd"
---
```

这五个字段（tags、layer、category、rocm_version、vendor）构成了 RAG 知识库 Metadata 的核心。

## Metadata 的三阶段生命周期

Metadata 在 RAG 系统中扮演着贯穿始终的角色，可以分为三个阶段：

### 阶段 1：存储阶段

```mermaid
YAML Frontmatter → chunker.py 解析 → chunk.metadata → vector_store.py 持久化
```

程序启动时，一次性加载两个核心文件：

| 文件 | 存储内容 | 作用 |
|------|----------|------|
| `faiss.index` | FAISS 索引（只存向量 embeddings） | 快速相似度搜索 |
| `chunks.pkl` | Pickle 文件（存 metadata + content） | 完整 chunk 信息 |

> [!tip]
> **分工明确的设计**
>
> - **FAISS 索引**：只存储向量，不存储文本或 metadata
> - **Chunks 文件**：存储每个 chunk 的完整信息（文本 + frontmatter 的 metadata）

### 阶段 2：检索阶段

检索流程是**先语义搜索，后 Metadata 过滤**：

```
用户查询 → FAISS 向量相似度搜索 → Metadata 后过滤 → 返回结果
```

每次查询的执行过程：
1. Query → embedding 向量
2. `self._index.search(embedding)` → 索引位置 [3, 17, 42]
3. `self.chunks[3]`, `self.chunks[17]` → chunk 对象
4. 用 `chunk.metadata` 进行后过滤
5. 返回最终结果

### 阶段 3：展示阶段

格式化输出时，chunk 的 section 标题、Layer、Category、Tags、ROCm Version 这四个参数会随检索结果一起返回。

## Tags 过滤的两种策略

> [!warning]
> **Tags 过滤的核心差异**
>
> - **@amd:query**：❌ 不使用 tags 过滤，纯语义搜索
> - **@amd:optimize**：✅ 使用 tags 过滤，tags=["optimization", "performance"]

Tags 过滤使用**或逻辑（OR）**，只要匹配过滤器中的任意一个 tag 就保留：

```python
# Chunk A: tags = ["optimization", "bf16", "hip"]
# 包含 "optimization" ✅ → 保留

# Chunk B: tags = ["performance", "memory"]
# 包含 "performance" ✅ → 保留

# Chunk C: tags = ["hip", "kernel", "bf16"]
# 不包含 "optimization" 也不包含 "performance" ❌ → 排除
```

主查询 `query()` 故意不用 tags 硬过滤，让语义相似度决定结果。而专用方法（如 optimization）则使用 tags 过滤来精确匹配。

## 当检索结果显示 unknown

如果检索到的 chunk 返回的 Layer 和 Category 显示 "unknown"，原因是 **md 文档里面的 frontmatter 缺少这两个字段**。

解决方案：
1. 在 frontmatter 中添加 `layer` 和 `category` 字段
2. 重新构建索引——删除缓存后重新加载，让新 metadata 生效

## Chunk 分割策略：字符数 vs Token 数

> [!question]
> **RecursiveCharacterTextSplitter 默认是按什么计算的？**
>
> 按字符数（characters）计算的，不是 token 数。

### 中英文差异

| 语言 | 字符数 vs Token 数 |
|------|-------------------|
| 英文 | 约 4 个字符 ≈ 1 个 token（平均） |
| 中文 | 约 1-2 个字符 ≈ 1 个 token |

### 为什么默认用字符数

默认的 `chunk_size=1000` 字符对于 RAG 场景通常是足够的：

1. **字符计数更快**：不需要调用 tokenizer
2. **精确度够用**：对于知识库检索，精确的 token 数量不是那么关键

## 索引重建流程

当批量修改 md 文件（如为 206 个文件添加 layer 和 category 字段）后，或者新增 BM25 索引时，需要重建索引。

> [!tip]
> **重建索引的正确方式**
>
> 不需要手动删除缓存文件！`--force` 参数就是让它强制重建并覆盖现有索引。

```bash
# 在容器内执行
pip install rank-bm25>=0.2.2
amd-ai-devtool maintenance build-index --force
```

`--force` 参数会自动：
- ✅ 覆盖 `~/.cache/amd-ai-devtool/semantic-index/` 下的所有文件
- ✅ 更新 FAISS 索引 (faiss.index)
- ✅ 更新 BM25 索引 (bm25.pkl)
- ✅ 更新 chunks 数据 (chunks.pkl)

索引文件默认存放在 `/root/.cache/amd-ai-devtool/semantic-index/` 目录下。

## 参数优先级的业界标准

> [!success]
> **参数配置的黄金法则**
>
> **显式参数 > 配置文件 > 代码默认值**

| 优先级 | 来源 | 说明 |
|--------|------|------|
| 最高 | 显式参数 | 用户主动传入的值（命令行参数或函数调用时传入） |
| 中等 | 配置文件 | chunking.yaml 中定义的值 |
| 最低 | 代码默认值 | 函数签名中的 `top_k=10` |

`@amd:query` 必须有 `topic` 参数（这是语义搜索的核心），其他参数可选——不传就不过滤，传了就会根据传入的参数进行 metadata 后过滤。

## 相关文章

- [[Git 分支管理与代码回滚完全指南]]
- [[RAG 混合检索：从 RRF 融合到 Rerank 精排序]]
- [[容器化 AI 开发：从环境搭建到 Agent 自动化工作流]]
