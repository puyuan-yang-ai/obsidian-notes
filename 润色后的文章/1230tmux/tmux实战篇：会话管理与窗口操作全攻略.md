# tmux 实战篇：会话管理与窗口操作全攻略

## tmux 的三层结构

理解 tmux 的操作，首先要掌握它的三层架构：**session → window → pane**

```
session（会话）→ window（窗口）→ pane（面板）
```

这三个层级可以类比为：
- **session**：一个大的工作台（像一个书包）
- **window**：工作台里的标签页（书包里的盒子）
- **pane**：标签页中的小格子（盒子里的物品）

> [!info]
> **脚本的运行层级**
>
> 脚本只会运行在 **pane** 里。window 是装 pane 的"盒子"，session 是装盒子的"书包"。

主流用法是：**1 个 session，配少量 window，每个 window 里开 1～2 个 pane。** 每次新开终端本质上是建立新的 SSH 连接，而使用 tmux 后，连上某台机器或容器后，通常只固定用 **一个 session**，在里面开多个 window 来区分任务。

## 会话管理操作

### 创建会话

```bash
# 创建一个名为 "hip" 的新会话
tmux new -s hip

# new 是 new-session 的简写，两者等价
tmux new-session -s hip
```

### 脱离会话

脱离会话的意思是退出 tmux 界面，但让会话和其中的程序继续在后台运行。

```bash
# 快捷键方式：Ctrl + b，松开后再按 d
```

> [!tip]
> **让脚本在后台运行**
>
> 使用 tmux 跑起任务之后，如果是远程服务器上运行，可以放心关机（本地电脑关机不会影响远程服务器）。但如果是本地 WSL 上运行，不能关机，关机会关闭 Windows 和 WSL，tmux 会话和脚本都会终止。

### 查看所有会话

```bash
# 列出所有会话
tmux ls

# ls 是 list-sessions 的简写
tmux list-sessions
```

tmux ls 显示的是所有 **session**，而不是 window。

### 重新连接会话

```bash
# 连接到名为 "hip" 的会话
tmux attach -t hip

# attach 会自动连接到上一次使用的会话
tmux attach

# -t 是 target-session 的缩写，用来指定目标会话
```

`tmux attach` 和 `tmux attach -t <会话名>` 的区别是：前者会自动连接到上一次使用的会话；后者用于明确连接到指定会话。

### 一键连接或创建

这是一个非常实用的技巧：

```bash
# 先尝试连接名为 hip 的 session；如果不存在，就新建一个
tmux attach -t hip || tmux new -s hip
```

常用于方便地进入同一个工作环境。

### 删除会话

```bash
# 删除名为 "hip" 的会话
tmux kill-session -t hip
```

## 理解 kill-session 与 kill-window

tmux 中有两个容易混淆的删除操作：

- **kill-session**：删除整个会话
- **kill-window**：只删除指定窗口，不影响其他窗口或会话

如果一个 session 包含多个 window，使用 `kill-session` 会直接删除整个会话，比逐个删除窗口更干净、更快捷。

> [!danger]
> **选择正确的删除命令**
>
> 当 `tmux ls` 显示有多个会话（如 hip 和 step2）时，应该用 `kill-session`，因为 hip 和 step2 都是独立会话。

## Window 和 Pane 操作

### Window 操作

创建、切换、重命名 window 的快捷键：

| 操作 | 快捷键 |
|------|--------|
| 新建 window | `Ctrl + b` 然后 `c` |
| 重命名 window | `Ctrl + b` 然后 `,` |
| 切换 window（按编号） | `Ctrl + b` 然后 `0-9` |
| 切换 window（上一/下一） | `Ctrl + b` 然后 `n/p` |
| 结束 window | 在 window 中输入 `exit` 或 `Ctrl + b` 然后 `&` |

> [!info]
> **tmux 底部状态栏含义**
>
> 底部绿色区域显示 `[hip] 0:bash  1:tcsh-  2:tcsh*` 表示：
> - `hip`：session 名
> - `0:bash`：第 0 号 window，名字叫 bash
> - `*`：当前所在的 window
> - 这个 session 里面一共有 3 个 windows

### Pane 操作

一个 window 可以被分割成多个 pane（分屏）：

| 操作 | 快捷键 |
|------|--------|
| 切换 pane | `Ctrl + b` 然后方向键 |
| 停止运行脚本 | 切换到对应 pane 按 `Ctrl + C` |

> [!tip]
> **分屏场景**
>
> tmux 里"一开就是三分屏"就是一个终端窗口被分成三块独立的小窗，每块都有自己的 shell，可以同时操作不同任务。每个 pane 都是独立的终端环境，互不干扰。

## 复制模式：查看历史输出

默认情况下，tmux 中鼠标滚动无法查看历史输出。需要进入**复制模式**：

```bash
# 进入复制模式
Ctrl + b 然后 [

# 退出复制模式
按 q 或 Esc
```

进入复制模式后，可以用方向键滚动查看历史输出。需要注意的是，复制模式不会显示终端的实时变化。

> [!warning]
> **tmux 缓冲区限制**
>
> tmux 只保存有限的滚动缓冲区（history-limit），超过部分会被覆盖，不会自动保留完整日志。使用 tmux 跑一晚上，第二天不能查完整输出，除非提前将日志重定向到文件。

## 停止运行中的程序

在 tmux 里停止正在运行的脚本，需要切换到运行脚本的那个 pane，然后按下 `Ctrl + C`。`Ctrl + C` 会向当前前台进程发送中断信号（SIGINT），通常用来停止正在运行的程序。

## 推荐的工作流配置

一个实用的 tmux 工作流配置：

| Session | Window | 用途 |
|---------|--------|------|
| work | 0:run | 用来运行脚本或程序 |
| work | 1:logs | 查看日志输出 |
| work | 2:git | 进行代码管理和提交 |
| work | 3:monitor | 监控系统/硬件资源或进程状态 |

这样的结构让你在一个 session 中就能完成开发、调试、监控的全部工作。

## 小结

tmux 的强大在于它将终端操作系统化。通过 session → window → pane 三层结构，你可以灵活地组织工作空间；通过复制模式，你可以方便地查看历史输出；通过会话管理，你可以随时断开和恢复工作。掌握这些操作后，远程开发效率将大幅提升。

---

**相关文章：**
- [[tmux 原理篇：理解终端复用与持久化会话机制]]
- [[tmux 进阶篇：环境配置与日志管理最佳实践]]
