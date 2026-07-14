# The requested URL returned error: 403

# GitHub Actions push 403 权限拒绝解决

## 内容来源

- [豆包：GitHub Actions 403](https://www.doubao.com/chat/38434639772911618)
- [The requested URL returned error: 403](https://github.com/shenjuexiao/giscafer/issues/1)

<img src="https://cdn.jsdelivr.net/gh/shenjuexiao/shenjuexiao@main/Github_png/Giscafer_GitHub_Actions_push_403.png" />

## fatal: unable to access

```yaml
Run yarn sync-post
  yarn sync-post
  git add .
  git commit -m 'chore(ci): blog sync'
  git push
  shell: /usr/bin/bash -e {0}
  env:
    GH_TOKEN: ***
    GH_USER: ***
    GH_PROJECT_NAME: ***
yarn run v1.22.22
$ node ./scripts/sync-post.js
文章全部同步成功！ 0
Done in 0.33s.
ℹ No staged files found.
[main 7226b3a] chore(ci): blog sync
 3 files changed, 33 deletions(-)
 delete mode 100644 data/blog/post-1.mdx
 delete mode 100644 data/blog/post-2.mdx
 delete mode 100644 data/blog/post-3.mdx
remote: Permission to ***/***.git denied to github-actions[bot].
fatal: unable to access 'https://github.com/***/***/': The requested URL returned error: 403
Error: Process completed with exit code 128.
```

## 文件：`.github/workflows/sync-post.yml`
```yaml
name: Sync Post

# Controls when the workflow will run
on:
  # schedule:
  #   - cron: "30 1 * * *"
  # https://docs.github.com/cn/developers/webhooks-and-events/events/issue-event-types
  issues:
    types:
      - opened
      - closed
      - renamed
      - labeled
      - unlabeled
      - reopened
      - committed # 修改？
  workflow_dispatch:
env:
  GH_TOKEN: ${{ secrets.GH_TOKEN }}
  GH_USER: ${{ secrets.GH_USER }}
  GH_PROJECT_NAME: ${{ secrets.GH_PROJECT_NAME }}
jobs:
  Publish:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout 🛎️
        uses: actions/checkout@v2

      - name: Git config 🔧
        run: |
          git config --global user.name "shenjuexiao"
          git config --global user.email "shenjuexiao@qq.com"

      - name: Display runtime info ✨
        run: |
          echo '当前目录：'
          pwd

      - name: Install 🔧
        run: yarn install

      # - name: Build ⛏️
      #   run: yarn build

      - name: Update blog files ⛏️
        run: |
          yarn sync-post
          git add .
          git commit -m 'chore(ci): blog sync'
          git push
```

**github-actions[bot]** 没有仓库写入推送权限，触发 403 forbidden。

## 三种常见场景
1. 仅配置 GH_TOKEN，但仓库默认工作流令牌无写权限（默认只读）
2. 个人 Token 权限范围不足、过期，或未正确注入
3. Git remote 使用 https 但未自动替换授权头

## 方案一：开启仓库工作流写入权限（最简单，推荐）
步骤
仓库首页 ：`Settings` -> `Actions` -> `General`
下滑找到 `Workflow permissions`
勾选： `Read and write permissions`
（默认是 `
Read repository contents and packages permissions`）
保存设置，重新运行 Actions。
原理：内置 GITHUB_TOKEN 会获得推送、修改文件权限，无需额外个人 Token。

<img src="https://cdn.jsdelivr.net/gh/shenjuexiao/shenjuexiao@main/Github_png/Giscafer_Settings_Actions_General_Workflow-permissions.png" />

## 方案二：改用个人 PAT Token（适合需要跨仓库 / 更多权限）
1. 创建 GitHub 个人访问令牌 PAT
GitHub 头像，`Settings` -> `Developer settings` -> `Personal access tokens` -> `Tokens (classic)`
新建 Token，勾选权限：repo（全部仓库读写权限）。
复制生成的 Token（仅展示一次，丢失需重建）。
2. 仓库配置 Secret
仓库首页， `Settings` -> `Secrets and variables` -> `Actions` -> `New repository secret`
Name：GH_TOKEN。
Value：粘贴上面的 PAT。
3. 修改 `workflow` 注入授权（关键，你现有脚本缺授权）
你的脚本直接 `git push`，但 https 地址没带上 token，需要在 shell 里替换 remote url：
```yaml
steps:
  - uses: actions/checkout@v4
    # 必须加上 fetch-depth=0，否则无法正常commit push
    with:
      fetch-depth: 0

  - name: Sync blog post
    env:
      GH_TOKEN: ${{ secrets.GH_TOKEN }} # 如果用PAT填GH_TOKEN: ${{ secrets.GH_TOKEN }}
      GH_USER: ${{ secrets.GH_USER }}
      GH_PROJECT_NAME: ${{ secrets.GH_PROJECT_NAME }}
    run: |
      yarn sync-post
      # 核心：替换remote地址，带上token鉴权，解决403
      git remote set-url origin https://${GH_USER}:${GH_TOKEN}@github.com/${GH_USER}/${GH_PROJECT_NAME}.git
      git add .
      git commit -m 'chore(ci): blog sync'
      git push
    shell: /usr/bin/bash -e {0}
```

## 关键遗漏点（原配置2个问题）
1. `checkout` 未加 `fetch-depth:0`
默认只拉取最新一次提交，缺少完整 git 历史，`commit/push` 极易异常。
2. 未配置带 Token 的 `remote origin`
直接 git push 使用无授权的 https 地址，bot 被拦截 403。

## 补充排查点
1. 如果你用内置 `${{ secrets.GITHUB_TOKEN }}`
必须完成【方案一：开启读写权限】，否则就算改 remote 也推不上去。
2. PAT 失效处理
重新生成 Token，更新仓库 Secret 内的 `GH_TOKEN`。
3. 分支保护规则
若 main 分支开启 `Require pull request reviews before merging`，直接 push 会失败，两种解决：
- 临时放开分支保护允许 bot 直接推送
- 脚本改成自动创建 PR，而非直接 push

## 完整可直接复制的 job 模板

```yaml
jobs:
  sync-blog:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout code
        uses: actions/checkout@v4
        with:
          fetch-depth: 0

      - name: Setup Node & Yarn
        uses: actions/setup-node@v4
        with:
          node-version: 20
          cache: 'yarn'

      - name: Install dependencies
        run: yarn install

      - name: Sync post & push
        env:
          GH_TOKEN: ${{ secrets.GH_TOKEN }}
          GH_USER: ${{ secrets.GH_USER }}
          GH_PROJECT_NAME: ${{ secrets.GH_PROJECT_NAME }}
        run: |
          yarn sync-post
          git remote set-url origin https://${GH_USER}:${GH_TOKEN}@github.com/${GH_USER}/${GH_PROJECT_NAME}.git
          git config --global user.name "github-actions[bot]"
          git config --global user.email "github-actions[bot]@users.noreply.github.com"
          git add .
          git commit -m 'chore(ci): blog sync' || echo "无文件变更，跳过提交"
          git push
```

新增 `git config` 配置提交者用户名邮箱，避免无身份提交报错；
`|| echo` 兼容「无修改文件」场景，不会因为没有变更直接终止任务。