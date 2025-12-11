# CLI程序开发实践

## CLI基础概念

### 什么是CLI程序

CLI（Command Line Interface）程序是通过命令行操作的程序，它可以在终端中直接用命令名运行。CLI程序的本质是一个可执行文件，操作系统知道如何找到并运行它。

> [!tip] CLI vs 终端
> CLI不等于终端。终端是输入命令的窗口（如Terminal、CMD），CLI是在终端中运行的命令（如ls、git、python）。

### CLI程序的三要素

一个Python脚本要成为CLI程序，需要满足三个条件：

1. **系统可以找到**（在PATH中）
2. **知道如何运行**（有shebang或entry point）
3. **有执行权限**（可执行）

## CLI实现方式

### 方式一：Shebang + 可执行权限

这是Unix/Linux系统的传统方式。

```python
#!/usr/bin/env python3
# mytool.py

import sys
print(f"Hello from {sys.argv[0]}")
```

实现步骤：

```bash
# 1. 添加可执行权限
chmod +x mytool.py

# 2. 移动到PATH中的目录
sudo mv mytool.py /usr/local/bin/mytool

# 3. 现在可以直接运行
mytool
```

### 方式二：Python包安装（推荐）

这是更现代、跨平台的方式。

#### 项目结构

```
mytool/
├── pyproject.toml
├── mytool/
│   ├── __init__.py
│   └── cli.py
```

#### pyproject.toml配置

```toml
[build-system]
requires = ["setuptools>=45", "wheel"]
build-backend = "setuptools.build_meta"

[project]
name = "mytool"
version = "0.1.0"
description = "My awesome CLI tool"

[project.scripts]
mytool = "mytool.cli:main"
```

#### CLI实现

```python
# mytool/cli.py
import typer

app = typer.Typer(help="My awesome CLI tool")

@app.command()
def main(name: str = "World"):
    """Greet someone"""
    typer.echo(f"Hello, {name}!")

if __name__ == "__main__":
    app()
```

#### 安装和使用

```bash
# 安装到开发模式
pip install -e .

# 现在可以直接运行
mytool
mytool --name Alice
```

## 参数解析

### argparse（标准库）

```python
import argparse

parser = argparse.ArgumentParser(description="Process some integers.")
parser.add_argument("integers", metavar="N", type=int, nargs="+",
                   help="an integer for the accumulator")
parser.add_argument("--sum", dest="accumulate", action="store_const",
                   const=sum, default=max,
                   help="sum the integers (default: find the maximum)")

args = parser.parse_args()
print(args.accumulate(args.integers))
```

### typer（推荐）

Typer是argparse的现代化替代，提供了更好的类型提示和自动生成帮助文档。

```python
import typer

app = typer.Typer()

@app.command()
def process(
    input_file: str = typer.Argument(..., help="Input file path"),
    output: str = typer.Option(..., "--output", "-o", help="Output file path"),
    verbose: bool = typer.Option(False, "--verbose", "-v", help="Verbose output")
):
    """Process a file"""
    typer.echo(f"Processing {input_file}...")
    if verbose:
        typer.echo("Verbose mode enabled")
    typer.echo(f"Output written to {output}")

if __name__ == "__main__":
    app()
```

## CLI包装器开发

### 什么是CLI包装器

CLI包装器是给某个功能添加命令行入口的程序。比如Docker就是一个CLI包装器，将复杂的容器管理功能封装成简单的命令。

```bash
# docker run nginx
# docker就是包装器，run nginx是参数
```

### 实现MCP工具的CLI包装器

```python
#!/usr/bin/env python3
"""
amd-tool.py - AMD MCP工具的CLI包装器
"""

import sys
import json
import argparse
from mcp_client import MCPClient

def main():
    parser = argparse.ArgumentParser(description="AMD MCP Tool CLI Wrapper")
    parser.add_subcommand = parser.add_subparsers(dest="command", required=True)

    # query子命令
    query_parser = parser.add_subcommand.add_parser("query")
    query_parser.add_argument("params", help="JSON string of parameters")

    # list子命令
    list_parser = parser.add_subcommand.add_parser("list")
    list_parser.add_argument("--format", choices=["json", "table"], default="table")

    args = parser.parse_args()

    try:
        client = MCPClient()

        if args.command == "query":
            params = json.loads(args.params)
            result = client.call_tool("query", params)
            print(json.dumps(result, indent=2))

        elif args.command == "list":
            result = client.call_tool("list_resources", {})
            if args.format == "json":
                print(json.dumps(result, indent=2))
            else:
                for resource in result["resources"]:
                    print(f"- {resource['name']}")

    except Exception as e:
        print(f"Error: {e}", file=sys.stderr)
        sys.exit(1)

if __name__ == "__main__":
    main()
```

