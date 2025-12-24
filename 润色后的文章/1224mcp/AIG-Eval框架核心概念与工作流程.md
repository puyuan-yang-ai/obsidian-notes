# AIG-Eval框架核心概念与工作流程

## 一、框架概述

[[AIG-Eval]]是一个用于评测不同AI Agent在代码优化任务上表现的评估框架。它的核心设计理念是将**评测框架**与**被测Agent**完全解耦，实现了Agent的可插拔（Pluggable）架构。这意味着你可以今天用[[Claude Code]]跑一遍任务，明天换成[[SWE-agent]]跑同样的任务，然后直接对比两者的表现。

> [!info]
> **框架核心价值**
> - 统一的评测标准：所有Agent在相同任务下公平竞争
> - 自动化流程：一次配置，自动完成编译、验证、测速、评分全流程
> - 结果可复现：每次运行都在独立隔离环境中进行

## 二、快速上手

### 2.1 基本配置

使用AIG-Eval进行评测非常简单，只需要在配置文件中指定两个核心要素：

```yaml
# config.yaml
agent:
  template: "SWE-agent"  # 或 "Claude Code" 等

tasks:
  - customer_hip/silu    # 指向 tasks/customer_hip/silu/config.yaml
```

配置完成后，执行以下命令即可启动评估：

```bash
cd /home/puyuyang/projects/work/code_backup/AIG-Eval
python main.py
```

> [!tip]
> **入口文件说明**
> - `main.py`是框架入口文件
> - 执行`python main.py`会触发完整的评测流程
> - 框架会自动读取config.yaml并完成所有后续工作

### 2.2 任务配置深度解析

每个具体任务都有独立的config.yaml，这个文件存放的是什么信息呢？答案是：**各种执行指令**。

这些指令不会直接被框架执行，而是作为[[Prompt]]的一部分送给Agent，让Agent自己决定如何执行。这正是评测Agent智能程度的核心意义——框架只告诉Agent"你要做什么"，不告诉它"你要怎么做"。

## 三、工作流程全景

AIG-Eval的完整工作流程包含九个阶段，每个阶段都有明确的职责分工：

### 3.1 阶段划分与职责

| 阶段 | 负责方 | 作用说明 |
|------|--------|----------|
| 工作空间准备 | 框架 | 复制任务目录到隔离workspace |
| Prompt构建 | 框架 | 自动读取任务配置，生成包含指令的prompt |
| 代码优化 | Agent | 修改源代码，实施优化策略 |
| 编译 | Agent | 按prompt指令运行make等编译命令 |
| 正确性验证 | Agent | 按prompt指令运行验证程序 |
| 性能测试 | Agent | 按prompt指令运行性能分析命令 |
| 结果记录 | Agent | 填写task_result.yaml |
| 评分 | 框架 | 读取task_result.yaml计算分数 |
| 报告生成 | 框架 | 汇总所有任务结果，输出评分报告 |

> [!note]
> **关键设计思想**
> 框架负责"管理"和"评判"，Agent负责"执行"和"决策"。这种清晰的职责分离使得框架可以无缝切换不同的Agent。

### 3.2 Agent的执行流程

当Agent接管优化任务后，它会自主执行以下流程：

```
a) 阅读源代码，分析优化点
   ↓
b) 修改kernel代码（实施优化策略）
   ↓
c) 运行编译命令（如make）
   ├─ 如果失败 → 分析错误信息 → 修复代码 → 重新编译
   └─ 如果成功 → 继续
   ↓
d) 运行正确性验证命令（如./applications_silu）
   ├─ 检查输出是否包含"Validation passed"
   ├─ 如果失败 → 修复bug → 重新测试
   └─ 如果成功 → 继续
   ↓
e) 运行性能测试命令（如rocprof-compute）
   ├─ 测量优化前后的执行时间
   └─ 计算加速比
   ↓
f) 填写task_result.yaml
```

