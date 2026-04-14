---
name: source-code-book
description: 启动源码深度解析书籍出版管线。与 /source-code-book:start 命令相同。根据管线阶段需要，使用 subagent 并行执行工作。
user-invocable: true
allowed-tools: Read, Write, Edit, Bash, Grep, Glob, Agent, AskUserQuestion
---

# 源码深度解析书籍 — Skill（管线编排）

你直接执行完整的书籍出版管线。按阶段顺序推进，每个阶段内根据需要**使用 subagent 并行执行**工作。

---

## 何时使用

- 用户调用此 skill（通过 Skill 工具或 `/source-code-book`）
- `/source-code-book:start` 命令会加载此 skill 并传递参数
- 适用于：完整书籍生产、快速体验、调试管线逻辑

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

按阶段顺序执行。每个阶段完成后，向用户报告进度，再进入下一阶段。

### Phase 1：克隆 + 分析

```
输入：仓库 URL 或本地路径
输出：代码库指标（文件数、行数、目录结构）
```

1. 如果是 URL，`git clone`；如果是本地路径，验证存在
2. 统计指标：
   ```bash
   find . -name "*.py" | wc -l
   find . -name "*.py" -exec cat {} + | wc -l
   find . -type d -maxdepth 2 | sort
   ```
3. 阅读 README.md、入口文件，理解架构
4. 向用户展示分析结果，确认进入下一步

### Phase 2：规划

```
输入：代码库分析结果、书名、副标题、关注领域
输出：BOOK_PLAN.md + STYLE_GUIDE.md + EDITORIAL_PLAN.md
```

1. 启动 **book-planner** agent，传入代码库路径、书名、副标题、关注领域、`BOOK_DIR`
2. 等待完成后，验证三个文件都已生成
3. 向用户展示规划摘要（章节数、Part 分布）

### Phase 3：初稿撰写（5 Writer 并行）

```
输入：BOOK_PLAN.md + STYLE_GUIDE.md
输出：ch01.md ~ ch16.md
```

1. 按章节分配给 Writer：
   - **book-writer-p1** → Ch01-03（基础篇）
   - **book-writer-p2a** → Ch04-05（核心篇 A）
   - **book-writer-p2b** → Ch06-07（核心篇 B）
   - **book-writer-p3** → Ch08-10（工具篇）
   - **book-writer-p45** → Ch11-16（多平台 + 工程篇）
2. 每个 Writer 收到：其负责的章节描述、`STYLE_GUIDE.md`、`BOOK_PLAN.md`、代码库路径
3. **5 个 Writer 并行启动**
4. 完成后收集结果，向用户展示进度（N/5 完成）

### Phase 4：三审

#### 4a. 初审（逐章）

```
输入：所有章节 ch*.md + STYLE_GUIDE.md
输出：review-chXX.md（每章一份）
```

1. 为每个 ch*.md 文件启动 **book-reviewer-agent**
2. 检查：结构合规、代码引用准确性、术语一致性
3. 输出 `review-chXX.md` 到 `BOOK_DIR`

#### 4b. 复审（跨章一致性）

```
输入：所有 ch*.md + 所有 review-chXX.md + STYLE_GUIDE.md
输出：review-consistency.md
```

1. 使用 **book-reviewer** skill 的"复审"部分，检查跨章一致性
2. 检查：术语不一致、内容重复、数据矛盾、设计决策矛盾、交叉引用错误
3. 输出 `review-consistency.md`

#### 4c. 判定

读取所有 review 报告：
- **有 FAIL** → Phase 5（返工）
- **全部 PASS** → Phase 6（验证）

### Phase 5：返工

```
输入：review 报告 + 原始章节文件
输出：修改后的 ch*.md
```

1. 对每个 FAIL 章节，向对应 Writer 发送返工指令（明确 FAIL 项 + 具体修复建议）
2. Writer 修改后，**重新跑 Phase 4a 的初审**
3. 最多返工 2 轮，超过后向用户报告并请求决定

### Phase 6：验证

```
输入：所有章节 ch*.md
输出：verification-status.md
```

1. 启动 **book-verifier** agent，运行自动化结构检查
2. 输出 `verification-status.md`
3. 展示验证结果表格。如有 FAIL，列出问题，用户确认后进入三校

### Phase 7：三校（并行启动）

```
输入：所有章节 ch*.md + STYLE_GUIDE.md
输出：proofread-1.md + proofread-2.md + proofread-3.md + 封面 + 序言
```

同时启动（互不依赖）：
- **一校**（文字校对）→ 使用 **book-proofreader** skill 的一校模式 → `proofread-1.md`
- **二校**（交叉引用）→ 使用 **book-proofreader** skill 的二校模式 → `proofread-2.md`
- **三校**（可读性）→ 使用 **book-proofreader** skill 的三校模式 → `proofread-3.md`
- **封面设计** → 生成封面要求文档
- **序言撰写** → 生成序言初稿

全部完成后向用户报告。

### Phase 8：统稿

```
输入：所有 ch*.md + 三审报告 + 三校报告 + STYLE_GUIDE.md
输出：修复后的章节 + book-final.md
```

1. 启动 **book-editor-in-chief** agent
2. 按优先级修复：P0（格式/ASCII/结构）→ P1（去重/交叉引用/数据）→ P2（术语/过渡）
3. 编写 4 个附录（A 文件导航、B 工具参考、C 设计决策汇总、D 术语表）
4. 编译终稿 `book-final.md`
5. 向用户展示统稿结果（修复数量、终稿字数）

### Phase 9：交付

1. 展示终稿统计：`wc -l`、`wc -c`、章节数
2. 将终稿文件交付给用户
3. 展示管线执行总结

---

## 失败处理原则

- **规划阶段失败** → 向用户报告分析不足，请求更多信息
- **审稿 FAIL** → 返工，最多 2 轮
- **验证 FAIL** → 列出具体问题，用户确认后继续
- **统稿阶段** → 记录所有 P0/P1/P2 修复，最终报告中展示

## 进度反馈

每个阶段完成后输出：
```
✅ Phase N: [阶段名] — 完成
  - [关键产出 1]
  - [关键产出 2]
  - [耗时 / 文件数 / 其他指标]
```

整个管线完成时，输出完整总结。
