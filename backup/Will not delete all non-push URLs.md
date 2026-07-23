# Git 远程 URL 修改

## 问题

```bash
PS D:\Git\shenjuexiao> git remote set-url --delete gitee git@gitee.com:shenjuexiao/shenjuexiao.git
fatal: Will not delete all non-push URLs
```

这个错误是因为你试图删除远程仓库 `gitee` 的**唯一** URL，Git 不允许这样做——每个远程仓库至少需要保留一个 URL。

## 解决方案

### 方案一：先添加新 URL，再删除旧的
```bash
# 先添加一个新的 URL
git remote set-url --add gitee git@gitee.com:new-repo.git

# 然后再删除旧的 URL
git remote set-url --delete gitee git@gitee.com:shenjuexiao/shenjuexiao.git
```

### 方案二：直接修改 URL（推荐）
```bash
# 直接替换整个 URL，而不是删除
git remote set-url gitee git@gitee.com:new-repo.git
```

### 方案三：删除整个远程仓库再重新添加
```bash
# 删除整个远程仓库
git remote remove gitee

# 重新添加
git remote add gitee git@gitee.com:new-repo.git
```

### 方案四：查看当前所有 URL，确认情况
```bash
# 查看 gitee 远程仓库的所有 URL
git remote get-url --all gitee
```

## 最常见的使用场景

如果你是**想修改远程仓库地址**，直接用方案二最简单：
```bash
git remote set-url gitee git@gitee.com:新的仓库地址.git
```

如果你是**想添加多个推送地址**（同时推送到多个仓库），用方案一。

## 重命名远程仓库

```bash
git remote rename github origin
```

这个命令会：
- 将远程仓库 `github` 重命名为 `origin`
- 保留所有原有的 URL 和配置
- 更新所有本地跟踪分支（如 `github/main` → `origin/main`）

### 验证是否成功

```bash
# 查看所有远程仓库
git remote -v
```

你应该能看到类似这样的输出：
```
origin  git@github.com:你的用户名/仓库名.git (fetch)
origin  git@github.com:你的用户名/仓库名.git (push)
```

### 注意事项

- 如果 `origin` 已经存在，这个命令会报错。你需要先删除或重命名现有的 `origin`
- 如果你的 `origin` 已经指向其他仓库，可以先确认一下：

```bash
# 查看当前 origin 指向哪里
git remote get-url origin
```

如果 `origin` 已经存在且不是你想用的，可以先删除它：

```bash
git remote remove origin
git remote rename github origin
```

执行后记得检查一下 `git remote -v` 确认结果正确。