# FastAPI开发实战：从环境配置到项目架构

## FastAPI应用启动方式详解

FastAPI应用的启动方式灵活多样，最常见的两种方式在功能上是等价的：

```bash
# 方式一：直接运行Python文件
python code/app.py

# 方式二：使用uvicorn命令
uvicorn code.app:app --host 0.0.0.0 --port 9006
```

这两种启动方式本质上是相同的，因为`python code/app.py`会执行文件末尾的`uvicorn.run(...)`，启动的就是同一个FastAPI应用，初始化逻辑完全一致。

### 启用热重载功能

在开发阶段，热重载功能极大提升了开发效率。在`code/app.py`的`uvicorn.run(...)`配置中添加`reload=True`参数即可启用：

```python
uvicorn.run(app, host="0.0.0.0", port=9006, reload=True)
```

启用后，当代码被修改并保存时，Uvicorn会自动检测变化并重启服务，无需手动重启，大大加快了开发调试的迭代速度。

> [!warning] 关于uvloop的使用限制
> uvloop是一个用于Linux和macOS的高性能事件循环库，但需要注意的是，**Windows上无法编译或使用**uvloop。此外，uvloop只适用于基于asyncio的异步框架（如FastAPI、aiohttp），对于Flask这样的同步框架并无作用。

## API文档与接口测试

当FastAPI服务启动后，访问根路径（如http://127.0.0.1:9006）可能会显示`{"detail":"Not Found"}`，这表示服务已正常启动，但根路径没有定义路由，这是正常现象。

FastAPI自动生成的Swagger UI是开发者的重要工具，访问`http://127.0.0.1:9006/docs`（或`/redoc`）即可查看自动生成的接口文档。

### Swagger UI的使用价值

Swagger UI不仅提供接口文档，更重要的是它可以直接用于接口测试：
1. 在接口页面填写必要的参数
2. 点击"Execute"按钮发送请求
3. 查看返回结果，状态码200表示请求成功

这种内置的测试功能极大地方便了开发和调试过程。

## VS Code开发环境配置

### 终端与编辑器的分离设计

VS Code采用了"终端是终端，编辑器是编辑器"的设计理念：
- **终端**：由用户完全控制，可以开启多个终端窗口，切换不同的环境
- **编辑器**：Python解释器独立负责运行与调试功能

这种分离设计避免了环境混乱，降低了出错概率。

### 虚拟环境管理

在使用conda虚拟环境时，需要理解两个关键概念的区别：
- **(career_env)**：终端当前激活的Python环境
- **Select Interpreter**：VS Code编辑器内部绑定的Python解释器路径

> [!tip] 环境隔离的重要提醒
> VS Code的Select Interpreter选择**不会**改变pip的安装路径。它只影响VS Code运行、调试和代码补全时使用的解释器。pip的安装位置由终端当前激活的环境决定。

要在career_env激活状态下安装包，只需在终端执行：
```bash
pip install package_name
```

包会被安装到当前激活的career_env环境中，而不是主环境(base)。同样，VS Code的解释器选择也不会改变虚拟环境本身，只是让编辑器跟随并使用指定环境。

查看虚拟环境的安装位置：
```bash
conda env list
```

## FastAPI项目架构设计

### 核心目录结构

一个规范的FastAPI项目通常采用分层架构：

```
project/
├── app.py              # FastAPI应用入口文件
├── middleware/         # 中间件层
│   ├── auth_middleware.py  # 鉴权中间件
│   └── limiter.py          # 限流中间件
├── routers/            # 路由层
├── service/            # 业务逻辑层
└── config/             # 配置层
```

### 入口文件的作用

app.py作为FastAPI应用的入口文件，主要承担以下职责：
1. **加载配置**：加载环境变量、日志设置、数据库连接、模型路径等全局配置
2. **注册中间件**：按顺序注册CORS、限流、鉴权等中间件
3. **注册路由模块**：将各个功能模块的路由注册到应用中

### 中间件系统

中间件是介于请求与响应之间的处理层，用于统一拦截和处理逻辑：

- **CORS中间件**：处理跨域资源共享，允许前端访问API
- **限流中间件**：控制请求频率，防止接口被滥用（如使用SlowAPI第三方库）
- **鉴权中间件**：验证用户身份，确保只有合法用户能访问资源

> [!note] 中间件执行顺序
> 中间件在请求到达路由之前执行，所以鉴权、限流等检查会在业务逻辑处理前完成。

### 分层架构职责

- **路由层（routers/）**：处理HTTP请求与响应，解析请求参数，调用业务逻辑层并返回响应结果
- **业务层（service/）**：实现核心业务逻辑，包括构建Prompt、调用大模型、组织结果等
- **工具层**：提供可复用的通用工具函数

配置层"贯穿各层"，意味着各层都会通过`from app.config import ai_config`等方式读取配置。

## 流式响应处理

FastAPI支持流式响应（SSE），特别适合处理耗时较长的AI任务。在Postman中查看流式响应时，会看到连续的数据流：

```
data: {"type":"chunk","content":"您好"}
data: {"type":"chunk","content":"！"}
data: {"type":"chunk","content":"欢迎"}
...
data: {"type":"DONE","content":"[DONE]"}
```

流式响应的结束标记是：`{"type":"DONE","content":"[DONE]"}`

### 请求头配置

在测试API时，需要正确设置请求头：
- Key: `Content-Type`
- Value: `application/json`

## 相关阅读

- [[外包项目管理全流程实战指南]] - 了解项目管理和交付流程
- [[AI面试系统架构设计与实现详解]] - 深入了解基于FastAPI的AI系统实现