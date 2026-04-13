# SFT 数据处理与训练机制

理解 SFT（监督微调）的核心，需要掌握四个关键组件：**dataset（数据集）、tokenizer（分词器）、model（模型）、trainer（训练器）**。这四个组件构成了完整的 SFT pipeline，从原始数据到最终的微调模型，每个环节都不可或缺。

一个典型的 SFT pipeline 流程如下：

```
dataset → tokenizer → model → trainer → SFT model
```

对于初学者，推荐从最小 demo 入手：使用 TinyLlama 或 Qwen small 模型，搭配 Alpaca 数据集，并通过 [[LoRA]] 技术进行高效微调。

## 数据格式：从 Instruction 到 Prompt

几乎所有 SFT demo 都遵循相同的四步骤：1) 准备数据（dataset）；2) tokenizer 处理；3) 加载模型（model）；4) 训练（trainer）。

SFT 数据通常以 **instruction 格式**存储，这是一种 JSON 格式，包含三个字段：`instruction`、`input`、`output`。以经典的 Alpaca 数据为例：

```json
{
  "instruction": "Translate English to Chinese",
  "input": "Hello",
  "output": "你好"
}
```

但在训练前，这些数据需要转换为 **prompt 格式**（Markdown 格式）。上述数据会被拼接成：

```markdown
### Instruction: Translate English to Chinese
### Input: Hello
### Response: 你好
```

模型学习生成的是 "你好" 这部分。这种格式转换看似简单，但背后蕴含着深刻的设计理念。

## 为什么不直接训练 "Hello → 你好"？

这是一个值得深思的问题。如果目标是让模型学会翻译，为什么不直接用 `Hello → 你好` 这样的简单映射，而要构造复杂的 Instruction、Input、Response 结构？

答案在于：**SFT 的目标不是让模型学会一个特定的映射，而是让模型学会指令结构**。这正是 **Instruction Tuning（指令微调）** 的核心思想。

通过这种结构化的训练，模型学到的是"看到指令 → 完成任务"的通用能力，而不是只记住一个孤立的 "Hello → 你好" 的 mapping。这种能力被称为 **task following ability（指令遵循能力）**，是现代大模型能够理解和执行各种复杂任务的基础。

> [!tip]
> **Instruction 格式的价值**：
> 通过明确的指令结构，模型不仅学会了如何翻译，还学会了如何理解指令、如何根据输入生成输出。这种结构化的学习方式，使得模型能够泛化到训练时从未见过的新任务上。

## 从数据到模型：完整的训练流程

假设我们有一条训练数据：

```
User: 写一句关于AI的句子
Assistant: AI正在改变世界
```

在训练时，这条数据会被拆分为 **prompt** 和 **label** 两个部分：
- **Prompt**：`"User: 写一句关于AI的句子\nAssistant:"`
- **Label**：`"AI正在改变世界"`

模型的输入是 prompt 部分，它需要预测 label 部分。具体来说，模型会逐个 token 预测："AI 正在 改变 世界"。

例如，label "北京是中国的首都" 可能被分词器切分为：`北京 | 是 | 中国 | 的 | 首都`，共 5 个 token，因此模型需要预测 5 次。

**Loss 计算只针对 label 部分**。Prompt 的作用是提供上下文，让模型理解当前的任务和场景；而 label 则是模型需要学习生成的目标内容。

## Tokenizer：从文字到数字

在正式训练之前，所有文本都需要通过 **Tokenizer** 转换为 token id。模型只认识数字，不认识文字。

例如，"北京是中国的首都" 可能被转换为：

```
[1453, 2331, 991, 5021, 18]
```

每个数字对应词汇表中的一个 token。这个转换过程看似简单,但它是连接人类语言和模型计算的关键桥梁。

## Teacher Forcing：训练的核心机制

SFT 的训练方式采用 **Teacher Forcing**，这是一种强制使用真实答案（ground truth）作为输入的训练策略。

**Teacher Forcing** 这个术语的含义是：
- **Teacher（教师）**：指使用真实答案（ground truth）
- **Forcing（强制）**：强行把正确 token 喂给模型

训练流程如下：真实 token → 输入模型 → 预测下一个 token。而不是：模型预测 token → 再喂给模型 → 再预测。

换句话说，**训练输入始终是正确序列**。

### 为什么需要 Teacher Forcing？

如果不使用 Teacher Forcing，模型在中间某个位置预测错一个 token，误差就会一路传播，后面的预测全部错位，导致训练极其困难。Teacher Forcing 确保输入永远是真实 token，从而保证训练的稳定性。

在 LLM 训练中，Teacher Forcing 的表现形式是 **shift token**。例如：
- **输入**：`<BOS> I love machine`
- **标签**：`I love machine learning`

也就是说，输入永远是真实 token，模型学习的是"下一个 token 应该是什么"。

### Teacher Forcing vs 推理

Teacher Forcing 和推理（Inference）的区别非常明显：
- **训练期间（Teacher Forcing）**：输入的都是真实 token
- **推理阶段（Inference）**：输入的是模型自己预测的 token，即每一步输入都是上一步模型的预测

这种训练和推理之间的 gap，引出了一个经典问题。

## Exposure Bias：Teacher Forcing 的代价

Teacher Forcing 虽然让训练变得稳定，但也带来了一个经典问题：**Exposure Bias（曝光偏差）**。

问题的根源在于：
- **训练时**：模型只见过正确历史
- **推理时**：模型看到的是自己的预测

一旦模型在推理时预测错误，错误会不断累积，导致生成质量迅速下降。

为了缓解这个问题，后来提出了许多改进方法：
- **Scheduled Sampling**：训练时逐步混入模型自己的预测
- **RLHF**：通过人类反馈强化学习优化生成质量
- **DPO**：直接偏好优化
- **Sequence-level training**：在序列级别进行优化

> [!warning]
> **Exposure Bias 的影响**：
> 这是 SFT 训练中一个无法完全避免的问题。理解这个问题，有助于我们在实践中做出更好的训练策略选择，比如引入 [[RLHF]] 或 [[DPO]] 来进一步优化模型的生成质量。

## SFT 核心原理总结

综合来看，SFT 的核心原理可以归纳为以下几点：

1. **训练目标**：next-token prediction
2. **训练数据**：instruction → response
3. **loss 计算**：只计算 assistant tokens
4. **训练方式**：teacher forcing

SFT 的核心是 **Teacher Forcing + Next Token Prediction**。相比 Pretraining 对所有 token 计算 loss，SFT 只计算 assistant token 的 loss，从而让模型专注学习如何生成高质量的回答。

这种设计让模型能够快速适应特定的任务格式，同时保持预训练阶段学到的丰富知识。
