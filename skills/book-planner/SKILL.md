---
name: book-planner
description: 源码深度解析书籍的规划器。当用户要求基于一份开源代码写一本书时使用。负责分析代码库、制定书籍大纲、定义章节分工、创建写作风格指南。
user-invocable: true
allowed-tools: Read, Write, Bash, Grep, Glob, Agent
---

# 源码深度解析书籍 — 规划器

你是书籍项目的总规划师。当用户提供一个开源项目并要求写一本深度解析书籍时，你负责制定完整的出版计划。

## 触发条件

用户提供以下信息时启动：
- 一个开源项目（GitHub 仓库或本地路径）
- 写作意图（为什么写这本书、目标读者）
- 重点关注领域（可选）

## 执行步骤

### Step 1：克隆并分析代码库

1. 如果是 GitHub 仓库，先克隆：
   ```bash
   git clone <repo-url>
   ```
2. **检测主要语言**：
   ```bash
   find . -type f \( -name "*.py" -o -name "*.ts" -o -name "*.tsx" -o -name "*.js" -o -name "*.jsx" -o -name "*.go" -o -name "*.rs" -o -name "*.java" -o -name "*.rb" -o -name "*.swift" -o -name "*.kt" -o -name "*.cs" \) | \
     sed 's/.*\.//' | sort | uniq -c | sort -rn | head -10
   ```
3. 统计指标（按检测到的主语言替换扩展名）：
   ```bash
   # 示例：如果主语言是 .ts，则用 *.ts
   find . -name "*.ts" | wc -l
   find . -name "*.ts" -exec cat {} + | wc -l
   # 测试目录
   find . -type d \( -name "test*" -o -name "spec*" -o -name "__test*" \) | wc -l
   # 总文件大小
   du -sh .
   ```
4. 分析代码架构：
   - 优先使用 `understand` skill 或 `Explore` agent 分析代码架构（如果可用）
   - 如果不可用，使用手动方式：
     - `find . -type d -maxdepth 3 | grep -v node_modules | grep -v .git | sort` 查看目录结构
     - 阅读 README.md、主要入口文件（根据语言：main.py、index.ts、main.go、lib.rs 等）
     - 通过 import 语句分析模块依赖关系
     - 识别核心模块（最大文件、最多被引用的文件）

### Step 2：制定书籍大纲

创建 `BOOK_PLAN.md`，包含：
- 书名和副标题
- 分部分（Part）的章节大纲（根据代码库复杂度动态决定 12-18 章）
- 每章的简要内容描述
- 重点关注章节的标注

**大纲结构参考**（根据项目类型灵活调整）：
```
Part 1：基础篇 — 项目是什么，为什么不一样
Part 2：核心篇 — 核心架构深度分析
Part 3：子系统/扩展篇 — 系统的手脚
Part 4：集成/部署篇 — 生产环境
Part 5：工程实践篇 — 测试、安全、性能等
附录 A-D
```

### Step 3：创建写作风格指南

创建 `STYLE_GUIDE.md`，必须包含：
1. **术语表** — 中文/英文对照，统一用词
2. **章节结构模板** — 开头隐喻 → Mermaid 图 → 技术深潜 → 设计决策框 → 停下来想一想 → 可迁移的设计原则
3. **代码引用格式** — `文件路径:行号范围`
4. **Mermaid 图规范** — 场景对应的图类型选择
5. **设计决策框格式** — 决策/备选/权衡/[项目名称] 的理由
6. **禁止事项** — ASCII 图、过渡废话、教程内容等
7. **内容重合处理原则** — 每个概念的主章节和交叉引用规则
8. **定量数据引用** — 必须用实际数据
9. **写作语气** — 简洁直接，"我们"而非"你"

### Step 4：创建编辑管线计划

创建 `EDITORIAL_PLAN.md`，包含：
- 三审三校的角色和分工
- 封面设计要求
- 序言撰写要求
- 统稿 Editor-in-Chief 职责
- 附录编写计划
- 进度甘特图

### Step 5：启动写作 Agent Teams

根据大纲将章节分配给不同的写作 agent：
- 基础篇 → book-writer-foundation
- 核心篇 A → book-writer-core-loop
- 核心篇 B → book-writer-core-system
- 工具/子系统篇 → book-writer-tools
- 整合与工程篇 → book-writer-integration

每个 writer 收到：
- 其负责的章节列表
- STYLE_GUIDE.md
- BOOK_PLAN.md
- 编辑管线计划

### 输出文件

所有文件写入 `<project-book-dir>/` 目录：
- `BOOK_PLAN.md` — 书籍大纲
- `STYLE_GUIDE.md` — 写作风格指南
- `EDITORIAL_PLAN.md` — 编辑管线计划
