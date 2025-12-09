# 实战MCP开发：使用FastMCP构建高效的工具服务

## FastMCP简介

FastMCP是专门用于构建MCP Server和Client的Python框架。它不是临时项目，而是已经成为官方MCP Python SDK的一部分。FastMCP的核心价值在于**封装了MCP协议的底层细节**，让开发者能够像写普通函数一样构建MCP服务。

> [!tip] 为什么选择FastMCP？
> 如果你想构建MCP服务器，FastMCP是官方推荐的首选框架。它简化了开发流程，让你专注于业务逻辑而非协议细节。

## MCP工具注册方式

MCP提供两种主要的工具注册方式：

### 1. Decorator注册（主流方式）
这是最简单易用的方式，通过装饰器来标记函数、资源和提示模板。

```python
from mcp.server.fastmcp import FastMCP

mcp = FastMCP("My MCP Server")

@mcp.tool()
def calculate_sum(a: int, b: int) -> int:
    """计算两个数字的和"""
    return a + b

@mcp.resource("data://users/{user_id}")
def get_user_data(user_id: str) -> str:
    """获取用户数据"""
    return f"User data for {user_id}"

@mcp.prompt()
def analysis_prompt() -> str:
    """数据分析提示模板"""
    return "请分析以下数据..."
```

### 2. Schema注册
这是更底层的实现方式，需要手动实现注册逻辑。虽然更灵活，但也更复杂，一般只在特殊需求时使用。

## "注册"的准确含义

在MCP上下文中，"注册"指的是将功能添加到MCP Server的能力列表中，使其能够被外部调用。具体来说：

- 将tool/resource/prompt添加到MCP Server的注册表
- 暴露相关的元数据和Schema信息
- 使功能对外部系统可见和可用

函数被注册为tool后，LLM或客户端就能发现并调用这个工具。

## 开发MCP工具的典型场景

以下情况适合开发自定义MCP工具：

### 1. 数据库操作
当需要让AI助手访问或操作数据库时：
```python
@mcp.tool()
def query_database(sql: str) -> dict:
    """执行SQL查询并返回结果"""
    # 数据库查询逻辑
    return results
```

### 2. 文件处理
当需要处理特定格式的文件时：
```python
@mcp.tool()
def parse_csv(file_path: str) -> dict:
    """解析CSV文件"""
    # CSV解析逻辑
    return parsed_data
```

### 3. 内部API调用
当需要集成企业内部系统时：
```python
@mcp.tool()
def call_internal_api(endpoint: str, params: dict) -> dict:
    """调用内部API"""
    # API调用逻辑
    return response
```

### 4. 计算密集型操作
当需要执行复杂计算时：
```python
@mcp.tool()
def optimize_schedule(tasks: list) -> dict:
    """优化任务调度"""
    # 优化算法
    return optimized_schedule
```

## 将现有服务转换为MCP工具

一个强大的特性是，**现有服务不需要重写**就能转成MCP tool。你只需要添加一层adapter/wrapper包装即可：

```python
# 原有函数
def existing_complex_function(param1, param2):
    # 复杂的业务逻辑
    return result

# MCP包装器
@mcp.tool()
def wrapped_function(param1: str, param2: int) -> dict:
    """包装后的函数，提供MCP接口"""
    return {
        "result": existing_complex_function(param1, param2),
        "metadata": {
            "timestamp": datetime.now().isoformat()
        }
    }
```

这种设计让你能够：
- 复用现有代码
- 保持业务逻辑不变
- 添加必要的类型注解和文档
- 提供统一的错误处理

## FastMCP的核心特性

### 1. 简化的开发体验
FastMCP让你能够：
- 使用熟悉的Python语法
- 专注于业务逻辑
- 自动处理协议细节
- 获得完整的类型提示

### 2. 丰富的功能支持
除了基本的工具函数，FastMCP还支持：
- **资源暴露**：文件、数据库等静态或动态资源
- **提示模板**：预定义的提示词，提高AI响应质量
- **元数据管理**：完整的工具描述和参数说明

### 3. 灵活的部署选项
开发完成后，你可以：
- 作为独立服务运行
- 集成到现有系统
- 容器化部署
- 云原生部署

## 开发最佳实践

### 1. 类型注解
始终使用类型注解，这不仅能提供更好的文档，还能帮助MCP正确解析参数：

```python
@mcp.tool()
def process_data(
    data: List[Dict[str, Any]],
    options: Optional[ProcessingOptions] = None
) -> ProcessingResult:
    """处理数据并返回结果"""
    pass
```

### 2. 完整的文档
为每个工具提供清晰的文档，包括：
- 功能说明
- 参数描述
- 返回值说明
- 使用示例

### 3. 错误处理
实现健壮的错误处理：
```python
@mcp.tool()
def safe_operation(input_data: str) -> dict:
    """执行安全操作"""
    try:
        # 业务逻辑
        result = perform_operation(input_data)
        return {"status": "success", "data": result}
    except Exception as e:
        return {"status": "error", "message": str(e)}
```

### 4. 资源管理
对于需要外部资源的操作，确保正确管理资源：
```python
@mcp.tool()
def process_large_file(file_path: str) -> dict:
    """处理大文件"""
    with open(file_path, 'r') as f:
        # 流式处理，避免内存问题
        for line in f:
            process_line(line)
    return {"status": "completed"}
```

## 测试MCP服务器

开发完成后，你可以通过以下方式测试：

1. **直接测试**：使用MCP客户端直接调用
2. **集成测试**：在IDE中测试工具调用
3. **单元测试**：为每个函数编写测试用例

```bash
# 启动MCP服务器
python -m mcp.server.fastmcp

# 测试工具调用
mcp call your_tool_name '{"param": "value"}'
```

---

相关文章：
- [[深入理解MCP：模型上下文协议的核心概念与架构设计]]
- [[MCP服务器部署与管理指南：从配置到运行的完整流程]]
- [[AI工具调用机制解析：从Function Call到MCP的演进]]