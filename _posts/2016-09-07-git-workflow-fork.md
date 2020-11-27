---
title: 与上游仓库保持同步
tags: Git-Workflow Git Fork 
---
> How to sync a fork with an upstream
<!--more-->

# 检查本地仓库的远程配置
```bash
git remote -v
origin  https://github.com/YOUR_USERNAME/YOUR_FORK.git (fetch)
origin  https://github.com/YOUR_USERNAME/YOUR_FORK.git (push)
```

# 添加上游远程仓库
```bash
git remote add upstream https://github.com/ORIGINAL_OWNER/ORIGINAL_REPOSITORY.git

git remote -v
origin  https://github.com/YOUR_USERNAME/YOUR_FORK.git (fetch)
origin  https://github.com/YOUR_USERNAME/YOUR_FORK.git (push)
upstream https://github.com/ORIGINAL_OWNER/ORIGINAL_REPOSITORY.git (fetch) 
upstream https://github.com/ORIGINAL_OWNER/ORIGINAL_REPOSITORY.git (push)
```
# 同步fork仓库
```bash
git checkout master
Switched to branch 'master'

git fetch upstream
git merge upstream/master
```

> 请注意 `merge` 和 `rebase` 的区别