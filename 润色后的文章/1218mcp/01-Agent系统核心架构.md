# Agent系统核心架构：从循环控制到自主决策

Agent系统的核心架构看似简单，却蕴含着精妙的设计哲学。本质上，所有Agent都可以被理解为一个`while`循环加上一个异常控制机制。这种设计模式虽然在形式上统一，但不同Agent的核心差异主要体现在提示词中定义的workflow工作流程。

## 核心结构解析

Agent的基本运行机制遵循一个简单的模式：通过无限循环持续执行任务，直到特定条件满足时抛出异常来终止循环。这种设计体现了"控制反转"的思想——不是外部代码控制何时停止，而是Agent内部根据任务完成情况自主决定。

```python
# Agent核心结构伪代码
while True:
    observation = observe_environment()
    if is_task_complete(observation):
        raise Submitted("任务结果")
    action = decide_next_action(observation)
    execute(action)
```

在Agent的工作流中，observation（观察）环节承担着三个关键作用：判断任务是否完成、决定下一步行动、以及触发结束信号。这三个作用共同构成了Agent的感知-决策-执行闭环。

## Mini-SWE-Agent架构解析

[[Mini-SWE-Agent]]采用了精巧的三层解耦架构，展现了优秀的软件设计原则：

- **Protocol层**：定义标准接口，包括Model、Environment、Agent的抽象协议
- **实现层**：包含各种具体实现，如AnthropicModel、MCPEnabledEnvironment、DefaultAgent等
- **集成层**：将各组件组合成完整的Agent系统

> [!tip] 这种架构设计的核心优势在于各组件可以独立扩展和替换，无需修改其他部分。

### Agent的可替换性设计

任何实现了`run()`方法并拥有model、env、messages、config属性的类，都可以作为Agent使用。Mini-SWE-Agent中定义了一个清晰的继承链：DefaultAgent → InteractiveAgent → TextualAgent。系统默认使用InteractiveAgent，而通过`-v`参数可以切换到TextualAgent。

```bash
# 默认使用InteractiveAgent
mini-swe-agent

# 切换到TextualAgent
mini-swe-agent -v
```

## 工作流设计

Agent的Workflow通常包含带数字序号的明确步骤，这些步骤指导Agent如何执行任务。不同的workflow会导致执行同样任务时，有的Agent只需三步就能完成，有的则需要十五步。这种差异直接影响Agent的效率和可靠性。

> [!warning] Workflow的设计必须清晰明确，避免模糊的指令导致Agent行为不可预测。

## 自主决策机制

Agent的"自主决策"本质上是LLM根据提示词中的指导结合当前上下文，生成下一步的行动。这种决策过程不是随机的，而是基于规则和上下文的推理过程。LLM会根据：
- 提示词中的指导原则
- 当前任务进度
- 已获得的信息
- 下一步需要什么

来决定是否调用工具、执行什么操作。

## 任务完成机制

Mini-SWE-Agent采用了特定的标记机制来完成任务提交。这个机制有严格的约束：

1. 标记必须被````bash代码块包裹
2. 必须包含`COMPLETE_TASK_AND_SUBMIT_FINAL_OUTPUT`标记
3. 标记必须位于输出内容的第一行

```bash
echo "COMPLETE_TASK_AND_SUBMIT_FINAL_OUTPUT 最终答案是42"
```

这种设计确保了任务提交的一致性和可追踪性。

## 日志与调试

Agent生成的日志默认保存在`~/.config/mini-swe-agent/last_mini_run.traj.json`，这为调试和分析Agent行为提供了重要依据。通过分析这些日志，可以理解Agent的决策过程，发现潜在问题，并优化其性能。

Agent系统的架构设计体现了"简单中见复杂"的哲学。虽然基本结构简单，但通过workflow的差异、组件的灵活组合以及自主决策机制，能够实现复杂多样的智能行为。这种设计不仅保证了系统的可靠性，还为后续的扩展和优化留下了充足空间。