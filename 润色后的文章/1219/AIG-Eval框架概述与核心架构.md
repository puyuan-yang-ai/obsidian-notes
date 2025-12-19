# AIG-Eval框架概述与核心架构

AIG-Eval是一个专门为GPU Kernel优化评测设计的创新框架，其核心目标是自动化评测不同AI Agent（代码生成智能体）在GPU kernel优化任务上的表现能力。作为AI评测领域的重要工具，AIG-Eval通过标准化的评分机制和独特的解耦设计，为研究者提供了一个公平、可靠的评测平台。

## 核心评测维度

AIG-Eval建立了完整的三维评分体系，全面评估Agent的优化能力：

1. **编译成功度**（20分）- 评估Agent生成的代码是否能够成功编译通过
2. **正确性验证**（100分）- 确保优化后的代码在功能上与原始实现完全一致
3. **性能加速比**（×100）- 测量优化后代码相对于原始实现的性能提升倍数

最终得分计算公式为：编译成功（20分）+ 正确性（100分）+ 加速比×100。这种评分机制既保证了代码的基本可用性，又强调了性能优化的重要性，形成了科学合理的评价标准。

## 解耦设计理念

AIG-Eval最突出的设计特点是其高度的解耦架构，主要体现为三个独立可插拔的维度：

### 1. Agent的可插拔性
框架支持多种AI Agent的无缝切换，用户只需修改配置文件中的一行代码即可更换评测的Agent：

```yaml
# config.yaml
agent:
  template: SWE-agent    # ← 修改这里即可切换Agent
```

[!tip] 可插拔（Pluggable）意味着系统组件可以随时替换、切换，为用户提供灵活的配置选项。

### 2. Task的可插拔性
评测任务同样支持动态配置，用户可以轻松选择或切换不同的GPU kernel优化任务：

```yaml
# config.yaml 第26-27行
tasks:
  - customer_hip/point_to_voxel    # ← 修改这里选择任务
  - customer_hip/silu             # ← 可以配置多个任务
```

### 3. 独立的评测逻辑
评测逻辑与具体实现完全分离，确保评测过程的客观性和一致性。框架只负责调度Agent并计算最终评分，而不干预具体的优化策略。

## 配置管理架构

AIG-Eval的配置体系分为两个层次：

### 全局配置（config.yaml）
负责管理Agent选择、任务列表和目标GPU型号等全局设置：

```yaml
agent:
  template: SWE-agent     # 【Agent 在这里改】
tasks:                     # 【Task 选择在这里改】
  - customer_hip/silu
  - customer_hip/point_to_voxel
target_gpu_model: MI300   # 【目标 GPU 在这里改】
```

[!warning] 需要注意的是，AIG-Eval采用单Agent评测模式，一次只能评测一个Agent，不能同时评测多个Agent。

### 任务配置（tasks/xxx/config.yaml）
每个任务都有独立的配置文件，定义具体的执行参数：

- **source_file_path** - 指定需要优化的源代码文件
- **compile_command** - 定义编译命令
- **correctness_command** - 设置正确性验证指令
- **performance_command** - 配置性能测试命令
- **task_type** - 指定任务类型（如hip2hip）

## 框架设计哲学

AIG-Eval的设计遵循"告诉Agent做什么，而不是怎么做"的核心理念。框架通过[[任务配置与管理]]系统，将编译、测试等命令通过提示词喂给Agent，让Agent自主决定执行策略。这种设计的目的是评测Agent的自主编程能力，而不是简单地运行优化脚本。

当遇到编译失败等错误时，传统脚本会直接退出，而Agent可以分析错误信息、修复代码、重新编译，直到成功完成优化任务。这正是评测Agent智能程度的关键所在。

通过这种解耦架构和自主设计理念，AIG-Eval为GPU kernel优化领域提供了一个强大、灵活的评测工具，推动了AI代码生成技术的发展和评估标准的建立。