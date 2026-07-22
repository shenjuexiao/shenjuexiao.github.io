# Gitee remote error hook declined

## 问题

-同时推送到 GitHub 和 Gitee 两个仓库：

### 基本设置

```bash
git remote set-url origin git@github.com:shenjuexiao/shenjuexiao.git
git remote set-url --add origin git@gitee.com:shenjuexiao/shenjuexiao.git
```

### 查看配置
```bash
git remote -v
origin  git@github.com:shenjuexiao/shenjuexiao.git (fetch)
origin  git@github.com:shenjuexiao/shenjuexiao.git (push)
origin  git@gitee.com:shenjuexiao/shenjuexiao.git (push)
```

### 推送文件
```bash
git add .
git commit -m "20260722"
git push origin main
origin  git@github.com:shenjuexiao/shenjuexiao.git (fetch)
origin  git@github.com:shenjuexiao/shenjuexiao.git (push)
origin  git@gitee.com:shenjuexiao/shenjuexiao.git (push)
```

### 返回结果
```bash
Everything up-to-date
Enumerating objects: 68, done.
Counting objects: 100% (68/68), done.
Delta compression using up to 16 threads
Compressing objects: 100% (67/67), done.
Writing objects: 100% (68/68), 5.68 MiB | 5.83 MiB/s, done.
Total 68 (delta 21), reused 0 (delta 0)
remote: Powered by GITEE.COM [1.1.23]
remote: Set trace flag b6498f96
remote: Push will publish a hidden email, make email public or abandon related commits
remote: error: hook declined to update refs/heads/main
To gitee.com:shenjuexiao/shenjuexiao.git
 ! [remote rejected] main -> main (hook declined)
error: failed to push some refs to 'git@gitee.com:shenjuexiao/shenjuexiao.git'
```

这个错误是 Gitee 服务端的 **pre-receive hook（预接收钩子）** 拒绝了你的推送。`hook declined` 表示服务端的自定义检查脚本未通过。

![Gitee remote error hook declined](https://cdn.jsdelivr.net/gh/shenjuexiao/shenjuexiao@main/Github_png/Gitee_remote_error_hook_declined.png)

## 常见原因及解决方案

### 1. **提交信息不符合规范**（最常见）
Gitee 可能要求提交信息符合某种格式（如包含 Issue ID、特定前缀等）。

**解决方案：**

```bash
# 查看最后一次提交信息
git log -1

# 修改最后一次提交信息
git commit --amend -m "新的提交信息"

# 强制推送（注意：如果多人协作需谨慎）
git push -f origin main
```

### 2. **邮箱/用户信息未公开**
错误信息中提到 "Push will publish a hidden email, make email public or abandon related commits"，这说明你的提交中使用了 Gitee 的私有邮箱。

**解决方案：**
- 在 Gitee 网页端设置中，将邮箱设置为公开
- 或者修改本地 Git 配置，使用提交邮箱（`gitee.com` -> `设置` -> `邮箱设置`）

```bash
git config user.email "你的真实邮箱@example.com"
# 然后修改最后一次提交的作者信息
git commit --amend --reset-author
```

### 3. **推送了敏感文件或大文件**
Gitee 可能限制了文件大小或禁止推送某些类型的文件。

**解决方案：**
- 检查是否有超过 100MB 的文件
- 使用 `.gitignore` 排除不必要的文件

### 4. **分支保护规则**
main 分支可能设置了保护规则，不允许直接推送。

**解决方案：**
- 通过 Pull Request/Merge Request 方式合并
- 或者联系仓库管理员修改保护规则

## 快速排查步骤

1. **查看详细错误信息：**
```bash
GIT_CURL_VERBOSE=1 git push origin main
```

2. **检查 Gitee 仓库设置：**
   - 进入仓库 → 管理 → 提交信息规范
   - 查看是否有强制要求

3. **尝试使用 --no-verify 跳过（如果允许）：**
```bash
git push --no-verify origin main
```

