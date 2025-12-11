# MCP基础概念与架构

## MCP协议概述

MCP（Model Context Protocol）是一种通信协议，专为AI模型与外部工具交互而设计。它采用经典的客户端/服务端架构（C/S架构），实现了AI模型与外部功能的解耦，让AI模型可以专注于决策，而将具体执行交给专门的服务处理。

## 核心组件解析

### MCP Server的本质

MCP Server的本质是一个后台应用程序，它作为工具的提供者，暴露多个功能接口供外部调用。Server需要独立运行，是一个独立的进程，它不负责决策何时运行工具，只是单纯地提供工具功能。

> [!tip] MCP Server的独立性
> 虽然MCP Server遵循MCP协议，但它不一定必须通过MCP协议调用。由于底层通常是Python代码实现，因此可以直接作为Python库import使用。

### MCP Tool的本质

MCP Tool是MCP Server对外暴露的功能接口，类似于函数。每个Tool都包含完整的信息：
- 名称和说明
- 参数schema定义
- 返回值schema
- 权限要求

这些元数据让Client能够了解如何正确调用该工具。

### MCP Client的定位

在MCP架构中，Client位于AI模型端，负责发起调用请求。它起到了桥梁作用：
- 接收LLM的决策指令
- 解析并转发调用请求到对应的Server
- 将执行结果返回给LLM

## 架构设计理念

### C/S架构的优势

MCP采用客户端/服务端架构的设计理念是：
- **客户端（AI端）**：负责发起调用请求，做决策
- **服务端（工具端）**：暴露多个工具，执行具体功能

这种设计的好处是AI只需知道使用哪个工具，无需了解工具的实现细节，实现了关注点分离。

### 组件协作关系

在LLM领域中，"工具"指的是外部功能。但值得注意的是，MCP中并非由LLM直接触发工具调用，而是由MCP Client来触发MCP Server上暴露的工具。

LLM的作用是决定并输出调用意图（选择哪个工具及参数），但不执行代码或直接访问外部API。这是因为LLM的本质是生成文本，无法直接执行外部功能。

## 与传统方案的区别

### LangChain Tool vs MCP Tool

- **LangChain的@tool**：面向LangChain Agent的装饰器，自动生成工具schema
- **MCP Server Tool**：面向MCP协议的Client/Server，通过协议通信

两者面向的对象不同，不能混用。不能用LangChain的@tool声明MCP工具。

## 相关概念

- [[MCP工作流程详解]]：了解完整的调用流程
- [[AI Agent架构设计]]：深入理解Agent的整体设计
- [[FastMCP框架应用]]：简化MCP开发的高级框架

## 总结

MCP通过清晰的C/S架构划分，实现了AI决策与工具执行的解耦。Server作为工具提供者，Client作为调用桥梁，LLM作为决策大脑，三者各司其职，构建了一个灵活、可扩展的工具调用体系。理解这些基础概念，是掌握MCP协议的第一步。