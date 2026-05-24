# 独立 Prettier 批量格式化工具

一个基于独立安装的 Prettier 的批量代码格式化工具，支持自定义配置、忽略指定目录，适用于格式化多个项目或大型代码库，而不会“污染”原项目。

---

## 🚀 功能特性

- 💡 使用独立的 Prettier 安装环境，不依赖项目内的 `node_modules`，也不使用全局安装版本；
- 🛠️ 支持通过 `.prettierrc.js` 自定义格式化规则（如 `tabWidth: 4` 等）；
- 📂 支持批量格式化一个或多个目录；
- 🙈 自动忽略 `node_modules`、`dist` 等不需要格式化的目录；
- 🧰 封装为简单易用的 Shell 脚本，一键运行。

---

## 🛠️ 创建步骤

以下是在你的本地环境中搭建此工具的完整流程：

### 1️⃣ 创建工具目录

在你的用户目录（或其他你喜欢的位置）创建一个用于存放 Prettier 及其配置的目录，例如：

```bash
mkdir -p ~/tools/prettier
cd ~/tools/prettier
```

### 2️⃣ 初始化 npm 项目并安装 Prettier
在工具目录下初始化一个 npm 项目，并安装指定版本的 Prettier：

npm init -y
npm install prettier@2.8.1  # 你可以根据需要更改版本号，如 3.x
这里使用 2.8.1 作为示例，你也可以安装最新的稳定版。

### 3️⃣ 创建 Prettier 配置文件

在工具目录下创建 .prettierrc.js 文件，用于定义你的代码风格规则，例如：

```js
// ~/tools/prettier/.prettierrc.js
module.exports = {
  tabWidth: 4,             // 缩进宽度为 4 个空格
  useTabs: false,          // 不使用 Tab 字符
  semi: true,              // 语句结尾加分号
  singleQuote: true,       // 使用单引号
  trailingComma: 'es5',    // 尾随逗号规则
  printWidth: 100,         // 每行最大字符数
};你可以根据团队或个人喜好调整这些配置项。
```



### 4️⃣ 创建忽略规则文件

为了让 Prettier 自动跳过不需要格式化的目录（如 node_modules、dist 等），创建 .prettierignore 文件：

```js
# ~/tools/prettier/.prettierignore

node_modules
dist
build
coverage
*.min.js
*.min.css
```



### 5️⃣ 创建运行脚本 run.sh

在工具目录下创建 run.sh 文件，内容如下（​​注意：此脚本已支持动态获取脚本所在目录，无需硬编码路径​​）：

```bash
#!/bin/bash

# ------------------------------
# 独立 Prettier 批量格式化工具
# 使用 find 命令递归查找文件，避免依赖 shell 的 globstar
# ------------------------------

# 获取当前脚本所在目录的绝对路径
SCRIPT_DIR="$(cd "$(dirname "$0")" && pwd)"

# 设置 Prettier 相关路径（基于脚本所在目录）
PRETTIER_DIR="$SCRIPT_DIR"
PRETTIER_BIN="$PRETTIER_DIR/node_modules/.bin/prettier"
PRETTIER_CONFIG="$PRETTIER_DIR/.prettierrc.js"

# 检查 Prettier 是否存在
if [ ! -f "$PRETTIER_BIN" ]; then
  echo "❌ 错误：未找到 Prettier 可执行文件，请确保已在当前目录（$PRETTIER_DIR）正确安装 Prettier。路径: $PRETTIER_BIN"
  exit 1
fi

# 检查是否传入了目录参数
if [ $# -eq 0 ]; then
  echo "❌ 错误：请传入至少一个需要格式化的目录作为参数。"
  echo "用法: $0 /path/to/dir1 /path/to/dir2 ..."
  exit 1
fi

# 遍历所有传入的目录参数
for dir in "$@"; do
  if [ ! -d "$dir" ]; then
    echo "⚠️  跳过无效目录: $dir"
    continue
  fi

  echo "🔧 开始格式化目录: $dir"

  # 执行 Prettier 格式化，使用自定义配置，并跳过忽略目录
  # 注意: ⚠️⚠️⚠️⚠️⚠️⚠️⚠️⚠️⚠️  这里使用 "$dir"/**/*.{js,jsx,ts,tsx,html,css,json,md} 只会处理第一个层级,不会处理深层的的文件,所以改成下面的 find命令!!!
    # "$PRETTIER_BIN" \
    # --config "$PRETTIER_CONFIG" \
    # --write \
    # --ignore-path "$PRETTIER_DIR/.prettierignore" \
    # "$dir"/**/*.{js,jsx,ts,tsx,html,css,json,md}


  # ------------------------------
  # 使用 find 命令递归查找所有需要格式化的文件 
  # 注意: ⚠️⚠️⚠️⚠️⚠️⚠️⚠️⚠️⚠️  "$dir"/**/*.{js,jsx,ts,tsx,html,css,json,md} 只会处理第一个层级,不会处理深层的的文件,所以改成下面的 find命令!!! fuck!
  # ------------------------------
  find "$dir" -type f \( \
    -name "*.js" -o \
    -name "*.jsx" -o \
    -name "*.ts" -o \
    -name "*.tsx" -o \
    -name "*.css" -o \
    -name "*.json" -o \
    -name "*.md" \
  \) -print0 | while IFS= read -r -d '' file; do
    # 对每个找到的文件运行 Prettier 格式化
    "$PRETTIER_BIN" \
      --config "$PRETTIER_CONFIG" \
      --write \
      "$file"
  done

  echo "✅ 完成格式化: $dir"
done

echo "🎉 所有指定目录格式化完成！"
```



赋予脚本执行权限：

```bash
chmod +x run.sh
```



## ▶️ 使用说明

确保你已经完成了前面的「创建步骤」，然后按如下方式运行工具：

### 1️⃣ 进入脚本所在目录

cd ~/tools/prettier
如果你把脚本放在了其他位置，记得进入对应的目录。

### 2️⃣ 运行脚本并传入需要格式化的目标目录

```bash
./run.sh /path/to/your/project1 /path/to/your/project2示例（根据你之前的路径）
```


脚本会自动：

- 使用同目录下的 Prettier 和 .prettierrc.js 配置；
- 格式化目标目录下的所有支持文件（如 .js、.html 等）；
- 跳过 node_modules、dist 等被忽略的目录。

## ⚠️ 注意事项

- 确保目标目录中包含需要格式化的代码文件（如 .js、.html 等）；
- 如果格式化未生效，请检查：
    - 传入的目录路径是否正确；
    - 文件扩展名是否匹配脚本中的匹配模式（如 *.{js,jsx,html,...}）；
    - 是否被 .prettierignore 文件排除；
    - 如果你升级了 Prettier 版本，请重新运行 npm install prettier@新版本；
    - 该工具不会修改或删除任何文件，但建议在运行前备份重要代码（或使用 Git 管理）。

