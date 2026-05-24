# vscode 代码片段(记录)

---

- ```json
    "export default ": {
    		"scope": "javascript,typescript,javascriptreact,typescriptreact",
    		"prefix": "ee export default ",
    		"body": [
                "export default function $1() {",
                "\t $2",
                "}"
    		],
    		"description": "export default "
    	},
    ```

- **\t** 代表制表符,缩进tab

- `$1` 是**光标**跳转位置,按tab跳转下一个 `$2`

    ```json
    "JavaScript-自定义分割线-******************": {
    		"scope": "javascript,typescript,javascriptreact,typescriptreact",
    		"prefix": "ffff",
    		"body": [
                "/******************** ${1:分割线} ********************/",
    		],
    		"description": "自定义分割线"
    	},
    ```

    

    ```
    "JavaScript-React-setState-自动大写": {
    		"scope": "javascript,typescript,javascriptreact,typescriptreact",
    		"prefix": "uuu",
    		"body": [
                "const [${1:state}, set${1/(.*)/${1:/capitalize}/}] = useState($2);"
    		],
    		"description": "const [state, setState] = useState();"
    	},
    ```

- > 关键点解释：
    >    * `${1:state}`: 这是你的第一个占位符，默认值是 state。
    >    * `set${1/(.*)/${1:/capitalize}/}`: 这是核心逻辑。
    >        * ${1/.../} 表示引用第一个占位符的内容进行变换。
    >        * /(.*)/ 是正则表达式，匹配你输入的整个变量名。
    >        * ${1:/capitalize}
    >          是变换规则，表示将匹配到的第一个分组（即你输入的变量名）首字母大写。
    >        * 前面的 set 是固定的前缀。
    >
    >   实际效果：
    >    1. 输入 uuu 并回车，默认生成 const [state, setState] = useState();。
    >    2. 当你开始打字，把 state 修改为 index 时，后面的内容会实时变成 setIndex。
    >    3. 当你把 state 修改为 name 时，后面会自动变成 setName。
    >
    >   温馨提示：修改完 snippets 配置文件后通常不需要重启 VS Code，直接在代码中尝试即可。

我非常理解你的感受！每次都要多按一次 `Esc` 确实会打断心流（Flow），而且手感很“顿”。你希望保留全局提示，但只在这个特定场景下让 `Tab` 键优先服务于“跳转”而不是“采纳建议”。

这个问题可以通过**修改快捷键优先级**来完美解决，而不需要关闭全局提示。

### 解决方案：调整 Tab 键的判断逻辑

我们需要告诉 VS Code：
> **“如果在 Snippet 模式下（代码片段填写中），按下 Tab 键，请无视那个弹出的提示框，直接跳转到下一个位置。”**

请按照以下步骤操作：

1.  在 VS Code 中按下 `Ctrl` + `Shift` + `P` (Mac: `Cmd` + `Shift` + `P`) 打开命令面板。
2.  输入 **`Open Keyboard Shortcuts (JSON)`** 并回车（注意是打开 JSON 文件的那个）。
3.  在打开的 `keybindings.json` 文件中（在中括号 `[]` 里面），粘贴以下配置：

```json
  {
    "key": "tab",
    "command": "-acceptSelectedSuggestion",
    "when": "suggestWidgetVisible && textInputFocus"
  },
  {
    "key": "tab",
    "command": "acceptSelectedSuggestion",
    "when": "suggestWidgetVisible && textInputFocus && !inSnippetMode"
  }
```

### 这段配置做了什么？

1.  **第一段 (`-acceptSelectedSuggestion`)**：
    前面的减号 `-` 表示**移除** VS Code 默认的 Tab 行为。默认情况下，只要提示框出现（`suggestWidgetVisible`），按 Tab 就会采纳建议。我们先把它干掉。

2.  **第二段 (`acceptSelectedSuggestion`)**：
    我们把这个行为**加回来**，但多加了一个条件：`&& !inSnippetMode`。
    意思是：只有在 **“不在代码片段模式中”** 时，Tab 才是采纳建议。

### 最终效果

*   **当你输入 `uuu` 触发代码片段时**：
    你输入 `showMore`，提示框依然会弹出（保留了你想要的提示），但是当你按下 **`Tab`** 时，VS Code 现在的逻辑是：“哦，你在 snippet 模式里，我不采纳建议”，于是它会执行优先级第二的操作——**跳转到括号里**。
    （不需要按 `Esc` 了！）

*   **当你正常写代码时**：
    你输入 `con` 弹出 `console`，按下 `Tab`，依然会正常自动补全为 `console`。一切如常，不会影响全局体验。

### 补充技巧
如果在极少数情况下，你在写 Snippet 时确实想采纳那个提示（比如引用了一个很长的变量名），你可以改用 **`Enter`** 键来采纳它，而保留 `Tab` 键专门用于跳转。
