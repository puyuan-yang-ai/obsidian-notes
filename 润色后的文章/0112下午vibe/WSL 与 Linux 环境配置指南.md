# WSL 与 Linux 环境配置指南

## 概述

在 Windows 上使用 AI 开发工具链时，WSL（Windows Subsystem for Linux）是必不可少的桥梁。许多工具（如 [[SpecStory]]）不支持 Windows 原生环境，必须通过 WSL 运行。本文将介绍 WSL 的基本操作、包管理器使用以及常见问题的解决方法。

## WSL 身份切换

在日常开发中，有时需要以 root 权限执行操作，有时需要以普通用户身份工作。WSL 提供了灵活的身份切换机制：

### 不同身份进入 WSL

```bash
# 以 root 角色进入
wsl -d Ubuntu -u root

# 以普通用户进入（默认）
wsl
```

> [!info] **参数说明**
> - `-d Ubuntu`：指定发行版名称（可以是 Ubuntu、Debian 等）
> - `-u root`：指定用户身份
> - 省略参数时使用默认配置

### 何时使用 root？

需要系统级操作时使用 root 身份：
- 安装系统包（apt install）
- 修改系统配置
- 移动文件到 `/usr/local/bin`

> [!warning] **安全提示**
> - 日常开发尽量避免使用 root
> - 只有必要时才切换到 root 身份
> - 使用 sudo 比直接 root 更安全

## Homebrew 包管理器

Homebrew 是 macOS 和 Linux 上流行的包管理器，但两个系统的默认情况不同：

### 系统差异

| 系统 | Homebrew 默认情况 |
|------|------------------|
| macOS | 几乎默认存在，预装率高 |
| Linux / WSL | **默认没有**，需要手动安装 |

### 安装 Homebrew（Linux/WSL）

```bash
# 安装 Homebrew
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"

# 添加到 PATH（根据系统提示执行）
echo 'eval "$(/home/linuxbrew/.linuxbrew/bin/brew shellenv)"' >> ~/.zshrc
source ~/.zshrc
```

> [!tip] **为什么需要 Homebrew？**
> - 统一的包管理体验
> - 许多开发工具提供 brew tap
> - 比 apt 更新更及时（如 SpecStory）

## 常见错误与解决

Linux/WSL 环境下最常见的问题是 `command not found`，这类错误有一个通用规律：

### 90% 原因法则

> [!danger] **command not found 的真相**
> - 90% 的原因是：**这个工具根本没有安装**
> - 不要怀疑是 PATH 问题或权限问题
> - 先尝试安装，而非调试配置

### 案例：unzip 未安装

在 WSL 里解压文件时遇到 `zsh: command not found: unzip` 错误：

```bash
# 解决方法：安装 unzip
sudo apt update
sudo apt install -y unzip
```

### 案例：wget 未安装

```bash
# 解决方法：安装 wget
sudo apt update
sudo apt install -y wget
```

### 通用排查流程

```
遇到 command not found
    ↓
尝试安装该工具
    ↓
如果 apt 找不到包
    ↓
检查包名是否正确
    ↓
考虑使用其他安装方式（brew、wget 等）
```

## WSL 与 Windows 互操作

### 文件路径访问

```bash
# 在 WSL 中访问 Windows 文件
cd /mnt/e/obsidian-notes

# 在 Windows 中访问 WSL 文件
\\wsl$\Ubuntu\home\username
```

### 环境变量传递

WSL 和 Windows 的环境变量是隔离的，如需共享需要手动配置。

## 实用工具安装清单

基于 [[SpecStory]] 和 [[Claude Code]] 的使用需求，以下是常用工具清单：

```bash
# 基础工具
sudo apt update
sudo apt install -y unzip wget curl git

# Node.js（如需使用 npm）
curl -fsSL https://deb.nodesource.com/setup_lts.x | sudo -E bash -
sudo apt install -y nodejs

# Homebrew（可选，但推荐）
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```

## 总结

WSL/Linux 环境配置的核心要点：

1. **身份管理**：区分 root 和普通用户使用场景
2. **包管理器**：Linux 默认无 Homebrew，需手动安装
3. **问题排查**：90% 的 command not found 是工具未安装
4. **工具链安装**：按需安装 unzip、wget 等基础工具

良好的环境配置是高效开发的基础，花时间配置好 WSL 环境，后续使用各种 AI 开发工具都会事半功倍。

---

**相关文章**：[[SpecStory 工具全解]] | [[Claude Code 基础使用指南]] | [[CLAUDE.md 规范管理]]
