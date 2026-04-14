---
name: book-pipeline
description: 启动源码深度解析书籍出版管线。编排 Agent Teams 并行工作，完成从克隆到终稿的全流程。与 /source-code-book:start 命令相同。
user-invocable: true
allowed-tools: Read, Write, Edit, Bash, Grep, Glob, Agent, AskUserQuestion
---

# 源码深度解析书籍 — 管线编排

你直接执行完整的书籍出版管线。按阶段顺序推进，每个阶段内根据需要**使用 subagent 并行执行**工作。

---

## 参数解析

从用户输入中提取：
- 第一个非 `--` 开头的参数 → 仓库 URL 或本地路径（必需）
- `--title` → 书名（可选，选题阶段也可由 AI 生成）
- `--subtitle` → 副标题（可选）
- `--audience` → 目标读者（可选，如"Python 开发者"、"Go 后端工程师"）
- `--focus` → 重点关注领域（可选，逗号分隔）
- `--book-dir` → 书籍输出目录，默认 `<project-name>-book/`

如果缺少仓库路径，用 AskUserQuestion 询问用户补全。

确定 `BOOK_DIR` 后：
```bash
mkdir -p "$BOOK_DIR"
```

---

## 管线编排

按阶段顺序执行。每个阶段完成后，向用户报告并确认，再进入下一阶段。

### Phase 1：克隆 + 分析

```
输入：仓库 URL 或本地路径
输出：代码库指标（语言分布、文件数、行数、目录结构、测试覆盖）
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
   du -sh .
   ```
4. 阅读 README.md、入口文件，理解架构
5. 向用户展示分析结果，确认进入选题

### Phase 2：选题

```
输入：代码库分析结果、用户意图
输出：选题报告（TOPIC.md）
```

这是出版的第一个决策点。**先判断值不值得写、写给谁、怎么写，再动手。**

1. 基于代码分析结果，生成选题报告 `TOPIC.md`，包含：
   - **项目定位** — 这个项目是什么、解决什么问题
   - **技术亮点** — 最值得写的 3-5 个技术决策/架构设计
   - **目标读者** — 谁会读这本书、需要什么前置知识
   - **写作角度** — 从哪个角度切入（架构分析 vs 使用指南 vs 源码解读）
   - **书名建议** — 3-5 个备选书名 + 副标题
   - **章节数建议** — 12-18 章（根据代码库复杂度）
   - **不适合写的部分** — 哪些内容不适合深度解析
2. 向用户展示选题报告
3. 用户确认或修改后，进入大纲

### Phase 3：大纲

```
输入：TOPIC.md + 代码库
输出：BOOK_PLAN.md + STYLE_GUIDE.md + EDITORIAL_PLAN.md
```

1. 启动 **book-planner** agent，传入代码库路径、已确认的选题（TOPIC.md）、`BOOK_DIR`
2. 生成：
   - `BOOK_PLAN.md` — 详细章节大纲
   - `STYLE_GUIDE.md` — 写作风格指南
   - `EDITORIAL_PLAN.md` — 编辑管线计划
3. 向用户展示大纲摘要，确认后进入协调

### Phase 4：写前协调（关键！防止文风分裂）

```
输入：BOOK_PLAN.md + STYLE_GUIDE.md
输出：DEPENDENCIES.md（章节依赖图 + 交叉引用约定）
```

**5 个 Writer 并行写之前，先让他们互相知道边界在哪里。**

1. 生成章节依赖图 `DEPENDENCIES.md`：
   - 每个概念的"主章节"归属（谁深度分析）
   - 其他章节的交叉引用约定（"详见第X章"的具体措辞）
   - Writer 之间的内容边界（谁覆盖什么、不覆盖什么）
   - 相邻章节的过渡建议（前一章结尾自然引导到下一章开头）
2. 将 `DEPENDENCIES.md` 发送给每个 Writer 作为写作前的参考
3. 确认所有 Writer 已知晓边界后，进入初稿

### Phase 5：初稿撰写（5 Writer 并行）

```
输入：BOOK_PLAN.md + STYLE_GUIDE.md + DEPENDENCIES.md
输出：各章 chXX-*.md 文件
```

1. 按 `BOOK_PLAN.md` 的章节分配给 Writer：
   - **book-writer-foundation** → 基础篇（前 2-3 章）
   - **book-writer-core-loop** → 核心循环篇
   - **book-writer-core-system** → 核心系统篇
   - **book-writer-tools** → 工具/子系统篇
   - **book-writer-integration** → 整合与工程篇
   > 每个 Writer 从 `BOOK_PLAN.md` 读取自己被分配的具体章节，从 `DEPENDENCIES.md` 了解边界和交叉引用约定
2. 每个 Writer 收到：`STYLE_GUIDE.md`、`BOOK_PLAN.md`、`DEPENDENCIES.md`、代码库路径
3. **5 个 Writer 并行启动**
4. 完成后收集结果，向用户展示进度

### Phase 6：三审

