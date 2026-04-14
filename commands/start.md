---
name: start
description: 启动完整的源码深度解析书籍出版管线。需要一个 GitHub 仓库 URL 或本地路径，以及书名。
argument-hint: <repo-url-or-path> --title "书名" [--subtitle "副标题"] [--focus "领域"]
---

# 源码深度解析书籍 — 启动命令

你是整个出版管线的**总指挥（Orchestrator）**。你的职责是：解析用户输入、确认参数、加载并启动 `source-code-book` skill 执行完整管线。**所有实际工作由 skill 完成，你只负责参数解析和启动。**

---

## 参数解析

从 `$ARGUMENTS` 中提取：
- 第一个非 `--` 开头的参数 → 仓库 URL 或本地路径（必需）
- `--title` → 书名（必需）
- `--subtitle` → 副标题（可选）
- `--focus` → 重点关注领域（可选，逗号分隔）
- `--chapters` → 章节数，默认 16
- `--book-dir` → 书籍输出目录，默认 `<project-name>-book/`

**如果缺少必需参数（仓库路径或 title），用 AskUserQuestion 询问用户补全。**

确认参数后，**加载 `source-code-book` skill 并将参数传递给它**，由 skill 执行完整的出版管线。

## 技能加载

使用 Skill 工具加载 `source-code-book` skill，告知用户你已收到：
- 仓库：`<repo>`
- 书名：`<title>`
- 副标题：`<subtitle>`（如有）
- 关注领域：`<focus>`（如有）

然后按照 skill 中的管线流程逐步执行。
