# tmux 进阶篇：环境配置与日志管理最佳实践

## 日志管理的核心问题

使用 tmux 时有一个容易被忽视的问题：**tmux 的滚动缓冲区是有限的**。即使使用 tmux 跑一晚上，第二天也不能查完整输出，因为输出超过缓冲区大小的内容会被覆盖。

> [!success]
> **完整运行历史的根本解决方案**
>
> 把输出持久化到文件（重定向/tee/日志系统）。

## tee 命令：边显示边保存

`tee` 是一个非常有用的命令，它可以将输出同时写到终端和文件，实现"一边显示一边保存"。

```bash
# 基本用法
python -u step2a_generate_complete_code.py 2>&1 | tee step2a.log
```

这个命令的含义：
- `python -u`：使用无缓冲模式运行 Python
- `2>&1`：把标准错误（stderr，编号 2）重定向到标准输出（stdout，编号 1），让两者合并
- `| tee step2a.log`：将合并后的输出同时显示在终端并写入日志文件

### 理解文件描述符重定向

`2>&1` 这个语法值得深入理解：

| 符号 | 含义 |
|------|------|
| `2` | 标准错误输出（stderr） |
| `1` | 标准输出（stdout） |
| `>` | 重定向 |
| `&` | 表示目标是文件描述符而不是文件路径 |

加上 `&`，表示 `>` 后面跟的是文件描述符（如 1、2），不是文件名。如 `>&1` 意味着"重定向到描述符 1"，而不是重定向到名为"1"的文件。

> [!info]
> **tee 命令的完整名称**
>
> `python -u xxx.py 2>&1 | tee xxx.log` 这样的命令叫"**使用 tee 命令同时输出到终端和日志文件**"的命令。

## 前台运行 vs 后台运行

### & 符号的作用

```bash
# 前台运行
python -u xxx.py > xxx.log 2>&1

# 后台运行
python -u xxx.py > xxx.log 2>&1 &
```

两者的区别：
- **不加 &**：脚本在前台运行，占用终端，直到结束；Ctrl+C 会终止执行
- **加 &**：脚本在后台运行，终端立即可用，你可继续输入其他命令；可用 ps 等方式查看状态

为什么执行 `python -u xxx.py > xxx.log 2>&1 &` 会马上回到提示符？因为末尾的 `&` 会把该命令放到后台执行，shell 不会等待任务完成，因此会立即返回提示符。

### nohup 命令

```bash
nohup python -u xxx.py > xxx.log 2>&1 &
```

| 命令 | 作用 |
|------|------|
| `nohup` | 忽略挂断信号（SIGHUP） |
| `&` | 丢后台运行 |

> [!tip]
> **tmux 与 nohup/& 的区别**
>
> - **tmux**：属于「会话管理 + 防断线」维度，用来保持会话不断开
> - **& / nohup**：属于「前台 / 后台运行」维度，用来控制进程是否占用终端
>
> 它们解决的是不同维度的问题，可以配合使用。

## 环境判断：WSL vs 远程服务器

### 如何判断 WSL 环境

```bash
# 如果路径以 /mnt/c/Users/... 开头，就说明是在 WSL 环境中
```

`/mnt/c/Users/...` 路径的含义：
- `mnt`：表示 mount，用于挂载外部文件系统的目录
- `c`：表示挂载的 Windows C 盘
- 这个路径只表示你当前访问的代码文件存放在 Windows 的 C 盘；并不代表 WSL 安装在 C 盘

在 WSL 下，如果代码在 D 盘，路径会变成 `/mnt/d/你的目录`。

### 宿主机 vs 容器

在宿主机和容器里运行 `tmux ls`，显示的会话是不一样的。因为 tmux 会话存储在各自系统的 `/tmp` 或用户目录下，宿主机和容器环境隔离，互不共享 tmux 会话信息。