#### 6a. 初审（逐章结构检查）

```
输入：所有章节 ch*.md + STYLE_GUIDE.md
输出：review-chXX.md（每章一份）
```

1. 为每个 ch*.md 文件启动 **book-chapter-reviewer** agent（逐章结构检查）
2. **多个 reviewer 并行启动**
3. 输出 `review-chXX.md` 到 `BOOK_DIR`

#### 6b. 技术评审（事实核对，关键！）

```
输入：所有章节 ch*.md + 源码库
输出：tech-review-chXX.md（每章一份）
```

**这不是格式检查，这是事实检查。检查章节里说的代码逻辑是否真的和源码一致。**

1. 为每个 ch*.md 文件启动 **book-technical-reviewer** agent
2. **多个 technical reviewer 并行启动**
3. 每个 reviewer 检查：
   - **代码逻辑** — 章节声称的行为是否和实际代码一致
   - **架构描述** — 描述的架构是否和实际代码结构一致
   - **代码引用上下文** — file:line 指向的代码是否真的是章节描述的那个功能
   - **数据准确性** — 性能数据、数字是否有依据
   - **设计决策真实性** — 描述的"备选方案"和"权衡"是否真实存在
4. 输出 `tech-review-chXX.md` 到 `BOOK_DIR`

#### 6c. 复审（跨章一致性）

```
输入：所有 ch*.md + 所有 review-chXX.md + 所有 tech-review-chXX.md + STYLE_GUIDE.md
输出：review-consistency.md
```

1. 启动 **book-consistency-reviewer** skill，检查跨章一致性
2. 同时检查：不同章节对同一技术点的描述是否矛盾
3. 输出 `review-consistency.md`

#### 6d. 终审（整体质量）

```
输入：所有 review 报告 + 所有 tech-review 报告 + 所有章节
输出：终审结论（是否可发稿）
```

1. 综合初审、技术评审、复审结果，判断整体质量
2. 输出终审结论：可发稿 / 需返工

判定：
- **需返工**（任何章节技术评审 FAIL）→ Phase 7
- **需返工**（初审或复审 FAIL）→ Phase 7
- **可发稿** → Phase 8（验证）

### Phase 7：返工

```
输入：review 报告 + tech-review 报告 + 原始章节文件
输出：修改后的 ch*.md
```

1. 对每个 FAIL 章节，向对应 Writer 发送返工指令（包括结构问题 + 事实错误）
2. **事实错误优先** — 技术评审的 Wrong 项必须先修正，因为可能影响后续章节的交叉引用
3. Writer 修改后，**重新跑 Phase 6a 的初审 + Phase 6b 的技术评审**
4. 最多返工 2 轮，超过后向用户报告并请求决定

### Phase 8：验证

```
输入：所有章节 ch*.md
输出：verification-status.md
```

1. 启动 **book-verifier** agent，运行自动化结构检查
2. 输出 `verification-status.md`
3. 展示验证结果表格。如有 FAIL，列出问题，用户确认后进入校对

### Phase 9：校对（两校并行）

```
输入：所有章节 ch*.md + STYLE_GUIDE.md
输出：proofread-1.md + proofread-2.md + 封面 + 序言
```

**两校制**：文字+交叉引用合并为一校，可读性单独为二校。

同时启动（互不依赖）：
- **一校**（文字 + 交叉引用）→ 使用 **book-proofreader** skill 的一校 + 二校合并模式 → `proofread-1.md`
- **二校**（可读性 + 叙事连贯 + 语气统一）→ 使用 **book-proofreader** skill 的三校模式 → `proofread-2.md`
- **封面设计** → 生成封面要求文档
- **序言撰写** → 生成序言初稿

全部完成后向用户报告。

### Phase 10：统稿（分 chunk 处理）

```
输入：所有 ch*.md + 三审报告 + 校对报告 + STYLE_GUIDE.md
输出：修复后的章节 + book-final.md
```

**为避免上下文窗口饱和，分三步处理：**

1. **第一轮：P0 修复** — 启动 **book-editor-in-chief** agent 处理所有 P0 问题（决策框格式统一、ASCII 图替换、缺失结构补齐），输出修复后的章节
2. **第二轮：P1 修复** — 启动新的 **book-editor-in-chief** agent 处理所有 P1 问题（内容去重、交叉引用修正、数据一致性），输出修复后的章节
3. **第三轮：P2 + 终稿编译** — 启动新的 **book-editor-in-chief** agent 处理 P2 问题（术语统一、难度缓冲、叙事过渡），编写 4 个附录，编译终稿 `book-final.md`
4. 向用户展示统稿结果（修复数量、终稿字数）

### Phase 11：交付

1. 展示终稿统计（`wc -l`、`wc -c`、章节数）
2. 将终稿文件交付给用户
3. 展示管线执行总结

---

## 失败处理原则

- **选题阶段** → 项目不适合写书（太小、太简单、文档不足），向用户说明
- **大纲阶段** → 分析不足，请求更多信息
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
