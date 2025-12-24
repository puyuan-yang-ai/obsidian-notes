# Agent加载与启动机制

## 一、前言

在了解了[[AIG-Eval框架核心概念与工作流程]]后，一个关键问题浮现：框架如何灵活地加载和启动不同的Agent？本文深入解析AIG-Eval的Agent加载与启动机制，揭示其"可插拔"设计的底层实现原理。

## 二、可插拔Agent的设计理念

> [!info]
> **可插拔（Pluggable）的含义**

"可插拔"意味着Agent与评测逻辑完全分离。框架设计成统一接口，你可以：
- 今天用[[Claude Code]]跑一遍任务
- 明天换成[[SWE-agent]]跑同样的任务
- 然后直接对比两者的评测结果

这种设计的核心价值在于**解耦**：框架不需要关心Agent的具体实现，只需要知道如何"启动"它。

## 三、Agent配置与切换

### 3.1 配置文件中的Agent定义

在主配置文件中，Agent通过`agent.template`字段指定：

```yaml
# config.yaml
agent:
  template: "SWE-agent"  # 可以切换为 "Claude Code" 等
```

### 3.2 Agent模板存放位置

每个Agent都有对应的配置目录，统一存放在`agents/`下：

```
AIG-Eval/
├── agents/
│   ├── SWE_agent/
│   │   ├── mini.yaml        # Agent专用配置
│   │   └── launch_agent.py  # 启动器脚本
│   ├── Claude_Code/
│   │   └── launch_agent.py
│   └── ...
```

## 四、Agent加载的完整流程

当框架需要启动一个Agent时，会经历以下流程：

```
第1步：读取config.yaml
       ↓
       agent名称："SWE-agent"
       ↓
第2步：module_registration.py
       ↓
       字符串 → AgentType.SWE_AGENT 枚举
       ↓
第3步：module_registration.py
       ↓
       根据枚举类型，import对应模块
       导入启动器
       ↓
第4步：launch_agent.py
       ↓
       启动器被调用
       本质执行：subprocess.Popen(cmd, ...)
       ↓
       调用系统命令"mini"
       ↓
第5步：系统PATH
       ↓
       找到"mini"命令并执行
```

> [!tip]
> **关键设计点**
> 框架通过**字符串配置** → **枚举映射** → **模块导入** → **进程启动**的链式机制，实现了Agent的动态加载。

## 五、launch_agent.py的实现模式

### 5.1 标准实现模式

大多数Agent的启动器遵循`AGENT + OPTIONS`模式：

```python
# agents/SWE_agent/launch_agent.py 示例

AGENT = "mini"  # CLI命令
OPTIONS = [
    "-c", "/path/to/mini.yaml",  # 配置文件
    "-t", "prompt",               # 提示词
    "--mcp"                       # MCP模式开关
]

def launch_agent(workspace, prompt):
    cmd = [AGENT] + OPTIONS + ["-t", prompt]
    return subprocess.Popen(cmd, ...)
```

### 5.2 实现方式的灵活性

> [!note]
> **launch_agent函数必须使用AGENT + OPTIONS模式吗？**

**答案是：不一定**。内部实现完全自由：

```python
# 方式1：调用CLI命令（subprocess）
subprocess.Popen(["mini", "-c", config])

# 方式2：调用Python模块
import minisweagent
minisweagent.run(config)

# 方式3：调用HTTP API
requests.post("http://agent-api/execute", json=config)

# 方式4：随便怎么实现都行
```

只要能启动Agent并返回结果，框架不关心具体实现方式。

## 六、CLI命令调用机制深度解析

### 6.1 源码目录 vs 已安装程序

> [!warning]
> **常见误区**

运行AIG-Eval选择mini-swe-agent时，框架**不会**读取本地源码目录，而是调用系统中已安装的程序！

```bash
# 通过pip安装后，mini命令被安装到系统PATH
pip install -e .
# 或
pip install minisweagent
```

### 6.2 验证实际调用路径

要确认CLI命令实际调用的程序位置，使用：

```bash
which mini
# 输出：/opt/conda/envs/xxx/bin/mini
```

这个路径才是AIG-Eval实际调用的程序！

### 6.3 完整执行链路

当你在终端输入`mini --mcp -c xxx.yaml -t "prompt"`时：

