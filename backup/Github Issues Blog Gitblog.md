# Gitblog

用过 Jekyll、Hexo，捣鼓了一阵子，都没能长期使用持续写作，现在转头使用 GIthub Issue 搭建的博客系统。后者有两个优点，一来省去了生成、部署的操作；二来解决了图床的需求。只要仓库存在，Issues 可以一直保留，改天也可以换个相似的博客系统。

## 内容来源

- [Gitblog](https://github.com/gaishanke/gaishanke.github.io/issues/1)

## 项目地址和安装指南
[yihong0618/gitblog](https://github.com/yihong0618/gitblog)  
[这个博客开源了](https://github.com/yihong0618/gitblog/issues/177)

## 安装方法
### 1. 克隆仓库
```bash
git clone git@github.com:yihong0618/gitblog.git
```

### 2. 生成tokens
个人首页，`Settings` -> `Developer Settings` -> `Personal access tokens` -> `Tokens (classic)` -> `Generate New Tokens` ->  `Generate new yokens (classic)`

或直达链接：
[settings/tokens](https://github.com/settings/tokens)

<img src="https://cdn.jsdelivr.net/gh/shenjuexiao/shenjuexiao@main/Github_png/Gitblog_Settings_Developer-Settings_Personal-access-tokens.png" />

### 3. 仓库 actions
仓库首页，`Settings` -> `Security and quality` -> `Secrets and variables` -> `Actions` -> `New repository secret`

<img src="https://cdn.jsdelivr.net/gh/shenjuexiao/shenjuexiao@main/Gitblog_Settings_Security-and-quality_Secrets-and-variables.png" />

直接使用 `G_T`，就不用修改文件： `.github\workflows\generate_readme.yml` 中的 `${{ secrets.G_T }}` 字段。

```yaml
- name: Generate new md
  run: |
    python main.py ${{ secrets.G_T }} ${{ github.repository }} --issue_number '${{ github.event.issue.number }}'
```

### 4.  ~~修改`generate_readme.yml`~~
`.github/workflows/generate_readme.yml`
新版不需要。

## 其他问题
原作者后续更新了 `*.yml` 文件，有些参数不需要修改了。
后续更新补充。