# 同步推送 GitHub 和 Gitee

## 内容来源

-   [同步推送 GitHub 和 Gitee](https://chat.deepseek.com/share/jcldaa4ma7iv5xmfb6)[（Deepseek）](https://chat.deepseek.com/a/chat/s/fcd433c7-8267-4ed0-bd52-ad2a78311fd9)

将代码同时推送到 GitHub 和 Gitee，主要有两种方案。你可以根据自己的习惯，选择 Git 原生命令，或使用更自动化的工具。

## 方案一：使用 Git 原生命令（推荐）

这是最基础和灵活的方式，只需几条命令配置好一个“聚合”的远程仓库，之后一条`git push`命令就能同时推送到两个平台。

**核心原理**：给本地的`origin`（或任意名字）远程仓库设置**多个推送（push）URL**。这样，当执行`git push origin`时，Git 会依次向这些 URL 推送代码。

**配置步骤**：

1.  **进入项目目录，并查看当前远程仓库**：

```bash
cd your-project
git remote -v
```

2.  **添加两个平台的仓库地址作为推送目标**：

假设你的 Gitee 仓库地址是 `https://gitee.com/你的用户名/项目名.git`，GitHub 仓库地址是 `https://github.com/你的用户名/项目名.git`。

首先，将 Gitee 的地址设为 `origin` 的主要推送地址（如果已有`origin`，这一步会覆盖它）：
   
```bash
git remote set-url --push origin https://gitee.com/你的用户名/项目名.git
```

然后，将 GitHub 的地址添加为 `origin` 的**额外**推送地址：

```bash
git remote set-url --add --push origin https://github.com/你的用户名/项目名.git
```

实际上，如果不写 `--push` ，Git 也会默认只添加 `(push)` ，即以下的命令效果相当：

```bash
git remote set-url --add origin https://github.com/你的用户名/项目名.git
```

3.  **验证配置**：

再次运行 `git remote -v`，你会看到 `origin` 有了两个 `(push)` 地址，但 `(fetch)` 仍只指向 Gitee。

```bash
origin  https://gitee.com/你的用户名/项目名.git (fetch)
origin  https://gitee.com/你的用户名/项目名.git (push)
origin  https://github.com/你的用户名/项目名.git (push)
```



4.  **日常使用**：

 *   **推送**：执行 `git push origin main`（或你的主分支名），代码就会同时推送到 Gitee 和 GitHub。

*   **拉取**：`git pull` 默认只会从 `(fetch)` 指向的 Gitee 拉取更新，避免从多个源拉取可能导致的混乱。

> **注意**：这种方法的一个小缺点是，如果其中一个仓库推送失败（比如网络问题），整个推送操作会报错，需要你根据提示处理。

### 方案二：使用第三方工具自动化

如果你希望有更丰富的功能，比如一键提交并推送、全量同步所有分支和标签，可以考虑使用现成的工具。

| 工具名称 | 安装方式 | 特点 |
| :--- | :--- | :--- |
| **git-multi-sync-tool** | `npm install -g git-multi-sync-tool` | 基于文本配置文件管理多个仓库，支持智能推送策略、提交后自动推送等功能。 |
| **gtl** | 从 [GitHub Releases](https://github.com/crates-dev/gtl) 下载 | 提供一键初始化和推送功能，通过配置文件管理多个远程仓库。 |
| **mgit-push** | `npm install -g mgit-push` | 功能更简洁，专注于一键推送到 GitHub、Gitee 等多个平台 |

以 **git-multi-sync-tool** 为例，使用流程大致如下：
1.  安装：`npm install -g git-multi-sync-tool`
2.  在项目中初始化配置：`git-sync init`
3.  编辑生成的 `.git-remotes.txt` 文件，把 GitHub 和 Gitee 的仓库地址分别写入一行。
4.  执行 `git-sync setup` 设置远程仓库。
5.  之后就可以用 `git-sync commit "你的提交信息" -p` 来完成“提交并一键推送到所有仓库”了。

对于大部分开发者来说，方案一（Git 原生命令）已经足够好用，它不依赖额外工具，配置清晰明了。如果你需要管理多个不同项目，或者需要更复杂的同步策略，可以尝试方案二。