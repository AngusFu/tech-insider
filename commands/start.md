---
name: start
description: 启动完整的源码深度解析书籍出版管线。需要一个 GitHub 仓库 URL 或本地路径，以及书名。
argument-hint: <repo-url-or-path> --title "书名" [--subtitle "副标题"] [--focus "领域"]
---

# 源码深度解析书籍 — 启动命令

启动完整的书籍出版管线。

用户输入：$ARGUMENTS

## 解析参数

从 $ARGUMENTS 中提取：
- 第一个非 `--` 开头的参数 → 仓库 URL 或本地路径（必需）
- `--title` → 书名（必需）
- `--subtitle` → 副标题（可选）
- `--focus` → 重点关注领域（可选，逗号分隔）
- `--chapters` → 章节数，默认 16
- `--book-dir` → 书籍输出目录，默认 `<project-name>-book/`

如果缺少必需参数（仓库路径或 title），用 AskUserQuestion 询问用户。

## 执行流程

1. **克隆仓库**（如果是 URL）或验证本地路径存在
2. **分析代码库** — 统计文件数、行数、测试数，识别架构
3. **创建书籍目录** — `--book-dir` 或 `<project-name>-book/`
4. **启动 book-planner skill** — 生成 BOOK_PLAN.md、STYLE_GUIDE.md、EDITORIAL_PLAN.md
5. **并行启动 5 个 Writer agents**（p1, p2a, p2b, p3, p45）撰写初稿
6. **并行启动 reviewers**（book-reviewer-agent + 跨章一致性检查）
7. **如果有 FAIL → 返工**，否则继续
8. **启动 book-verifier** 自动结构检查
9. **并行启动三校**（一校文字 + 二校交叉引用 + 三校可读性 + 封面设计 + 序言撰写）
10. **启动 Editor-in-Chief** 统稿（P0/P1/P2 修复 + 去重 + 风格统一）
11. **编写附录** + 编译终稿 `book-final.md`

每个阶段完成后自动进入下一阶段，遇到 FAIL 时暂停并等待用户确认。