## 入口点机制深入

### entry point的工作原理

当在`pyproject.toml`中定义：

```toml
[project.scripts]
mytool = "mytool.cli:main"
```

pip安装时会：
1. 在PATH目录创建名为`mytool`的可执行文件
2. 这个文件实际上是Python脚本的启动器
3. 启动器会导入`mytool.cli`模块并调用`main`函数

### 生成的可执行文件示例

```bash
# 在Python环境的bin目录下
cat mytool
#!/path/to/python
# -*- coding: utf-8 -*-
import re
import sys
from mytool.cli import main
if __name__ == "__main__":
    sys.argv[0] = re.sub(r"(-script\.pyw|\.exe)?$", "", sys.argv[0])
    sys.exit(main())
```

## 最佳实践

### 1. 使用Typer或Click

```python
# 推荐使用现代CLI框架
import typer

app = typer.Typer(help="Modern CLI tool")

@app.command()
@typer.argument("input_file", exists=True)
@typer.option("--output", "-o", help="Output file")
@typer.option("--verbose", "-v", is_flag=True)
def process(input_file: Path, output: Optional[Path] = None, verbose: bool = False):
    """Process files with modern CLI"""
    pass
```

### 2. 提供良好的帮助信息

```python
@app.command()
def complex_command(
    # 必需参数
    input_file: Path = typer.Argument(..., help="Input file to process"),

    # 可选参数带默认值
    threshold: float = typer.Option(0.5, help="Processing threshold (0-1)"),

    # 带选择的参数
    format: str = typer.Option("json", help="Output format",
                               click_type=Choice(["json", "yaml", "csv"])),

    # 标志参数
    force: bool = typer.Option(False, help="Force overwrite existing files")
):
    """A complex command with comprehensive help"""
    pass
```

### 3. 错误处理和用户友好

```python
def main():
    try:
        # 主逻辑
        pass
    except FileNotFoundError as e:
        typer.echo(f"Error: File not found - {e}", err=True)
        raise typer.Exit(1)
    except PermissionError:
        typer.echo("Error: Permission denied", err=True)
        raise typer.Exit(1)
    except Exception as e:
        typer.echo(f"Unexpected error: {e}", err=True)
        raise typer.Exit(1)
```

### 4. 配色和进度条

```python
from rich.console import Console
from rich.progress import track

console = Console()

# 彩色输出
console.print("Success!", style="bold green")
console.print("Warning!", style="bold yellow")

# 进度条
for item in track(tasks, description="Processing..."):
    process(item)
```

## 常见问题和解决方案

### Q1: 为什么`pip install -e .`后还是找不到命令？

**原因**：可能没有在`pyproject.toml`中定义entry point。

**解决**：
```toml
[project.scripts]
mytool = "mytool.cli:main"  # 确保这行存在
```

### Q2: 如何让单个Python文件成为CLI？

```bash
# 方法1：使用shebang
#!/usr/bin/env python3
print("Hello CLI")
# chmod +x file.py
# ./file.py

# 方法2：创建最小包结构
project/
├── pyproject.toml
└── tool.py
# pyproject.toml中定义entry point
```

### Q3: CLI如何接收管道输入？

```python
import sys

def main():
    # 检查是否有管道输入
    if not sys.stdin.isatty():
        data = sys.stdin.read()
        process_data(data)
    else:
        print("No pipe input detected")
```

### Q4: 如何处理复杂的参数？

```python
import json

@app.command()
def complex_params(config: str):
    """Accept complex parameters as JSON"""
    params = json.loads(config)
    # 处理复杂参数
    pass

# 使用方式
mytool complex-params '{"key": "value", "list": [1, 2, 3]}'
```

## 相关概念

- [[MCP集成实现方案]]：了解CLI包装器在MCP集成中的应用
- [[进程间通信机制]]：理解CLI程序间的通信方式
- [[FastMCP框架应用]]：查看FastMCP如何简化CLI开发

## 总结

CLI程序开发是构建命令行工具的基础技能。通过理解CLI的本质、掌握现代开发框架、遵循最佳实践，可以创建出专业、易用的命令行工具。记住，一个好的CLI程序应该：
- 安装简单（在PATH中）
- 帮助友好（清晰的帮助文档）
- 错误处理完善
- 遵循Unix哲学（做好一件事）