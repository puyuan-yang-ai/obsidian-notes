# SFT 与预训练的本质区别

大模型的训练是一个逐步演进的过程，通常经历三个关键阶段。首先是 **Pretraining（预训练）**，这一阶段的核心任务是让模型学习语言规律和世界知识。模型在海量的新闻文章、小说、Wikipedia、论坛帖子和代码等连续文本上进行训练，学习的目标是"给定前面的文本预测下一个 token"。例如，当模型看到 "The capital of China is" 时，它会预测 "Beijing"，但这仅仅是预测延续，而不是在真正意义上回答问题。

第二个阶段是 **SFT（监督微调，Supervised Fine-Tuning）**，专注于教会模型如何回答问题。预训练模型虽然掌握了丰富的知识，但并不会天然地以问答形式输出内容，因为它从未在训练中见过明确的问答结构。SFT 通过引入结构化的问答数据（如 `{User: 中国的首都是哪里？ Assistant: 北京}`），让模型学习"User 提问 → Assistant 回答"的对话模式。

第三个阶段是 **RLHF/DPO（人类反馈强化学习）**，目的是让模型的回答更符合人类偏好。简单理解：预训练是学知识，SFT 是学怎么回答，RLHF 是学怎么回答得更好。

## 训练目标：本质相同，但数据分布完全不同

很多人初次接触 SFT 时会有一个疑惑：SFT 和预训练的训练目标难道不一样吗？实际上，**两者的训练目标在技术层面完全一致，都是 next-token prediction**，即"给定前面的 token 预测下一个 token"。

那么它们的区别在哪里？**关键在于数据分布的改变**。

预训练数据是 plain text（连续文本），没有角色划分，也没有问答结构，模型学习的是语言统计规律。而 SFT 数据包含明确的 User 和 Assistant 两个角色，呈现为 `instruction → response` 的格式。模型在 SFT 阶段学习的不是知识本身，而是一种**行为模式**：看到 User 提问，就应该生成 Assistant 的回答。

> [!info]
> **SFT 的本质（三句话总结）**：
> - 训练目标没有变：SFT 仍然是 next-token prediction
> - 改变的是数据分布：预训练是 plain text，SFT 是 instruction → response
> - 模型学习的是行为模式：模型学到的是"User 提问 → Assistant 回答"，而不是单纯学习知识

## Loss 计算：SFT 的核心技巧

虽然训练目标相同，但 SFT 和预训练在 **loss 计算**上有显著差异。预训练时，模型会对所有 token 计算 loss，而 SFT 通常只计算 Assistant 部分的 loss，User 部分会被 mask 掉。

这是为什么？因为我们希望模型专注学习如何生成回答，而不是学习如何提问。底层原理是：训练时会 mask 掉 User 部分的 loss，只计算 Assistant answer 的 loss，让模型将参数更新集中在生成高质量回答上。

具体区别如下：

| 维度 | 预训练 | SFT |
|------|--------|-----|
| **数据** | 海量文本（连续文本） | 指令问答（结构化问答对） |
| **目标** | next token prediction | next token prediction |
| **loss 计算** | 所有 token | 通常只算 assistant 部分 |
| **目的** | 学语言和知识 | 学如何回答 |

## 数据结构的力量

有一个有趣的问题：如果 SFT 不 mask user 部分，是否就和预训练一样了？

答案是：**不一样**。即使不 mask，SFT 数据仍然在教模型一种"对话结构"。

预训练数据是连续文本，没有角色和问答结构，模型学的是语言统计规律。而 SFT 数据明确包含 User 和 Assistant 两个角色，模型在学习一种固定的模式："User 提问 → Assistant 回答"。这种结构本身就构成了一种强大的监督信号。

换句话说，**两者的 training objective 相同（next-token prediction），但 data distribution 完全不同**。数据结构的改变，足以让模型学会全新的行为模式。

> [!tip]
> **为什么数据结构如此重要？**
> SFT 的关键不在于"学习新知识"，而在于"学习新的输出格式"。通过引入明确的角色划分和问答结构，模型逐渐理解：当看到 User 的指令时，应该以 Assistant 的身份生成相应的回答。这种结构化的学习方式，正是 [[指令微调（Instruction Tuning）]] 的核心思想。
