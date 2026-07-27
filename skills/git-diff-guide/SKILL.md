---
name: "git-diff-guide"
description: "Master git diff commands — compare working tree, staging area, commits, and branches with practical examples"
license: "MIT"
metadata:
  author: "lubin"
  version: "1.0.0"
  created: "2026-07-27"
---

# Git Diff Guide

## Overview

`git diff` 是 Git 中最常用的命令之一，用于比较文件、提交、分支之间的差异。

**核心概念 —— Git 的三种状态:**

```
工作区 (Working Tree)  ──git add──>  暂存区 (Staging Area)  ──git commit──>  仓库 (Repository)
        │                                    │                               │
        └────────── git diff ────────────────┘                               │
                   (工作区 vs 暂存区)                                          │
                                             └────── git diff --staged ──────┘
                                                     (暂存区 vs 最新提交)       │
                                                                              │
                                             └────────── git diff HEAD ───────┘
                                                    (工作区 vs 最新提交)
```

## 快速命令表

| 场景 | 命令 |
|------|------|
| 查看工作区未暂存的改动 | `git diff` |
| 查看暂存区即将提交的改动 | `git diff --staged` |
| 查看工作区所有改动（含暂存） | `git diff HEAD` |
| 比较两个提交 | `git diff commit1 commit2` |
| 比较两个分支 | `git diff main..feature-b` |
| 查看某文件的改动 | `git diff -- path/to/file` |
| 只看文件名列表 | `git diff --name-only` |
| 统计增删行数 | `git diff --stat` |
| 按单词差异显示 | `git diff --word-diff` |
| 忽略空白字符 | `git diff -w` |

## 基础用法

### 1. 工作区 vs 暂存区

```bash
# 查看工作区中还未 git add 的改动
git diff
```

这是最常用的用法。显示自上次 `git add` 之后，工作区中的文件改动。

### 2. 暂存区 vs 最新提交

```bash
# 查看已经 git add 但还未 git commit 的改动
git diff --staged
# 或等效写法
git diff --cached
```

`--staged` 和 `--cached` 完全等价，推荐用 `--staged` 语义更清晰。

### 3. 工作区 vs 最新提交

```bash
# 查看工作区相对于最新一次 commit 的所有改动（含暂存+未暂存）
git diff HEAD
```

### 4. 指定文件

```bash
# 只看特定文件的差异
git diff -- src/index.js

# 只看某个目录
git diff -- src/components/

# 暂存区中某个文件
git diff --staged -- src/utils.js

# 排除某些文件
git diff -- . ':!package-lock.json'
```

### 5. 比较提交和分支

```bash
# 比较两个提交
git diff abc123 def456

# 比较当前分支与 main 分支的差异
git diff main

# 更明确的三点语法（功能分支与 main 的共同祖先对比）
git diff main...feature-b

# 比较两个分支
git diff main..feature-b
```

**两点 vs 三点语法:**
- `main..feature-b` — feature-b 相对于 main 的所有改动（main 为基准，累积 view）
- `main...feature-b` — 从共同祖先开始，feature-b 独有的改动（只看 feature-b 做了什么）

## 常用选项

### 输出格式控制

```bash
# 只显示改动的文件名
git diff --name-only

# 显示文件名 + 改动状态 (Added/Modified/Deleted)
git diff --name-status

# 统计增删行数，不显示具体内容
git diff --stat

# 精简的 stat 输出
git diff --stat=short

# 按单词而不是按行显示差异
git diff --word-diff

# 彩色单词差异（更直观）
git diff --word-diff=color

# 逐块摘要
git diff --stat --summary
```

### 查找与过滤

```bash
# 忽略空白字符差异（只比较非空白内容）
git diff -w

# 忽略所有空白差异（包括行末空格等）
git diff --ignore-all-space

# 搜索包含特定关键字的差异块
git diff -S "searchTerm"

# 搜索匹配正则的差异块
git diff -G "regexPattern"

# 只看某个函数/方法范围内的改动
git diff -L :functionName:file.js
```

### 上下文控制

```bash
# 显示更多上下文行（默认3行）
git diff -U10

# 只显示改动的函数名（不显示具体改了什么）
git diff --function-context
```

## 实用场景

