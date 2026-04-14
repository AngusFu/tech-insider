---
name: book-pipeline
description: 启动源码深度解析书籍出版管线。编排 Agent Teams 并行工作，完成从克隆到终稿的全流程。与 /source-code-book:start 命令相同。
user-invocable: true
allowed-tools: Read, Write, Edit, Bash, Grep, Glob, Agent, AskUserQuestion
---

# 源码深度解析书籍 — 管线编排

你直接执行完整的书籍出版管线。按阶段顺序推进，每个阶段内根据需要**使用 subagent 并行执行**工作。

---

## 何时使用

- 用户调用此 skill（通过 Skill 工具或 `/source-code-book:start` 命令）
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
输出：代码库指标（文件数、行数、目录结构、语言分布）
```

1. 如果是 URL，`git clone`；如果是本地路径，验证存在
2. **检测语言分布**（polyglot）：
   ```bash
   find . -type f \( -name "*.py" -o -name "*.ts" -o -name "*.tsx" -o -name "*.js" -o -name "*.jsx" -o -name "*.go" -o -name "*.rs" -o -name "*.java" -o -name "*.rb" -o -name "*.swift" -o -name "*.kt" -o -name "*.cs" \) | \
     sed 's/.*\.//' | sort | uniq -c | sort -rn | head -10
   ```
3. 统计指标（用检测到的主语言替换 `<ext>`）：
   ```bash
   find . -type f -name "*.<ext>" | wc -l
   find . -type f -name "*.<ext>" -exec cat {} + | wc -l
   find . -type d \( -name "test*" -o -name "spec*" -o -name "__test*" \) | wc -l
   find . -type d -maxdepth 3 | grep -v node_modules | grep -v .git | sort
   ```
4. 阅读 README.md、入口文件，理解架构
5. 向用户展示分析结果

### Phase 2：规划

```
输入：代码库分析结果、书名、副标题、关注领域
输出：BOOK_PLAN.md + STYLE_GUIDE.md + EDITORIAL_PLAN.md
```

1. **并行启动** **book-planner** agent，传入代码库路径、书名、副标题、关注领域、`BOOK_DIR`
2. 等待完成后，验证三个文件都已生成
3. 向用户展示规划摘要（章节数、Part 分布）

### Phase 3：初稿撰写（5 Writer 并行）

```
输入：BOOK_PLAN.md + STYLE_GUIDE.md
输出：各章 chXX-*.md 文件
```

1. 按 `BOOK_PLAN.md` 的章节分配给 Writer：
   - **book-writer-foundation** → 基础篇（前 2-3 章）
   - **book-writer-core-loop** → 核心循环篇（对话引擎相关章节）
   - **book-writer-core-system** → 核心系统篇（基础设施相关章节）
   - **book-writer-tools** → 工具/子系统篇
   - **book-writer-integration** → 整合与工程实践篇
   > 每个 Writer 从 `BOOK_PLAN.md` 读取自己被分配的具体章节，不是写死在 agent 文件里
2. 每个 Writer 收到：`STYLE_GUIDE.md`、`BOOK_PLAN.md`、代码库路径
3. **5 个 Writer 并行启动**
4. 完成后收集结果，向用户展示进度

### Phase 4：审稿

#### 4a. 初审（逐章）

```
输入：所有章节 ch*.md + STYLE_GUIDE.md
输出：review-chXX.md（每章一份）
```

1. 为每个 ch*.md 文件启动 **book-chapter-reviewer** agent（逐章结构检查）
2. **多个 reviewer 并行启动**
3. 输出 `review-chXX.md` 到 `BOOK_DIR`

#### 4b. 复审（跨章一致性）

```
输入：所有 ch*.md + 所有 review-chXX.md + STYLE_GUIDE.md
输出：review-consistency.md
```

1. 启动 **book-consistency-reviewer** skill，检查跨章一致性
2. 输出 `review-consistency.md`

#### 4c. 判定

读取所有 review 报告：
- **有 FAIL** → Phase 5（返工）
- **全部 PASS** → Phase 6（验证）

### Phase 5：返工

```
输入：review 报告 + 原始章节文件
输出：修改后的 ch*.md
```

1. 对每个 FAIL 章节，向对应 Writer 发送返工指令
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
2. 按优先级修复：P0 → P1 → P2
3. 编写 4 个附录
4. 编译终稿 `book-final.md`

### Phase 9：交付

1. 展示终稿统计
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
