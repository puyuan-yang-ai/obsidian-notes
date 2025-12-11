# FastMCP框架应用

## FastMCP概述

FastMCP是一个现代化的MCP（Model Context Protocol）开发框架，它通过Python装饰器和高级抽象大大简化了MCP服务器和客户端的开发。FastMCP的设计理念是"约定优于配置"，让开发者能够快速构建功能强大的MCP应用。

> [!tip] 为什么要使用FastMCP？
FastMCP不仅简化了server开发，同时对client开发也有帮助。它提供了更Pythonic、高层、简洁的接口，相比官方SDK能减少70%以上的样板代码。

## FastMCP核心特性

### 1. 装饰器驱动的开发模式

FastMCP大量使用装饰器来简化代码：

```python
from fastmcp import FastMCP

mcp = FastMCP("My Awesome MCP Server")

@mcp.tool()
def calculate(expression: str) -> str:
    """计算数学表达式"""
    try:
        result = eval(expression)
        return f"结果: {result}"
    except Exception as e:
        return f"错误: {str(e)}"

@mcp.resource()
def config_file() -> str:
    """获取配置文件内容"""
    return open("config.json").read()
```

### 2. 自动Schema生成

FastMCP自动从函数签名生成JSON Schema：

```python
# FastMCP自动生成schema，无需手动编写
@mcp.tool()
def search_files(
    path: str,
    pattern: str = "*.py",
    recursive: bool = False
) -> list[str]:
    """搜索文件"""
    # 实现逻辑
    pass
```

相比之下，官方SDK需要手动定义schema：

```python
# 官方SDK需要更多样板代码
def register_tools():
    return {
        "search_files": {
            "name": "search_files",
            "description": "搜索文件",
            "inputSchema": {
                "type": "object",
                "properties": {
                    "path": {"type": "string"},
                    "pattern": {"type": "string", "default": "*.py"},
                    "recursive": {"type": "boolean", "default": False}
                },
                "required": ["path"]
            }
        }
    }
```

## Server开发实践

### 基础Server实现

```python
from fastmcp import FastMCP

# 创建FastMCP实例
mcp = FastMCP("File System Tools")

# 定义工具
@mcp.tool()
def read_file(path: str) -> str:
    """读取文件内容"""
    with open(path, 'r', encoding='utf-8') as f:
        return f.read()

@mcp.tool()
def write_file(path: str, content: str) -> str:
    """写入文件内容"""
    with open(path, 'w', encoding='utf-8') as f:
        f.write(content)
    return f"成功写入 {path}"

@mcp.tool()
def list_directory(path: str) -> list[dict]:
    """列出目录内容"""
    import os
    items = []
    for item in os.listdir(path):
        full_path = os.path.join(path, item)
        items.append({
            "name": item,
            "type": "directory" if os.path.isdir(full_path) else "file"
        })
    return items

# 定义资源
@mcp.resource("config://app")
def get_app_config() -> dict:
    """获取应用配置"""
    return {
        "theme": "dark",
        "language": "zh-CN",
        "timeout": 30
    }

# 定义提示模板
@mcp.prompt()
def code_review_prompt(file_path: str) -> str:
    """生成代码审查提示"""
    content = read_file(file_path)
    return f"""请审查以下代码：

文件: {file_path}

{content}

请检查：
1. 代码质量和最佳实践
2. 潜在的安全问题
3. 性能优化建议
"""

# 启动服务器
if __name__ == "__main__":
    mcp.run()
```

### 高级功能

#### 1. 异步支持

```python
import asyncio
import aiohttp

@mcp.tool()
async def fetch_url(url: str) -> str:
    """异步获取URL内容"""
    async with aiohttp.ClientSession() as session:
        async with session.get(url) as response:
            return await response.text()

@mcp.tool()
async def parallel_search(queries: list[str]) -> list[dict]:
    """并行搜索多个查询"""
    tasks = [search_single(q) for q in queries]
    results = await asyncio.gather(*tasks)
    return results
```

#### 2. 错误处理

```python
from fastmcp import FastMCP, McpError

@mcp.tool()
def safe_divide(a: float, b: float) -> float:
    """安全的除法操作"""
    if b == 0:
        raise McpError("除数不能为零")
    return a / b

# 全局错误处理
@mcp.error_handler
def handle_error(error):
    """统一错误处理"""
    if isinstance(error, FileNotFoundError):
        return f"文件未找到: {error.filename}"
    return f"发生错误: {str(error)}"
```

#### 3. 中间件支持

```python
@mcp.middleware
async def logging_middleware(context, next_handler):
    """日志中间件"""
    print(f"调用工具: {context.get('tool_name')}")
    result = await next_handler(context)
    print(f"工具执行完成")
    return result

@mcp.middleware
async def auth_middleware(context, next_handler):
    """认证中间件"""
    api_key = context.get("headers", {}).get("authorization")
    if not validate_api_key(api_key):
        raise McpError("认证失败")
    return await next_handler(context)
```

## Client开发实践

### 基础Client使用

```python
from fastmcp import FastMCPClient

# 创建客户端
client = FastMCPClient("stdio", command="python server.py")

# 连接到服务器
await client.connect()

# 列出可用工具
tools = await client.list_tools()
print(f"可用工具: {[tool.name for tool in tools]}")

# 调用工具
result = await client.call_tool("read_file", {"path": "example.txt"})
print(result.content[0].text)

# 获取资源
resource = await client.read_resource("config://app")
print(resource.contents[0].text)
```

