# actions-gh-pages

## 内容来源

- [actions-gh-pages](https://github.com/peaceiris/actions-gh-pages)
-  [GitHub凭证区别](https://chat.deepseek.com/share/37lldwwwu2m7wobum4)[（Deepseek）](https://chat.deepseek.com/a/chat/s/351bd0f3-9c56-42f0-ac38-91bf90c0e184)
-  [GitHub令牌区别](https://chat.deepseek.com/share/8m05dxz9pqrkl0wo8s)[（Deepseek）](https://chat.deepseek.com/a/chat/s/26baece4-d43e-48eb-81d4-d6de0ece8f04)

actions-gh-pages 这个项目非常不错，利用 `Github Actions` 部署静态页面。

> This is a GitHub Action to deploy your static files to GitHub Pages. This deploy action can be combined simply and freely with Static Site Generators. (Hugo, MkDocs, Gatsby, mdBook, Next, Nuxt, and so on.)

这是一个用于将静态文件部署到 `GitHub Pages` 的 `GitHub Action`。此部署操作可与静态网站生成器简单灵活地结合使用（Hugo、MkDocs、Gatsby、mdBook、Next、Nuxt等）。

```yaml
- name: Deploy
  uses: peaceiris/actions-gh-pages@v4
  with:
    github_token: ${{ secrets.GITHUB_TOKEN }}
    publish_dir: ./public
```

值得注意的是，`github_token` 可以选择三种的类型，即该项目支持的三种。

> Three tokens are supported.

|Token|Private repo|Public repo|Protocol|Setup|
|:---|:---:|:---:|:---:|:---|
|github_token|✅|✅|HTTPS|Unnecessary|
|deploy_key|✅|✅|SSH|Necessary|
|personal_token|✅|✅|HTTPS|Necessary|

> Notes: Actually, the GITHUB_TOKEN works for deploying to GitHub Pages but it has still some limitations. For the first deployment, we need to select the gh-pages branch or another branch on the repository settings tab. See First Deployment with GITHUB_TOKEN.

实际上，`GITHUB_TOKEN` 可用于部署到 `GitHub Pages`，但仍存在一些限制。对于首次部署，我们需要在仓库设置的选项卡中选择 `gh-pages` 分支或其他分支。请参阅“使用 GITHUB_TOKEN 进行首次部署”。

## 区别

### GITHUB_TOKEN
GitHub Actions 工作流的"内置身份"，由系统自动创建和管理，权限仅限于当前仓库。

### DEPLOY_KEY
一种 SSH 密钥，权限被锁定在单个仓库，常用于从服务器拉取或推送代码到此仓库。

### PERSONAL_TOKEN
关联到你个人账户的令牌，拥有你在 GitHub 上的全部或部分权限，是最强大但也最需谨慎管理的一种。

## 对比表格

|特性|GITHUB_TOKEN|DEPLOY_KEY|PERSONAL_TOKEN|
|---|---|---|---|
|关联对象|当前工作流运行|单个仓库|你的个人账户|
|权限范围|仅限于当前仓库|仅限于绑定的单个仓库（可读或读写）|可访问你账户有权访问的所有仓库及资源|
|身份验证方式|HTTPS (Token)|SSH (公钥/私钥)|HTTPS (Token)|
|有效期|每次工作流作业开始时创建，作业完成后失效|长期有效，直到手动删除|长期有效，直到手动撤销或过期|
|典型使用场景|GitHub Actions 中拉取当前仓库代码、进行常规操作|在 Actions 中拉取或推送代码到特定仓库（尤其是私有仓库）|需要在 Actions 中访问其他仓库、调用 GitHub API|

## 如何选择
- 优先使用 `GITHUB_TOKEN`：如果工作流只需要访问当前仓库的代码，这是最安全、最便捷的选择，无需任何手动配置。

- 需要访问特定私有仓库时，使用 `DEPLOY_KEY`：当工作流需要从另一个特定的私有仓库拉取代码（例如一个公共工具库），但又不想把个人账户的所有权限都暴露出去时，`DEPLOY_KEY` 是绝佳选择。它的权限被"锁死"在目标仓库上。

- 万不得已时，使用 `PERSONAL_TOKEN`：当 `GITHUB_TOKEN` 权限不足（比如需要触发新的工作流），或者操作涉及到当前仓库之外的资源时，才考虑使用个人访问令牌。务必注意：由于它关联的是你的个人账户，权限非常宽泛，潜在风险也最大，因此要谨慎使用，并遵循最小权限原则（只授予必要的权限）

## 如何设置
- `GITHUB_TOKEN` 并不需要你手动去“设置”或“创建”。它是在 `GitHub Actions` 工作流运行时，由 GitHub 系统自动生成的一个临时凭证。你只需要在工作流的 YAML 文件中通过特定的语法来使用它。

- `DEPLOY_KEY` 的设置需要分两步走：生成密钥对，然后分别把公钥和私钥添加到正确的地方。它本质上是一个 SSH 密钥对，专门用于授权访问单个 GitHub 仓库。

- 设置 `PERSONAL_TOKEN`（个人访问令牌，简称 PAT）主要分几步：进入创建页面、配置权限、生成并安全保存。你可以把它当作 GitHub 密码的替代品，用于命令行或 API 操作。