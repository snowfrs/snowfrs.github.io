---
title: 迁移 SVN 到 Git
tags: SVN Git
---

​    最近需要考虑将某些项目从 SVN 迁移到 Git.  Atlassian有篇非常优秀的[文章][migrating-to-git]介绍如何从SVN迁移到Git; 如果你准备迁移到Gitlab, 那么Gitlab官方也有一篇[文章][migrating-to-gitlab]介绍如何操作.

​    从SVN迁移代码库到Git, 一般有三个选择:

- 一次性将整个项目代码库迁移到Git 并且 停用SVN
- 存在的项目继续使用 SVN, 新项目使用Git 
- 同一个项目既使用SVN又使用Git

实际情况往往是复杂的, 而且迁移到Git之后协同工作流应该做适当调整. 对于中小型项目, 可以选择一次性将整个代码库迁移到Git, 这种做法相对容易，当然操作前需要所有开发人员花时间熟悉和学习Git工作流.

<!--more-->

#### 获取 svn authors

```bash
svn log --quiet --username USERNAME [SVN_REPO_URL] | grep -E "r[0-9]+ \|.+ \|" | cut -d'|' -f2 | sed 's/ //g'|sort -u | tee authors.txt
```

#### 转换为 git authors 格式

```bash
awk 'NF{print $0 " = " $0" <"$0"@DOMAIN.com>"}' authors.txt | tee authors.git.txt
```

#### migration

##### 方法一: git-svn

使用git-svn clone

```bash
git svn clone -T TRUNK -t TAGS -b BRANCHES --authors-file=authors.git.txt --username USERNAME svn://your.server.tld/repo git_repo_name 
```

删除 svn peg-revisions

```bash
for p in $(git branch | grep @); do git branch -D $p; done
for p in $(git tag -l | grep @); do git tag -d $p; done
```

添加 remote origin

添加 remote origin之前, 我们先做一些本地配置

```bash
git config user.name "YOUR_USERNAME"
git config user.email "YOUR_EMAIL"
git config http.sslVerify false #Option 
```

现在添加 remote

```bash
git remote add origin git@your.server.tld:username/projectname.git
git push -u origin master
git push --all
git push --tags
```



##### 方法二: subgit

使用[subgit][subgit-link]工具

**配置**

```bash
subgit configure [SVN_REPO_URL] REPO.sugbit
```

执行后你会看到

```bash
SubGit version 3.3.10 ('Bobique') build #4368

Configuring writable Git mirror of remote Subversion repository:
    Subversion repository URL : svn://your.server.tld/repo
    Git repository location   : REPO.subgit

CONFIGURATION SUCCESSFUL

To complete SubGit installation do the following:

1) Adjust Subversion to Git branches mapping if necessary:
    /data/REPO.subgit/subgit/config
2) Define at least one Subversion credentials in default SubGit passwd file at:
    /data/REPO.subgit/subgit/passwd
   OR configure SSH or SSL credentials in the [auth] section of:
    /data/REPO.subgit/subgit/config
3) Optionally, add custom authors mapping to the authors.txt file(s) at:
    /data/REPO.subgit/subgit/authors.txt
4) Run SubGit 'install' command:
    subgit install REPO.subgit
```

按照要求配置好 

- branch mapping
- username/password
- authors mapping

**安装**

```bash
subgit install REPO.subgit
```

你将看到

```bash
SubGit version 3.3.10 ('Bobique') build #4368

Translating Subversion revisions to Git commits...

    Subversion revisions translated: 1218.
    Total time: 534 seconds.

INSTALLATION SUCCESSFUL

You are using SubGit in evaluation mode.
Your evaluation period expires on XXX (in 7 days).

Extend your trial or purchase a license key at https://subgit.com/pricing
```

```bash
subgit register --key subgit.key REPO.subgit
```

**Clone**

```bash
git clone REPO.subgit REPO.subgit.git
```

**上传**

```bash
cd REPO.subgit.git
git remote set-url origin git@your.server.tld:username/projectname.git
#master
git push origin master
#分支
for branch in `git branch -r | grep -Ev "HEAD|master" | cut -d/ -f2`; do git push origin remotes/origin/$branch:refs/heads/$branch; done  # 冒号后面为远程仓库中的地址,必须以refs/heads开头
#tags
git push --tags
```





[migrating-to-git]: https://www.atlassian.com/git/tutorials/migrating-overview
[migrating-to-gitlab]: https://docs.gitlab.com/ee/user/project/import/svn.html
[subgit-link]: https://subgit.com/



转换之后还需要处理 .gitignore

其他需要谨记的:

- 保持history 线性
- 尽量使用rebase 而不是 merge