### 场景 1: Code Review 时快速浏览改动

```bash
# 第一步：看改动了哪些文件
git diff main --name-status

# 第二步：看每个文件的改动量
git diff main --stat

# 第三步：逐个文件审查
git diff main -- src/module.js
```

### 场景 2: Commit 前自我检查

```bash
# 检查即将提交的内容
git diff --staged

# 确认没有遗漏的文件
git diff --staged --name-only

# 检查是否有调试代码
git diff --staged | grep -E "console\.|debugger|TODO"
```

### 场景 3: 对比某个文件的历史版本

```bash
# 对比文件在两个提交间的差异
git diff HEAD~3 HEAD -- path/to/file

# 查看文件在某个提交中的改动
git diff abc123^! -- path/to/file
# abc123^! 表示 "abc123 的父提交..abc123"，即只看这一个提交的改动
```

### 场景 4: 生成补丁文件

```bash
# 生成补丁（保留提交信息）
git diff main > feature.patch

# 包含二进制文件的补丁
git diff --binary main > feature.patch

# 应用补丁
git apply feature.patch
```

### 场景 5: 查看 merge 后的改动

```bash
# 查看合并提交引入的所有改动
git diff merge-commit^1 merge-commit

# 查看合并提交相对于第一个父分支的改动
git diff merge-commit^1..merge-commit

# 查看合并冲突解决中引入的差异
git diff --diff-filter=U
```

## diff 过滤器 (--diff-filter)

```bash
# 只看新增的文件
git diff --diff-filter=A

# 只看修改的文件
git diff --diff-filter=M

# 只看删除的文件
git diff --diff-filter=D

# 组合过滤：新增+修改
git diff --diff-filter=AM

# 支持的值: A(Added) M(Modified) D(Deleted) R(Renamed) C(Copied) T(Type changed) U(Unmerged)
```

## 高级技巧

### 1. 比较工作区与任意提交

```bash
# 工作区 vs 特定提交
git diff abc123

# 暂存区 vs 特定提交
git diff --staged abc123
```

### 2. 查看某次提交的内容

```bash
# 查看某次提交改了什么
git diff abc123^!

# 等价于
git show abc123
```

### 3. 查看分支分叉后的差异

```bash
# 找到共同祖先（merge base）
git merge-base main feature-b

# 查看 feature-b 从分叉点开始的独有改动
git diff $(git merge-base main feature-b) feature-b
# 等效于三点语法:
git diff main...feature-b
```

### 4. 外部 diff 工具

```bash
# 使用外部 diff 工具（如 VS Code）
git difftool

# 指定工具
git difftool --tool=vscode

# 比较暂存区
git difftool --staged
```

### 5. 检查空白字符问题

```bash
# 高亮行末空白
git diff --check

# 检查空白 + 显示错误
git diff --check --ws-error-highlight=all
```

## 工作流速查

```
修改文件后想看看改了什么:
    git diff

看改动不错, add 到暂存区:
    git add .

看暂存区要提交什么:
    git diff --staged

没问题, 提交:
    git commit -m "message"

提交完了, 看看这次提交和 main 的差异:
    git diff main

Code Review, 列出改动文件清单:
    git diff main --name-status
```

## 常见问题

**Q: `git diff` 没输出怎么办?**

A: 说明工作区和暂存区完全一致，没有未暂存的改动。检查是否已经 `git add` 了所有文件，用 `git diff --staged` 查看暂存区。

**Q: 如何对比两个完全不相关的提交?**

A: 直接用 `git diff commit1 commit2`，Git 会直接比较两个快照。

**Q: diff 输出太长怎么看?**

A: 组合使用 `--stat`（概览）、`--name-only`（文件列表）、或指定文件路径缩小范围。

**Q: 如何只看新增的行，不看删除的行?**

A: `git diff --diff-filter=A` 只看新增文件。如果是要在 diff 输出中只看 `+` 行，可以管道过滤: `git diff | grep "^+"`

**Q: 如何让 diff 忽略文件权限变化?**

A: 使用 `git diff --no-color` 可以去掉颜色。文件权限变更不显示差异内容，默认情况下 Windows/Linux 交叉开发时 Git 会忽略 exec 权限变化（`core.fileMode=false`）。
