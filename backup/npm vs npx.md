# npm 与 npx 区别

## 内容来源

-   [npm 与 npx 区别](https://chat.deepseek.com/share/nib71ed8ghaqfuyb1y)[（Deepseek）](https://chat.deepseek.com/a/chat/s/1ea1f80c-e11d-4e1f-845f-8023549a044c)
- [npx](https://docs.npmjs.com/cli/v11/commands/npx)

## 简单说明

`npm` 和 `npx` 是 **Node.js** 生态里两个核心但职责不同的工具，可以用一句话来区分它们：

> **npm 是“包管理器”，负责安装、管理依赖；而 npx 是“包执行器”，负责运行命令，无需显式安装**。

## 表格比较

| 特性 | npm (Node Package Manager) | npx (Node Package Execute) |
| :--- | :--- | :--- |
| **核心职责** | 管理项目的依赖包，比如安装、卸载、更新、发布包 | 执行某个 Node.js 包提供的命令 |
| **是否需要安装** | **需要**。包会被下载并保存到 `node_modules` 或全局目录 | **通常不需要**。它会临时下载并执行，用完即走，不污染环境 |
| **典型场景** | 为项目安装核心依赖（如 `npm install react`）<br>运行 `package.json` 中定义的脚本（如 `npm run build`） | 运行一次性的脚手架工具（如 `npx create-react-app my-app`）<br>快速测试不同版本的包（如 `npx webpack@4.44.1 -v`） |
| **执行本地包** | 需要通过 `./node_modules/.bin/` 路径或 `npm run` 命令间接运行 | 可以直接运行，它会自动在 `node_modules/.bin/` 中查找命令 |

除了用法上的不同，`npx` 还有一个非常便利的特性：它可以临时下载并运行一个远程的 npm 包，这在测试新工具或运行一次性脚本时能帮你节省不少时间。

### 💡 如何选择？

一个很实用的场景是：创建 React 项目时，会先全局安装 `create-react-app`，再用它生成项目。有了 `npx`，可以直接 `npx create-react-app my-app`，不用全局安装，每次用的还都是最新版，非常方便。

**一句话总结就是**：
*   需要一个包长期使用（比如项目依赖）→ 用 **`npm install`**。
*   只是想临时跑一下某个命令或工具 → 用 **`npx`**。