> [!warning]
> **docker exec 的断线问题**
>
> `docker exec -it` 进容器后跑长脚本，如果 SSH 断了脚本也会挂，这是因为 exec 的本质类似于一次 SSH 连接：会话断开，内部任务也会随之终止。此时在容器内使用 tmux 可以解决这个问题。

## bashrc 配置与自定义别名

### bashrc 的作用

`bashrc` 是 **bash run commands** 的缩写，表示 bash 启动时自动执行的配置脚本。

bashrc 用来配置环境变量、别名等，让每次打开终端时自动生效。修改后需要执行 `source ~/.bashrc`，因为只有这样当前终端才能重新加载新配置。

> [!note]
> **bashrc 的执行时机**
>
> 当你启动一个新的交互式非登录 bash（比如打开终端或输入 bash）时，bashrc 就会被执行。
>
> 在终端里输入 `bash` 就会进入 bash shell 环境。

bashrc 只针对 bash shell，对其他 shell（如 zsh、fish）不起作用。在终端里输入 `bash` 会启动一个新的 bash 会话，进入 bash shell 环境。

### 实用的 tmux 别名配置

在 `~/.bashrc` 中添加以下别名可以大幅提升效率：

```bash
# 工作会话：连接或创建
alias work='tmux attach -t work || tmux new -s work'

# 会话管理
alias tl='tmux list-sessions'    # 列出所有 session
alias tn='tmux new-session -s'   # 新建 session
alias ta='tmux attach'           # 回到最近 session
alias tat='tmux attach -t'       # 回到指定 session
alias tad='tmux attach -d'       # 回到最近（踢掉其他连接）
alias tk='tmux kill-session -t'  # 删除某个 session
alias tka='tmux kill-server'     # 删除所有 session
```

> [!tip]
> **alias 的作用域**
>
> alias 是针对 Linux shell 本身的，与 tmux 无关。

## 常用命令行工具

### grep：行筛选

```bash
# grep 会逐行匹配，只输出包含指定关键词的行
grep "pattern" file.txt
```

grep 是 **Global Regular Expression Print** 的缩写，意思是"全局正则表达式打印"。是的，grep 会逐行匹配，只输出包含指定关键词的行。

### htop：实时监控

```bash
htop
```

htop 会实时显示系统的进程、CPU、内存等使用情况。按 `q` 键即可退出。

### tail：实时查看日志

```bash
tail -f xxx.log
```

运行 `tail -f xxx.log` 后想退出，按 `Ctrl + C` 即可停止并退出。

### clear：清空屏幕

```bash
clear
# 或快捷键 Ctrl + L
```

两者都会清屏，但不会删除命令历史。

## 每次新开终端的例行操作

每次新开终端后，在运行脚本前要做两件事：

1. 先用 `cd` 进入项目目录
2. 再用 `conda activate` 激活对应环境

## Windows 兼容性说明

> [!danger]
> **Windows PowerShell 不支持 tmux**
>
> 在 Windows PowerShell 中 tmux 不能用。tmux 属于 Linux/Unix 终端复用器，Windows 默认不支持。需要使用 WSL 或 Git Bash 等类 Unix 环境。

## 换行符差异

Windows/Unix/Linux 使用不同的换行符：

| 系统 | 换行符 |
|------|--------|
| Windows | CRLF（Carriage Return + Line Feed） |
| Unix/Linux | LF（Line Feed） |

**Carriage Return（CR）**：将字头滑架移动回行首
**Line Feed（LF）**：让纸张向上卷动一行

## 小结

掌握 tmux 的进阶使用，需要理解日志管理、环境判断、bashrc 配置等周边知识。通过 tee 命令实现日志的持久化，通过别名提升操作效率，通过环境判断避免踩坑。这些技能组合在一起，才能发挥 tmux 的真正威力。

---

**相关文章：**
- [[tmux 原理篇：理解终端复用与持久化会话机制]]
- [[tmux 实战篇：会话管理与窗口操作全攻略]]