### 客户端高级用法

#### 1. 工具包装器

```python
class MCPHelper:
    def __init__(self, client):
        self.client = client

    async def read_file(self, path: str) -> str:
        """便捷的文件读取方法"""
        result = await self.client.call_tool("read_file", {"path": path})
        return result.content[0].text

    async def write_file(self, path: str, content: str) -> None:
        """便捷的文件写入方法"""
        await self.client.call_tool("write_file", {"path": path, "content": content})

    async def list_directory(self, path: str) -> list[dict]:
        """便捷的目录列表方法"""
        result = await self.client.call_tool("list_directory", {"path": path})
        return json.loads(result.content[0].text)

# 使用
helper = MCPHelper(client)
content = await helper.read_file("test.txt")
```

#### 2. 流式响应

```python
async def stream_large_file(client: FastMCPClient, path: str):
    """流式读取大文件"""
    result = await client.call_tool("read_file_stream", {"path": path})

    async for chunk in result.content:
        if chunk.type == "text":
            print(chunk.text, end="")
        elif chunk.type == "error":
            print(f"\n错误: {chunk.text}")
            break
```

## 与官方SDK对比

### 代码量对比

| 功能 | FastMCP | 官方SDK | 减少比例 |
|------|---------|---------|----------|
| 定义工具 | 5行 | 20行 | 75% |
| 定义资源 | 3行 | 15行 | 80% |
| 错误处理 | 1行 | 10行 | 90% |
| 总体样板代码 | 30% | 100% | 70% |

### 功能对比

| 特性 | FastMCP | 官方SDK |
|------|---------|---------|
| 装饰器支持 | ✅ | ❌ |
| 自动Schema生成 | ✅ | ❌ |
| 异步支持 | ✅ | ✅ |
| 中间件 | ✅ | ❌ |
| 类型提示 | ✅ | 部分 |
| 文档生成 | ✅ | ❌ |

### 性能对比

FastMCP在运行时性能上与官方SDK相当，因为：
- 底层都使用相同的MCP协议
- FastMCP主要是开发时的语法糖
- 编译后没有额外的运行时开销

## 最佳实践

### 1. 项目结构

```
my_mcp_project/
├── server/
│   ├── __init__.py
│   ├── main.py          # FastMCP服务器定义
│   ├── tools/           # 工具实现
│   │   ├── __init__.py
│   │   ├── file_ops.py
│   │   └── data_ops.py
│   └── resources/       # 资源定义
│       └── __init__.py
├── client/
│   ├── __init__.py
│   └── client.py        # FastMCP客户端
└── tests/
    ├── test_server.py
    └── test_client.py
```

### 2. 配置管理

```python
# config.py
from pydantic import BaseSettings

class Settings(BaseSettings):
    server_name: str = "My MCP Server"
    host: str = "localhost"
    port: int = 8080
    debug: bool = False

    class Config:
        env_file = ".env"

# main.py
from fastmcp import FastMCP
from config import Settings

settings = Settings()
mcp = FastMCP(settings.server_name)

if __name__ == "__main__":
    mcp.run(host=settings.host, port=settings.port)
```

### 3. 测试策略

```python
import pytest
from fastmcp.testing import FastMCPTestClient
from server.main import mcp

@pytest.fixture
def test_client():
    return FastMCPTestClient(mcp)

async def test_read_file(test_client):
    """测试文件读取工具"""
    result = await test_client.call_tool(
        "read_file",
        {"path": "test.txt"}
    )
    assert result.content[0].text == "Hello World"

async def test_config_resource(test_client):
    """测试配置资源"""
    resource = await test_client.read_resource("config://app")
    assert "theme" in resource.contents[0].text
```

## 迁移指南

### 从官方SDK迁移到FastMCP

1. **工具定义迁移**
   ```python
   # 官方SDK
   def get_tools():
       return {
           "my_tool": {
               "name": "my_tool",
               "description": "My tool",
               "inputSchema": {...}
           }
       }

   async def handle_call(tool_name, arguments):
       if tool_name == "my_tool":
           return await my_tool(arguments)

   # FastMCP
   @mcp.tool()
   async def my_tool(arg1: str, arg2: int = 10) -> str:
       """My tool"""
       return await implement_tool(arg1, arg2)
   ```

2. **服务器启动迁移**
   ```python
   # 官方SDK
   async def main():
       server = Server("my-server")
       async with server.run() as running:
           await running.wait_until_closed()

   # FastMCP
   if __name__ == "__main__":
       mcp.run()
   ```

## 相关概念

- [[MCP基础概念与架构]]：理解MCP的基本原理
- [[MCP工作流程详解]]：了解MCP的完整调用流程
- [[AI Agent架构设计]]：FastMCP在Agent系统中的应用

## 总结

FastMCP通过装饰器、自动Schema生成、中间件等高级特性，大大简化了MCP应用的开发。它不仅减少了样板代码，还提高了开发效率和代码可维护性。对于新的MCP项目，推荐使用FastMCP作为首选框架。对于现有项目，可以考虑逐步迁移到FastMCP以获得更好的开发体验。