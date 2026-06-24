# NVM、Node、npm、pnpm 与 Corepack 关系梳理

本文记录在 Debian 13 中使用 NVM 管理 Node.js，以及安装 pnpm 时容易混淆的几个点。

## 1. 基本原则

如果已经使用 NVM 安装 Node.js，那么 Node 生态相关工具应尽量跟随 NVM 环境，不要安装到系统 Node/npm 中。

推荐规则：

```text
系统 Node/npm：留给系统或系统包管理器，不主动使用
NVM Node/npm：用于个人开发环境
pnpm/yarn/vite/typescript 等工具：跟随当前 NVM Node 环境
```

不要使用：

```bash
sudo npm install -g pnpm
```

这样会把工具安装到系统目录，容易造成权限和版本混乱。

## 2. 检查当前 Node/npm 来源

打开终端后先检查：

```bash
which node
which npm
node -v
npm -v
```

如果路径类似：

```text
/home/ai/.nvm/versions/node/v24.17.0/bin/node
/home/ai/.nvm/versions/node/v24.17.0/bin/npm
```

说明当前使用的是 NVM 管理的 Node.js。

如果路径是 `/usr/bin/node` 或 `/usr/bin/npm`，则说明当前使用的是系统 Node/npm。

## 3. pnpm 推荐用 Corepack 安装

Node.js 自带 Corepack，可用于管理 pnpm、Yarn 等包管理器。

推荐安装方式：

```bash
nvm use 24
corepack enable
corepack prepare pnpm@latest --activate
pnpm --version
```

注意参数是 `--activate`，不是 `--active`。

错误写法：

```bash
corepack prepare pnpm --active
```

正确写法：

```bash
corepack prepare pnpm --activate
```

安装后检查：

```bash
which pnpm
pnpm --version
```

如果路径类似：

```text
/home/ai/.nvm/versions/node/v24.17.0/bin/pnpm
```

说明 pnpm 已经安装在 NVM 当前 Node 环境中。

## 4. NVM、npm、pnpm 的关系

NVM 负责安装和切换 Node.js 版本。

每个 Node.js 版本都会带自己的 npm：

```text
Node 24 -> 自己的 npm
Node 25 -> 自己的 npm
```

pnpm 属于 Node 生态工具，应安装在当前 NVM Node 环境下。

示例：

```text
/home/ai/.nvm/versions/node/v24.17.0/bin/node
/home/ai/.nvm/versions/node/v24.17.0/bin/npm
/home/ai/.nvm/versions/node/v24.17.0/bin/pnpm
```

这种状态是正确的。

## 5. 是否需要升级 Node

不需要频繁升级。

一般建议：

```text
学习、日常开发：优先使用 LTS 版本
老项目、工作项目：跟随项目要求
新项目、实验项目：可以尝试较新的版本
```

Node 偶数大版本通常更适合长期使用，奇数大版本通常更偏过渡和尝鲜。

如果当前 Node 24 能正常使用，没有必要急着升级到 Node 25。

## 6. 将来升级到 Node 25

安装 Node 25：

```bash
nvm install 25
nvm use 25
```

设置默认版本：

```bash
nvm alias default 25
```

检查：

```bash
node -v
which node
```

## 7. 迁移旧版本中的全局工具

如果要从 Node 24 迁移到 Node 25，可以在安装时迁移全局 npm 包：

```bash
nvm install 25 --reinstall-packages-from=24
```

如果 Node 25 已经安装完成，也可以执行：

```bash
nvm use 25
nvm reinstall-packages 24
```

对于 pnpm，切换到新 Node 版本后建议重新启用 Corepack：

```bash
corepack enable
corepack prepare pnpm@latest --activate
pnpm --version
```

## 8. 项目依赖是否需要迁移

项目依赖不需要从旧 Node 版本手动迁移。

项目依赖由项目目录中的文件决定：

```text
package.json
pnpm-lock.yaml
```

切换 Node 版本后，在项目目录重新安装即可：

```bash
pnpm install
```

## 9. 常用检查命令

```bash
which node
which npm
which pnpm
node -v
npm -v
pnpm --version
nvm current
nvm ls
```

判断是否正常的核心标准：

```text
node、npm、pnpm 都位于 /home/ai/.nvm/versions/node/某个版本/bin/ 下
```

## 10. 推荐工作流

安装或切换 Node：

```bash
nvm install 24
nvm use 24
nvm alias default 24
```

启用 pnpm：

```bash
corepack enable
corepack prepare pnpm@latest --activate
```

创建或进入项目：

```bash
pnpm install
pnpm dev
```

日常排查：

```bash
which node
which npm
which pnpm
```

只要这些命令显示的路径都在 NVM 目录下，环境就是清晰的。