## 四、Workspace隔离机制

### 4.1 为什么需要Workspace？

> [!warning]
> **Workspace = 独立的文件夹副本**

Workspace是一个简单的文件夹副本，**不是**Python虚拟环境！创建副本的目的是：

- **保护原始文件**：Agent可能会乱改代码，但不会影响原始任务目录
- **可重复实验**：每次运行都是全新的副本，互不干扰

### 4.2 Workspace创建流程

`setup_workspace`函数的执行流程如下：

```python
# 1. 获取任务文件夹名
task_name = "customer_hip/silu"

# 2. 创建带时间戳的新目录
timestamp = datetime.now().strftime("%Y%m%d_%H%M%S")
workspace_path = f"workspace_{timestamp}_{task_name}"

# 3. 复制任务文件夹的所有内容到新目录
shutil.copytree(f"tasks/{task_name}", workspace_path)
```

每次运行都会生成类似`workspace_20241224_143022_customer_hip_silu`的独立目录。

## 五、任务文件结构解析

以`assign_score_withk`任务为例，一个典型的评测任务包含以下文件：

| 文件 | 作用 |
|------|------|
| `src/*.hip` | 待优化的HIP kernel（AMD版本，评测目标） |
| `test_assign_score_withk.py` | 主测试脚本：编译kernel → 加载 → 验证 → 测性能 |
| `*_wrapper.py` | Python层封装，方便调用kernel |
| `kernel_loader.py` | 动态加载编译好的kernel |
| `*.pt` | 输入测试数据 |
| `expected_*.pt` | 正确性验证的参考答案 |
| `assign_score_withk.cpp` | 胶水代码，让Python能调用HIP kernel |
| `assign_score_withk_cuda.cu` | 原始CUDA代码（NVIDIA版本，仅供参考） |

> [!info]
> **CUDA → HIP移植场景**
> 文件结构清晰展示了一个典型的代码移植场景：
> - `.cu`文件：原始NVIDIA版本（保留参考）
> - `.hip`文件：移植后的AMD版本（待优化目标）
> - 最终目标：Agent产出的优化后`.hip`文件

Agent**不需要**修改胶水代码（`.cpp`文件），只需专注于优化`.hip` kernel即可。

## 六、框架的边界

> [!question]
> **AIG-Eval框架是否直接处理待评估的hip文件？**

**答案是：不会**。

框架本身不直接处理任何源代码文件，它只读取config.yaml中的命令，然后交给Agent执行。这种设计保证了框架的通用性和灵活性——无论评测什么类型的代码优化任务，框架本身都不需要修改。

## 七、随机性与评测可靠性

由于Agent本身存在随机性（如大模型输出的不确定性），单次运行的结果可能波动。AIG-Eval通过以下方式保证评测的可靠性：

- 评分维度固定：每次评测使用相同的评分标准
- 多次运行统计：实际评测会多次运行取平均值
- 独立workspace：每次运行互不干扰

## 八、Prompt构建机制

框架通过`prompt_builder.py`构建最终的Agent提示词，包含以下要素：

- **Task Type**：任务类型说明
- **Source Code**：源代码内容
- **Instructions**：编译、验证、测速等指令
- **Output Format**：输出格式要求
- **Cheatsheet**：固定的优化策略（可选）
- **Workspace Info**：工作目录信息

> [!tip]
> 如果需要移除Cheatsheet部分（让Agent完全自主决策），可以在`prompt_builder.py`中删除相关代码：
> ```python
> cheatsheet_prompt = ""
> prompt_sections.append(cheatsheet_prompt)
> ```

---

## 总结

[[AIG-Eval框架核心概念与工作流程]]是一个设计精良的评测框架，通过**可插拔Agent**、**独立Workspace**、**自动化流程**三大核心机制，实现了对不同AI Agent代码优化能力的客观评估。理解这些核心概念后，接下来可以深入了解[[Agent加载与启动机制]]。
