---
name: "worktree-guide"
description: "Learn how to use git worktree for parallel development, including Claude Code's built-in worktree tools"
license: "MIT"
metadata:
  author: "lubin"
  version: "1.0.0"
  created: "2026-07-27"
---

# Git Worktree Guild

## Overview

Git worktree 允许你在同一个仓库中同时检出多个分支到不同的目录，实现真正的并行开发。

**核心场景:**
- 正在 feature-a 分支写代码，突然需要切到 main 修一个紧急 bug —— 不用 stash 或 commit 当前工作
- 同时开发多个独立功能，每个功能在独立的目录和分支中
- Code review 时快速检出 PR 分支，不影响当前工作
- Claude Code 中启动隔离的 worktree 环境运行 agent，不污染主工作区

## 快速命令表

| 场景 | 命令 |
|------|------|
| 查看所有 worktree | `git worktree list` |
| 创建新 worktree + 分支 | `git worktree add ../path feature-b` |
| 创建 worktree 用已有分支 | `git worktree add ../path existing-branch` |
| 删除 worktree | `git worktree remove ../path` |
| 修剪已删除目录的记录 | `git worktree prune` |

## 手动管理 worktree (git 命令)

### 1. 创建 worktree

```bash
# 基于当前 HEAD 创建新分支并检出到指定目录
git worktree add ../worktree-demo-feature-b feature-b

# 基于指定 commit/branch 创建
git worktree add ../worktree-hotfix -b hotfix-urgent main

# 检出已有分支（不创建新分支）
git worktree add ../worktree-existing-branch existing-branch
```

### 2. 查看所有 worktree

```bash
git worktree list

# 输出示例:
# D:/project/main               abc123 [main]
# D:/project/../worktree-feat   def456 [feature-a]
```

### 3. 在 worktree 中工作

每个 worktree 是独立的目录，有自己的工作区和索引。可以直接 `cd` 进去正常开发:

```bash
cd ../worktree-demo-feature-b
# 正常写代码、add、commit
git add .
git commit -m "work on feature b"
```

**关键点:** 分支在一个 worktree 中被检出时，其他 worktree 不能再检出同一个分支。

### 4. 删除 worktree

```bash
# 先确保没有未提交的更改，然后删除
git worktree remove ../worktree-demo-feature-b

# 强制删除（丢弃未提交更改）
git worktree remove --force ../worktree-demo-feature-b
```

### 5. 清理无效记录

如果手动删除了 worktree 目录（如 `rm -rf`），用 prune 清理记录:

```bash
git worktree prune
```

## 实用技巧

### 同时运行多个 VS Code 窗口

每个 worktree 可以用独立的 VS Code 窗口打开:

```bash
code ../worktree-demo-feature-b
code ../worktree-hotfix
```

### 共享配置和依赖

worktree 共享 `.git` 目录（实际是引用指向主仓库的 `.git`），不会重复占用磁盘。但 node_modules、venv 等依赖目录需要各自安装。

### 跨 worktree 共享依赖

```bash
# Node.js: 使用同一个 node_modules（注意版本一致性）
# 在 worktree 中创建符号链接
mklink /D node_modules ..\main-project\node_modules

# Python: 使用同一个虚拟环境
# 在 worktree 中直接激活主仓库的 venv
source ../main-project/.venv/Scripts/activate
```

## 常见问题

**Q: 为什么 `git worktree remove` 报错 "directory contains uncommitted changes"?**

A: worktree 中有未提交的修改。要么提交，要么使用 `--force` 强制删除。

**Q: 删除 worktree 分支会丢失吗？**

A: `git worktree remove` 只删除目录，不会删除分支。如果要同时删除分支:
```bash
git branch -d feature-b
git worktree prune
```

**Q: worktree 之间能共享 staged changes 吗？**

A: 不能。每个 worktree 有独立的 index（暂存区）和工作目录。commit 之后才会共享。

## 进阶: 同一分支多 worktree

默认情况下，一个分支只能在一个 worktree 中检出。如果要同时检出同一个分支:

```bash
# 使用 --detach 创建分离 HEAD 的 worktree
git worktree add --detach ../worktree-temp main
```