```
用户输入：mini --mcp -c xxx.yaml -t "prompt"
    ↓
系统查找PATH环境变量
    ↓
依次在以下目录查找名为"mini"的可执行文件：
    - /usr/local/bin/mini  ❌ 不存在
    - /usr/bin/mini        ❌ 不存在
    - ~/.local/bin/mini    ✅ 找到了！
    ↓
执行找到的mini脚本
    ↓
mini是一个入口脚本，指向安装后的Python包
    ↓
实际执行：python -m minisweagent.run.mini ...
    ↓
运行的是安装后的代码
```

## 七、配置文件的优先级问题

### 7.1 mini.yaml的双重身份

在mini-swe-agent的生态中，存在两个`mini.yaml`文件：

| 位置 | 类型 | 作用 |
|------|------|------|
| `mini-swe-agent/mini.yaml` | Agent自带配置 | Agent默认行为 |
| `AIG-Eval/agents/SWE_agent/mini.yaml` | AIG-Eval专用配置 | 强制控制Agent行为 |

当AIG-Eval通过`-c`参数指定配置文件时，使用的是**AIG-Eval内部的mini.yaml**，而不是Agent源码目录下的配置文件。

### 7.2 为什么AIG-Eval要强制指定配置？

> [!question]
> **为什么不使用Agent的默认配置？**

因为AIG-Eval需要**从评估框架的角度控制Agent行为**。评估框架有特定的需求：
- 统一的超时设置
- 特定的输出格式
- 标准化的交互协议
- 评测相关的特殊配置

### 7.3 MCP模式的提示词覆盖机制

当使用`--mcp`模式时，会出现一个有趣的现象：提示词的优先级问题。

在mini-swe-agent中，有两个地方定义了提示词模板：
- `mini.yaml`中的`system_template`和`instance_template`
- `prompts.py`中的`SYSTEM_TEMPLATE`和`INSTANCE_TEMPLATE`

> [!info]
> **提示词优先级规则**

当同时使用`-c mini.yaml`和`--mcp`时：

```python
# mini-swe-agent内部逻辑（伪代码）
if "--mcp" in args:
    # 主动删除yaml中的prompt模板
    del yaml_config["system_template"]
    del yaml_config["instance_template"]
    # 让prompts.py的模板生效
    use_prompts_from_file()
```

**结论**：`-c`指定的yaml中的提示词模板**不会生效**，`prompts.py`中的模板会**覆盖**yaml中的定义。

### 7.4 MCP模式下yaml文件还有效吗？

> [!tip]
> **其他配置仍然生效**

虽然提示词模板被覆盖，但`mini.yaml`中的其他配置依然有效：
- 模型选择（`model`）
- 超时设置（`timeout`）
- 其他Agent行为参数

只有`system_template`和`instance_template`被`prompts.py`替换。

## 八、接入自定义Agent

### 8.1 必需文件

每个Agent目录必须包含`launch_agent.py`作为启动器。

### 8.2 最简实现模板

```python
# agents/MyAgent/launch_agent.py

import subprocess

def launch_agent(workspace, prompt, config_path):
    """
    启动自定义Agent

    Args:
        workspace: 工作目录路径
        prompt: 用户提示词
        config_path: Agent配置文件路径

    Returns:
        subprocess.Popen对象
    """

    # 方式1：CLI命令
    cmd = ["my-agent", "-c", config_path, "-p", prompt]
    return subprocess.Popen(cmd, cwd=workspace)

    # 方式2：Python模块
    # import myagent
    # return myagent.run(config_path, prompt, workspace)

    # 方式3：HTTP API
    # import requests
    # return requests.post("http://localhost:8000/run", json={...})
```

### 8.3 在框架中注册

在`module_registration.py`中添加新Agent的枚举映射：

```python
from enum import Enum

class AgentType(Enum):
    SWE_AGENT = "SWE-agent"
    CLAUDE_CODE = "Claude Code"
    MY_AGENT = "MyAgent"  # 新增
```

## 九、总结

> [!success]
> **核心要点回顾**

1. **可插拔设计**：Agent与框架解耦，通过配置文件灵活切换
2. **加载流程**：配置字符串 → 枚举 → 模块导入 → 进程启动
3. **CLI调用**：框架调用系统PATH中的已安装程序，而非源码目录
4. **配置优先级**：AIG-Eval的配置覆盖Agent默认配置；MCP模式下`prompts.py`覆盖yaml的提示词
5. **接入自由度**：`launch_agent.py`的实现方式完全灵活

理解了Agent的加载与启动机制后，接下来可以深入了解底层的[[系统配置与Python环境]]，这将帮助你更好地理解CLI命令和Python模块的查找与执行原理。
