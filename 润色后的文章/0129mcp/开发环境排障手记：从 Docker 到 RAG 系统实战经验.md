# 开发环境排障手记：从 Docker 到 RAG 系统实战经验

## Docker 容器与终端会话管理

### 容器重启后终端会自动断开

> [!info]
> **容器重启 = stop + start**
>
> - `stop` 会向容器内所有进程发送 SIGTERM/SIGKILL
> - 之前的 `docker exec` 会话会断开
> - 需要重新进入容器

这是一个容易混淆的点：当你在容器内的终端，然后重启了容器，这个终端会自动从容器里退出来。原因是容器重启后，之前的会话连接失效了。

### tmux 会话无法持久化

> [!warning]
> **容器重启后 tmux 会话会消失**
>
> - `docker restart` = stop + start
> - stop 会杀死容器内所有进程，包括 tmux server
> - tmux 是进程，不是文件
> - 容器重启后文件系统保留（如果有 volume）
> - 但运行中的进程状态无法保留

如果你想在容器内使用 tmux 保持会话，需要记住：容器重启后所有会话都会丢失，这是容器的特性，不是 bug。

## ROCm GPU 设备故障排查

### HIP 错误：no ROCm-capable device is detected

当在 ROCm 容器里运行脚本后，如果看到类似错误：

```
HIP error silu.hip:148: no ROCm-capable device is detected
```

> [!danger]
> **原因：/dev/kfd 设备号不匹配**
>
> - `/dev/kfd` 是 ROCm 的内核驱动接口
> - HIP 程序必须通过它访问 GPU
> - 宿主机的 `/dev/kfd` 被重新创建（系统重启或驱动重新加载）
> - 容器是 5 天前启动的，当时挂载的是旧设备号 239
> - 设备号不匹配，导致 GPU 不可用

**解决方案很简单：**

```bash
docker restart <container-name>
```

重启 Docker 容器，让它重新挂载正确的设备即可。

## Git Commit Amend 操作指南

### 合并修改到上次提交

当你发现上次提交漏了点东西，或者刚提交就发现有个小 bug 需要修复，可以使用 `git commit --amend`。

> [!tip]
> **完整流程**
> ```bash
> # 1. 暂存修改（如果还没暂存）
> git add <file>
>
> # 2. 合并到上次提交，保留原 commit message
> git commit --amend --no-edit
>
> # 如果想同时修改 commit message：
> git commit --amend -m "new message"
> ```

### 理解 amend 的参数

`--amend --no-edit` 分别是什么意思？

- `--amend`：修改最近一次提交（可以改 message、追加文件）
- `--no-edit`：保留原 commit message 不变

> [!question]
> **常见误解**
>
> ❌ 错误理解：`-m` 是"是否修改 message"的开关
>
> ✅ 正确理解：
> - 不加 `-m` 也能改 message（通过编辑器）
> - 加 `-m` 也会合并暂存内容
> - 真正控制"是否保留原 message"的是 `--no-edit`

### 两种用法的区别

| 命令 | 效果 |
|------|------|
| `--amend --no-edit` | 仅合并 stage，不改 msg |
| `--amend -m "xxx"` | 合并 stage + 改 msg |

`git commit --amend -m "xxx"` 会同时做两件事：把当前 stage 里的内容合并到最近一次 commit，并把 commit message 改成 "xxx"。

## RAG 系统评估与故障排查

### RAG 系统评估维度

评估一个 RAG 系统的质量，需要从多个维度考虑：

> [!info]
> **四大评估维度**
>
> - **Query 质量**：用户查询是否清晰、具体
> - **检索质量**：召回率和准确率的平衡
> - **Reranker 效果**：精排是否真正提升了相关性
> - **对最后结果的价值**：是否真正改善了用户体验

### 混合检索去重机制

> [!success]
> **混合检索层的去重保证**
>
> - 两路召回结果会有重复的 chunk 吗？**不存在**
> - 因为两路的结果会合并，进行融合
> - 去重是基于对象 ID
> - 每个 chunk 对象最多出现 1 次

这意味着你可以放心地将混合检索的全部结果送入 Reranker，不需要额外去重。

### 粗排截断的问题

在 [[RAG 架构实战：混合检索与模型选型完全指南]] 中详细讨论了这个问题：**粗排不要截断太狠**，因为精排才是最终决策者，应该有足够的选择空间。

RRF (粗排) 在做精排决策，会浪费 Reranker 的能力，不符合主流"粗排 → 精排"范式。

## 相关文章

- [[Git 进阶：从 checkout 拆分到分支管理最佳实践]] — Git 基础命令详解
- [[RAG 架构实战：混合检索与模型选型完全指南]] — RAG 系统架构与评估
