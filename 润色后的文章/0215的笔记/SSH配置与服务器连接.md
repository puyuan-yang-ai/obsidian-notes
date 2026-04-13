# SSH配置与服务器连接

SSH（Secure Shell）是远程服务器管理的核心工具。掌握其配置细节，能大幅提升工作效率和安全性。

## 公钥认证机制

Linux服务器上存放SSH公钥的标准位置是 `~/.ssh/authorized_keys`。当执行 `ssh root@服务器IP` 时，服务器会依次执行三个验证步骤：首先查找 `/root/.ssh/authorized_keys` 文件，然后将本地私钥对应的公钥指纹与文件中每一行进行匹配，匹配成功后才允许登录。

authorized_keys 文件采用简单的格式规则：一行代表一个公钥，可以放置多个公钥以支持多人登录。典型的文件内容如下：

```
ssh-ed25519 AAAAC3Nz... your-laptop
ssh-ed25519 AAAAC3Nz... office-pc
```

> [!warning]
> **文件权限至关重要**
> 必须执行 `chmod 700 ~/.ssh` 和 `chmod 600 ~/.ssh/authorized_keys`。权限设置不正确会导致 `Permission denied (publickey)` 错误，这是很多人配置时的常见卡点。

## 阿里云服务器配置流程

拿到阿里云服务器后，配置SSH免密登录的标准操作流程（SOP）是：

1. 通过 Workbench 登录 admin 账户
2. 编辑公钥文件：`nano ~/.ssh/authorized_keys`
3. 将本地电脑的公钥内容（如 `C:\Users\yangp\.ssh\id_rsa.pub`）粘贴进去
4. 保存退出：Ctrl+O → Enter → Ctrl+X
5. 设置权限：`chmod 700 ~/.ssh` 和 `chmod 600 ~/.ssh/authorized_keys`

需要注意的是，以不同用户身份登录时，服务器检测的 authorized_keys 路径也不同。例如 `ssh admin@服务器IP` 只会检测 `/home/admin/.ssh/authorized_keys`，不会检测 `/root/.ssh/authorized_keys`。

> [!info]
> **nano 保存退出记忆法**
> - **O** = Output（写出保存）→ Ctrl+O，然后 Enter 确认文件名
> - **X** = Exit（退出）→ Ctrl+X
> - 中间的 Enter 只是确认文件名，不改则直接回车

## SSH 别名与配置文件

通过配置 `~/.ssh/config` 文件，可以为常用的服务器连接设置别名。例如设置 `aliyun-sg` 别名后，只需执行 `ssh aliyun-sg` 即可连接。系统会自动读取配置文件，解析用户名、IP等参数并展开成完整的SSH命令。

SSH config 文件还支持端口转发功能。配置 `LocalForward 18789 127.0.0.1:18789` 后，访问本地 18789 端口的请求会通过SSH隧道转发到服务器的对应端口。

```
浏览器 localhost:18789
       ↓ SSH 隧道
       ↓
服务器 localhost:18789（如 OpenClaw gateway）
```

> [!danger]
> **切勿直接开放端口**
> 不要在阿里云/VPS防火墙里直接开放 18789 等端口对公网访问，这非常危险会被黑客扫描。请使用SSH隧道方式访问，更安全更稳定。

## 总结

SSH配置的核心在于理解公钥认证机制和文件权限。正确配置 authorized_keys、设置适当的权限、善用SSH别名和端口转发，能让服务器管理事半功倍。

[[OpenClaw安装与基础配置]] [[Shell类型与环境变量机制]]
