# Agent 系统设计：从 ReAct 模式到提示词工程

在现代 AI 系统的设计中，Agent 架构已经成为了实现复杂任务自动化的重要模式。理解其核心设计模式和工作机制，对于构建高效的 AI 应用至关重要。

## ReAct：Agent 设计的主流范式

ReAct (Reasoning + Acting) 模式已经成为目前最主流的 Agent 设计模式。这个模式的核心思想是让 AI 模型在执行任务时，像人类一样进行"思考-行动-观察"的循环过程。

### ReAct 循环的四个阶段

1. **LLM 思考与生成 Action**：模型分析当前情况，决定下一步行动
2. **执行 Action 获取 Observation**：系统执行模型选择的行动，获得结果
3. **Observation 作为反馈**：将观察结果作为 User message 反馈给 LLM
4. **继续思考循环**：LLM 基于新的观察结果，继续下一轮思考

这个循环过程体现了智能体与环境的持续交互，通过不断的反馈迭代来逼近目标。

> [!tip] 为什么 Observation 作为 User message？
> 这种设计符合 LLM 的训练模式。LLM 在训练过程中就学习了"assistant 回复 → user 反馈 → assistant 继续"的对话模式，这种自然的多轮推理机制让模型能够更好地理解和响应环境反馈。

## Agent 系统的核心组件

一个完整的 Agent 系统需要多个关键组件的协同工作。在 mini-swe-agent 中，这些组件通过精心设计的架构有机地结合在一起。

### 提示词模板系统

提示词模板在 Agent 系统中扮演着至关重要的角色，它们定义了模型的行为规范和工作流程。

**system_template** 负责定义 LLM 的基本角色和规则：
- 角色定义（如 "helpful assistant"）
- 响应格式要求
- 思考过程规范（如 THOUGHT 标签的使用）

**instance_template** 则专注于任务定义和工作流：
- 具体的任务描述
- 推荐的工作流程（分析→复现→修复→验证）
- 重要的规则约束
- 命令使用示例（创建文件、编辑操作等）

### 任务管理与控制机制

Agent 系统需要有效管理任务的执行过程。mini-swe-agent 通过一个巧妙的机制来实现任务的结束控制：

```python
# 伪代码示例
def has_finished(response):
    if "COMPLETE_TASK_AND_SUBMIT_FINAL_OUTPUT" in response:
        raise Submitted("任务完成")
```

当系统检测到特定的结束信号时，会抛出异常来跳出执行循环。这种通过异常控制流程的方式，虽然看起来有些"反直觉"，但在实际应用中非常有效。

## MCP 集成对 Agent 架构的影响

当我们将 [[深入理解 MCP：Model Context Protocol 在 Agent 系统中的集成与应用]] 集成到 Agent 系统中时，需要对传统的 ReAct 模式进行一些调整。

### 工作流程的改造

传统的 ReAct 循环中，模型可以自由选择是否使用工具。但在 MCP 集成的场景下，我们需要强制模型首先查询知识库：

```yaml
# 强制使用 MCP 的提示词
Your FIRST response MUST be an @amd:query command
DO NOT answer from your own knowledge
Always extract keywords from user tasks and query first
```

这种改造意味着：
- 模型不能直接回答问题，必须先查询知识库
- LLM 需要学会从用户任务中提取关键词
- 响应的生成必须基于查询到的专业知识

### 控制力的增强

通过 MCP 集成，我们获得了对 Agent 行为更强的控制力：

1. **调用时机控制**：可以强制要求在某些情况下必须调用工具
2. **输出格式规范**：可以要求特定的总结格式
3. **行为约束**：可以禁止模型使用某些行为（如不使用自身知识回答）

这种精细控制能力让 Agent 系统更加可靠和可预测。

## 提示词工程的实践

提示词工程是 Agent 系统设计中的核心技能。好的提示词不仅能够引导模型产生期望的行为，还能提高系统的整体性能。

### 分层设计原则

在 mini-swe-agent 中，提示词采用了分层设计：
- **角色层**：定义模型的基本身份和行为准则
- **任务层**：描述具体的工作流程和任务要求
- **工具层**：说明可用工具的使用方法

这种分层让提示词更加清晰和易于维护。

### 动态提示词生成

当集成 MCP 时，系统需要动态生成包含工具列表的提示词。这种动态生成机制让 Agent 能够自适应不同的工具环境：

```python
# 动态生成系统提示词的示例
def generate_system_prompt(tools):
    base_prompt = "You are a helpful assistant with access to tools:\n"
    for tool in tools:
        base_prompt += f"- {tool.name}: {tool.description}\n"
    return base_prompt
```

## Agent 系统的设计哲学

通过分析 mini-swe-agent 的架构，我们可以总结出一些重要的设计原则：

1. **模块化设计**：各个组件职责明确，易于维护和扩展
2. **可配置性**：通过配置文件（如 mini.yaml）灵活调整系统行为
3. **最小侵入性**：新功能的集成尽量不破坏原有架构
4. **可控性优先**：宁可牺牲一些灵活性，也要保证系统的可控性

这些原则对于构建复杂的 AI 系统具有重要的指导意义。

## 未来发展方向

Agent 系统的设计仍在不断演进中。随着大模型能力的提升和工具生态的丰富，我们可以预见：

1. **更强的推理能力**：模型将能够进行更复杂的多步推理
2. **更丰富的工具生态**：标准化协议将促进工具的互操作性
3. **更自然的交互方式**：Agent 将更好地理解人类的意图和偏好

对于开发者而言，掌握 Agent 系统的设计原理和实践经验，将成为构建下一代 AI 应用的关键技能。

---

**相关链接**：[[深入理解 MCP：Model Context Protocol 在 Agent 系统中的集成与应用]] | [[Python 开发实践：包管理与面向对象编程技巧]]