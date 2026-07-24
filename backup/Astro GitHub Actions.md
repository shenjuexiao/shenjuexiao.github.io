# Astro 安装及 GitHub Actions 自动部署

## 内容来源

-   [安装 Astro](https://docs.astro.build/zh-cn/install-and-setup/)
- [npx](https://docs.npmjs.com/cli/v11/commands/npx)

## 通过 CLI 向导安装

[docs.astro.build/zh-cn/install-and-setup/](https://docs.astro.build/zh-cn/install-and-setup/)

我选择的是 `Astro Use blog template`，也可以选择其他的选项，使用模板能快速建立页面。

![Astro Use blog template](https://cdn.jsdelivr.net/gh/shenjuexiao/shenjuexiao@main/Github_png/Astro_Use_blog_template.png)

```bash
# 使用 npm 创建一个新项目
npm create astro@latest
```

或

```bash
# 使用 pnpm 创建一个新项目
pnpm create astro@latest
```

或

```bash
# 使用 yarn 创建一个新项目
yarn create astro
```

![npm create astro@latest](https://cdn.jsdelivr.net/gh/shenjuexiao/shenjuexiao@main/Github_png/Astro_npm_create_astro@latest.png)

## 目录和文件

[docs.astro.build/zh-cn/basics/project-structure/](https://docs.astro.build/zh-cn/basics/project-structure/)

> Astro 采用一套约定俗成的文件夹布局来管理项目。每个 Astro 项目的根目录下都应该包括以下目录和文件：

- src/* - 你的项目源代码（组件、页面、样式、图片等）。
- public/* - 你的非代码、未处理的资源（字体、图标等）。
- package.json - 项目清单。
- astro.config.mjs - Astro 配置文件（推荐）。
- tsconfig.json - TypeScript 配置文件（推荐）。

## 启动 Astro 开发服务器

[docs.astro.build/zh-cn/develop-and-build/](https://docs.astro.build/zh-cn/develop-and-build/)

> Astro 自带了一个内置的开发服务器，它包含了项目开发所需的一切。astro dev CLI 命令将启动本地开发服务器，让你第一次看到你的新网站。

> 每个起始模板都预配置了一个脚本，该脚本将为你运行 astro dev。在进入你的项目目录后，使用你喜欢的包管理器运行此命令并启动 Astro 开发服务器。

```bash
npm run dev
```


或

```bash
pnpm run dev
```


或

```bash
yarn run dev
```

## 部署你的 Astro 站点至 GitHub Pages

[docs.astro.build/zh-cn/guides/deploy/github/](https://docs.astro.build/zh-cn/guides/deploy/github/)

```yaml
name: Deploy to GitHub Pages

on:
  # 每次推送到 `main` 分支时触发这个“工作流程”
  # 如果你使用了别的分支名，请按需将 `main` 替换成你的分支名
  push:
    branches: [ main ]
  # 允许你在 GitHub 上的 Actions 标签中手动触发此“工作流程”
  workflow_dispatch:

# 允许 job 克隆 repo 并创建一个 page deployment

```yaml
permissions:
  contents: read
  pages: write
  id-token: write

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout your repository using git
        uses: actions/checkout@v5
      - name: Install, build, and upload your site
        uses: withastro/action@v5
        # with:
          # path: . # 存储库中 Astro 项目的根位置。（可选）
          # node-version: 20 # 用于构建站点的特定 Node.js 版本，默认为 20。（可选）
          # package-manager: pnpm@latest # 应使用哪个 Node.js 包管理器来安装依赖项和构建站点。会根据存储库中的 lockfile 自动检测。（可选）
          # build-cmd: pnpm run build # 用于构建你的网站的命令。默认运行软件包的构建脚本或任务。（可选）
        # env:
          # PUBLIC_POKEAPI: 'https://pokeapi.co/api/v2' # 对变量值使用单引号。（可选）

  deploy:
    needs: build
    runs-on: ubuntu-latest
    environment:
      name: github-pages
      url: ${{ steps.deployment.outputs.page_url }}
    steps:
      - name: Deploy to GitHub Pages
        id: deployment
        uses: actions/deploy-pages@v4
```
 
 
##  Error: Failed to create deployment (status: 404)

![Astro Deploy to GitHub Pages](https://cdn.jsdelivr.net/gh/shenjuexiao/shenjuexiao@main/Github_png/Astro_Deploy_to_GitHub_Pages.png)

这个错误表明 `GitHub Pages` 部署失败，主要原因是 `GitHub Pages` 没有在仓库设置中启用。错误信息明确指出：

Ensure GitHub Pages has been enabled: https://github.com/shenjuexiao/astro/settings/pages