---
title: git custom hook
tags: Git-Workflow Git
---

<!--more-->

## server-side hooks
 - pre-receive
 - update
 - post-receive
### custom server-side hooks order
Gitlab官方给出的 git custom hook 执行顺序是:

 - \<project\>.git/hooks/\<hook_name\>
 - \<project\>.git/custom_hooks/\<hook_name\>
 - \<project\>.git/custom_hooks/\<hook_name\>.d/*
 - \<project\>.git/hooks/\<hook_name\>.d/*

[原文][gitlab_custom_hooks]

## client hooks
### committing-workflow hooks
 - pre-commit
 - prepare-commit-msg
 - commit-msg
 - post-commit
### email workflow hooks
 - applypatch-msg
 - pre-applypatch
 - post-applypatch
### other client hooks
 - pre-rebase
 - post-rewrite
 - post-checkout
 - post-merge
 - pre-push
## hooks执行阶段
 ![hooks](/assets/img/blog/git/git-custom-hook.webp)

[gitlab_custom_hooks]: https://docs.gitlab.com/ee/administration/custom_hooks.html
