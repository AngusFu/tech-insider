---
name: source-code-book
description: 启动源码深度解析书籍出版管线。与 /source-code-book:start 命令相同，但以 skill 模式运行——所有工作由当前 Agent 直接完成，不委派给 subagent。适用于快速体验、小批量章节撰写或调试管线。
user-invocable: true
allowed-tools: Read, Write, Edit, Bash, Grep, Glob
---

# 源码深度解析书籍 — Skill（内联模式）

你直接执行完整的书籍出版管线。**禁止使用 Agent 工具 spawn subagent——所有工作由你亲自完成。**

---

## 何时使用

- 用户调用此 skill（`/source-code-book` 或通过 Skill 工具）
- 与 `/source-code-book:start` 命令功能相同，但**不委派给 subagent**
- 适用于：快速体验管线、小批量章节撰写、调试管线逻辑

## 参数解析

从用户输入中提取：
- 第一个非 `--` 开头的参数 → 仓库 URL 或本地路径（必需）
- `--title` → 书名（必需）
- `--subtitle` → 副标题（可选）
- `--focus` → 重点关注领域（可选，逗号分隔）
- `--chapters` → 章节数，默认 16
- `--book-dir` → 书籍输出目录，默认 `<project-name>-book/`

如果缺少必需参数，用 AskUserQuestion 询问用户补全。

确定 `BOOK_DIR` 后：
```bash
mkdir -p "$BOOK_DIR"
```

---

## 管线编排

按阶段顺序执行。**所有工作你亲自完成，不 spawn subagent。**

### Phase 1：克隆 + 分析

1. 如果是 URL，`git clone`；如果是本地路径，验证存在
2. 统计指标：
   ```bash
   find . -name "*.py" | wc -l
   find . -name "*.py" -exec cat {} + | wc -l
   find . -type d -maxdepth 2 | sort
   ```
3. 阅读 README.md、入口文件，理解架构
4. 向用户展示分析结果

### Phase 2：规划

1. **你亲自创建** `BOOK_PLAN.md`：
   - 分析代码库架构
   - 制定 16 章大纲（5 个 Part）
   - 每章描述、关键源文件、设计决策
2. **你亲自创建** `STYLE_GUIDE.md`：
   - 术语表、章节结构模板、代码引用格式
   - Mermaid 规范、决策框格式、禁止事项
   - 内容重合处理原则
3. **你亲自创建** `EDITORIAL_PLAN.md`：
   - 三审三校分工、封面/序言要求、附录计划
4. 向用户展示规划摘要

### Phase 3：初稿撰写

1. 按 Part 逐个撰写章节（因无 subagent，串行执行）：
   - Ch01-03：基础篇
   - Ch04-05：核心篇 A
   - Ch06-07：核心篇 B
   - Ch08-10：工具篇
   - Ch11-16：多平台 + 工程篇
2. 每章遵循 STYLE_GUIDE.md：
   - 开头隐喻/名言
   - Mermaid 图（1-3 张）
   - 技术深潜（带 file:line 引用）
   - 设计决策框
   - 停下来想一想（≥2 问题）
   - 可迁移的设计原则（≥3 条）
3. 每完成一个 Part，向用户报告进度

### Phase 4：审稿

1. **你亲自运行**逐章检查：
   - 结构合规、代码引用准确性、术语一致性
2. **你亲自运行**跨章一致性检查：
   - 术语不一致、内容重复、数据矛盾
3. 生成 `review-*.md` 和 `review-consistency.md`
4. 如有 FAIL，自行修复后再检查

### Phase 5：验证

1. **你亲自运行**自动化结构检查：
   - ASCII 图检测、Mermaid 计数、开头/结尾段落
   - 决策框格式、代码引用数量
2. 输出 `verification-status.md`

### Phase 6：三校

**你亲自运行**三项检查（可串行，也可按依赖关系分批）：
- 一校：文字校对 → `proofread-1.md`
- 二校：交叉引用 → `proofread-2.md`
- 三校：可读性 → `proofread-3.md`

同时生成：封面要求文档、序言初稿。

### Phase 7：统稿

1. 按优先级修复所有问题：
   - P0：决策框格式、ASCII 图、缺失结构
   - P1：内容去重、交叉引用、数据一致性
   - P2：术语统一、难度缓冲、叙事过渡
2. 编写 4 个附录（A 文件导航、B 工具参考、C 设计决策汇总、D 术语表）
3. 编译终稿 `book-final.md`

### Phase 8：交付

1. 展示终稿统计（行数、字节数、章节数）
2. 将所有终稿文件交付给用户
3. 展示管线执行总结

---

## 与命令模式的区别

| 维度 | /source-code-book:start（命令） | source-code-book（skill） |
|------|------|------|
| 执行方式 | 委派给 subagent（并行） | 当前 Agent 亲自执行（串行） |
| 速度 | 快（5 Writer 并行） | 较慢（逐个 Part） |
| 适用场景 | 完整书籍生产 | 快速体验、调试、少量章节 |
| 可用工具 | 含 Agent | **不含 Agent** |
| 进度反馈 | 各阶段自动报告 | 每 Part 完成后报告 |

## 失败处理

- **规划阶段**：分析不足 → 请求更多信息
- **审稿 FAIL**：自行修复，最多 2 轮
- **验证 FAIL**：列出问题，用户确认后继续
- **统稿**：记录所有修复，最终报告展示
