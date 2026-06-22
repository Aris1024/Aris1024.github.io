# Systemd 部署 Node.js 服务：解决 NVM 环境变量不生效的问题

在使用 Systemd 将 Node.js 应用部署为后台守护进程时，经常会遇到一个关于 Node 版本的环境变量陷阱。本文记录了该问题的排查过程与最终解决方案，备查。

## 1. 问题背景与现象

在服务器中使用了 `nvm` 来管理 Node.js 版本。当前终端激活了 `v24.17.0`，而系统默认版本（System）为 `v22.22.0`。

为了让某个 Web 服务（如 `hermes-web-ui`）在后台运行，编写了如下的 Systemd 服务配置文件：

```ini
[Unit]
Description=Hermes Web UI Service
After=network.target

[Service]
User=ai
WorkingDirectory=/home/ai/hermes_web_ui
Type=forking

# 已经显式指定了 nvm v24 版本的绝对路径
ExecStart=/home/ai/.nvm/versions/node/v24.17.0/bin/hermes-web-ui start
ExecStop=/home/ai/.nvm/versions/node/v24.17.0/bin/hermes-web-ui stop

Restart=always
RestartSec=3

[Install]
WantedBy=multi-user.target
```

**现象**：服务成功启动，但在查看应用状态（或 Web UI 面板）时，系统提示当前运行的 Node.js 版本为 `v22.22.0`（需要升级），这意味着程序并未以 `ExecStart` 中指定的 `v24.17.0` 版本运行。

## 2. 根因分析

问题的核心在于 **Systemd 的运行环境与可执行文件的 Shebang 机制产生了冲突**。

1. **Systemd 的非交互式环境**：
   Systemd 在启动服务时，处于非交互式（Non-interactive）环境。它**不会**去加载用户的 `~/.bashrc`、`~/.profile` 等配置文件。因此，`nvm` 注入到终端的那些环境变量（尤其是 `PATH`）在 Systemd 进程中是完全不存在的。Systemd 仅维持一个极其简陋的默认 `PATH`。
2. **Shebang 解析机制**：
   虽然我们在 `ExecStart` 中写死了可执行文件的绝对路径（`/home/ai/.nvm/.../bin/hermes-web-ui`），但这类基于 Node.js 的 CLI 工具，其文件头部通常会包含一行 Shebang 声明，例如：
   ```javascript
   #!/usr/bin/env node
   ```
   这一行的作用是告诉系统：“请去当前的 `PATH` 环境变量中寻找 `node` 命令来执行这段脚本”。
3. **最终结果**：
   由于 Systemd 没有加载 `nvm` 的环境变量，当解析到 `#!/usr/bin/env node` 时，系统只能沿着默认的 `PATH` 去找，最终找到了系统全局安装的默认版本 `/usr/bin/node`（即 v22.22.0）。

## 3. 解决方案

要解决这个问题，必须在 Systemd 的 `[Service]` 配置块中，显式覆盖并指定 `PATH` 环境变量，将目标版本的 Node.js 的 `bin` 目录置于系统默认路径之前。

修改 `xxx.service` 文件，新增一行 `Environment` 配置：

```ini
[Unit]
Description=Hermes Web UI Service
After=network.target

[Service]
User=ai
WorkingDirectory=/home/ai/hermes_web_ui

# 【关键配置】显式指定环境变量 PATH，将 nvm 的目标版本 bin 目录放在最前面
Environment="PATH=/home/ai/.nvm/versions/node/v24.17.0/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin"

Type=forking
ExecStart=/home/ai/.nvm/versions/node/v24.17.0/bin/hermes-web-ui start
ExecStop=/home/ai/.nvm/versions/node/v24.17.0/bin/hermes-web-ui stop

Restart=always
RestartSec=3

[Install]
WantedBy=multi-user.target
```

*注：你可以通过在当前正确的终端下运行 `echo $PATH` 来获取完整的路径字符串，或者至少确保目标 Node 版本的路径处于 `/usr/bin` 之前。*

## 4. 使配置生效

修改并保存 `.service` 文件后，依次执行以下命令让 Systemd 重新加载配置并重启服务：

```bash
# 1. 如果文件不在系统目录下，先将其拷贝过去（根据实际情况调整）
sudo cp ~/hermes_web_ui/hermes-web-ui.service /etc/systemd/system/

# 2. 重新加载 Systemd 配置
sudo systemctl daemon-reload

# 3. 重启应用服务
sudo systemctl restart hermes-web-ui

# 4. 检查服务状态
sudo systemctl status hermes-web-ui
```

此时再次验证，应用已经按预期使用 `v24.17.0` 版本的 Node.js 运行。

## 5. 总结

在 Systemd 中部署依赖特定环境（如 nvm, rvm, pyenv 管理的 Node / Ruby / Python 等）的服务时：
**仅在 ExecStart 中使用可执行文件的绝对路径是不够的。** 只要该脚本使用了 `/usr/bin/env` 进行解释器寻址，就必须通过 `Environment="PATH=..."` 为其补全正确的环境变量上下文。