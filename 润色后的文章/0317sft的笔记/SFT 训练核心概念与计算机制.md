# SFT 训练核心概念与计算机制

在深度学习模型的监督微调（Supervised Fine-Tuning, SFT）过程中，理解训练的核心概念和计算机制是掌握模型优化的基础。本文将系统梳理 SFT 训练中的关键指标、文件格式、步数计算以及梯度累积等核心知识点。

## 训练指标与文件格式

训练日志中经常看到的 **it/s** 指标代表 **iterations per second**，即每秒完成的训练迭代数。一个 iteration 对应一个完整的训练步骤：前向传播、反向传播和参数更新。这个指标直接反映了训练速度，对于 Qwen-0.5B 这样的小模型配合 LoRA 微调，1.2 it/s 属于中等偏慢的速度，而在单卡 A100/A800 上通常可以达到 3-8 it/s。

在模型权重存储方面，**safetensors** 已经成为现代深度学习的主流格式。这个名称来源于 "safe + tensors"，由 Hugging Face 开发，核心设计目标是安全性。与传统的 .bin 文件（基于 Python pickle）不同，safetensors 采用纯数据格式（header + raw tensor data），加载时不执行任何代码，从根本上杜绝了反序列化攻击的风险。此外，safetensors 还具备以下优势：

- **加载速度更快**：支持内存映射（mmap），实现零拷贝加载
- **文件更小**：无冗余元数据
- **跨框架兼容**：PyTorch、TensorFlow、JAX 都可以使用

无论是完整模型权重还是 LoRA adapter 权重，都可以保存为 safetensors 格式，它本质上是一种存储容器，里面装的就是 tensor 数据。

> [!info]
> **训练输出目录的文件差异**：
> - `checkpoint-9762/` 包含完整的训练状态（adapter_model.safetensors、adapter_config.json、optimizer.pt、scheduler.pt、trainer_state.json、training_args.bin、rng_state.pth），用于断点续训
> - `final/` 只保留推理所需文件，不含优化器、调度器、随机数、训练状态等，专为推理部署设计

## 训练步数的自动计算机制

很多初学者会疑惑：训练总步数是人工指定还是自动计算的？答案是**自动计算**。框架会根据以下公式推导：

```
总步数 = ceil(样本总数 ÷ 等效 batch_size) × epoch 数
```

以一个具体场景为例：
- 样本数：52,049
- per_device_train_batch_size = 4
- gradient_accumulation_steps = 4
- GPU 数量 = 1
- 等效 batch_size = 4 × 4 × 1 = 16
- 每个 epoch 步数 = ceil(52049 ÷ 16) = 3254（向上取整）
- 总步数 = 3254 × 3 = 9762

你可以通过设置 `num_train_epochs=3` 让框架自动计算，也可以直接设置 `max_steps=N` 手动指定步数（此时 epoch 设置会被忽略）。

## 理解等效 batch_size 和梯度累积

**等效 batch_size** 是 SFT 训练中一个非常重要的概念，它的计算公式是：

```
等效 batch_size = per_device_train_batch_size × gradient_accumulation_steps × GPU数量
```

这里引入了 **gradient_accumulation_steps**（梯度累积）的概念。在正常情况下，每个 step 处理 batch_size 个样本后立即更新参数。而梯度累积的策略是：连续执行多次前向+反向传播，把梯度累加起来，最后才做一次参数更新。

举个例子，`bs=4, accum=4` 意味着：

1. Step 内部连续处理 4 个 mini-batch（每个 bs=4）
2. 分别计算梯度并累加
3. 最后用累加后的梯度更新一次参数
4. 这一次参数更新中实际看到了 4×4=16 个样本，等效于 bs=16

那么 `bs=4, accum=4` 和 `bs=16` 在数学和性能上是否等价呢？

- **数学上**：近似等价但不完全相同。bs=16 一次性求 16 个样本的梯度平均；bs=4, accum=4 是分 4 次各算 4 个样本的梯度再平均。由于 BatchNorm 等操作依赖 batch 内统计量，可能有微小差异。但对于大语言模型（使用 LayerNorm），几乎完全等价。
- **性能上**：`bs=16, accum=1` 更快，因为 GPU 并行度高，硬件利用率好；`bs=4, accum=4` 更省显存，因为每次只需在显存中放 4 个样本的激活值，而不是 16 个。

> [!tip]
> **梯度累积的核心价值**：
> 在显存不够跑大 batch 时，用时间换空间来模拟大 batch 的效果。

## 训练步数与样本数量的关系

epoch 里面的**步数**表示什么？每一步代表一次参数更新，涉及前向传播、反向传播、梯度计算和参数更新。每一步对应的样本数量指的是**等效 batch size**（更新一次权重所需的样本数量），计算公式：

```
每步样本数 = per_device_train_batch_size × gradient_accumulation_steps × GPU 数量
```

这意味着，当你看到"训练了 3254 steps"时，实际上模型已经处理了 `3254 × 16 = 52,064` 个样本（假设等效 batch_size=16）。

## 训练日志与指标监控

在 SFT 训练过程中，像 loss、step 等指标会自动记录在 `output_dir` 下的 `trainer_state.json` 文件中，具体存储在 `log_history` 字段里。Hugging Face Trainer 已经帮我们完成了这件事，无需手动实现。

记录频率由 `logging_steps` 参数控制，通常设为 10-50 步（建议让总日志条目在 200-1000 条之间比较合适）。在训练监控中，我们需要重点关注以下指标：

- **Loss 曲线**（最重要）：判断模型是否在收敛、是否过拟合
- **学习率曲线**：确认 warmup 和 decay 是否按预期执行
- **Eval loss 曲线**：如果有验证集，对比 train loss 和 eval loss 来判断过拟合
- **Grad norm**：梯度范数，观察训练稳定性（突然飙升说明训练不稳定）
- **吞吐量（samples/s）**：监控训练效率

## 工具选择：LLaMA-Factory 的定位

LLaMA-Factory 是目前中文社区最流行的 SFT 工具之一，它的特点包括：

1. 零代码/低代码配置，YAML 一键启动
2. 支持几乎所有主流模型（Qwen、LLaMA、ChatGLM、Baichuan 等）
3. 集成 LoRA / QLoRA / Full / Freeze 等多种微调方式
4. 内置数据集管理、评估、推理、合并等全流程

对于学习和实践而言，建议采用以下路径：

- **学习阶段**：先用原生 Transformers + PEFT 手写训练脚本，搞清楚每个参数的含义
- **实践阶段**：之后再迁移到 LLaMA-Factory，此时你会清楚它帮你封装了什么，遇到问题也能快速定位

> [!note]
> **工具与原理的关系**：
> LLaMA-Factory 是加速器，理解原理才是核心。只有深入理解底层机制，才能在出现问题时快速诊断和解决。

---

通过理解这些核心概念和计算机制，你将能够更加从容地应对 SFT 训练中的各种问题，无论是调整超参数、优化训练效率还是诊断训练异常，都能做到心中有数。
