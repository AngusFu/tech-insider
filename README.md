# Source Code Book — 源码深度解析书籍生成器

基于 Agent Teams 的出版级技术书籍全自动管线。从克隆一份开源代码到输出终稿，全流程自动化。

## 灵感来源

这个插件诞生于一次完整的出版实践：我们用 Claude Code 的 Agent Teams 模式，为 [Hermes Agent](https://github.com/NousResearch/hermes-agent)（50K+ stars, MIT）写了一本 6.7 万字、16 章、4 个附录的深度解析书籍。整个过程包含：

- 5 个 Writer 并行撰写
- 3 轮审稿（初审 + 复审一致性 + 跨章检查）
- 3 轮校对（文字 + 交叉引用 + 可读性通读）
- 1 个 Editor-in-Chief 统稿
- 封面设计、序言撰写、附录编写

这个插件把那个工作流封装成了可复用的 Claude Code 插件。

## 快速开始

```bash
# 从 GitHub 仓库启动
claude --plugin-dir /path/to/source-code-book-plugin

# 然后在 Claude Code 中运行：
/source-code-book:start https://github.com/NousResearch/hermes-agent \
  --title "Hermes Agent 深度解析" \
  --subtitle "从源码理解自我进化的 AI Agent 架构"
```

## 架构

```
source-code-book-plugin/
├── .claude-plugin/
│   └── plugin.json              # 插件清单
├── skills/
│   ├── book-planner/            # 规划器：分析代码库、制定大纲、风格指南
│   ├── book-writer/             # Writer 模板：章节写作规范
│   ├── book-reviewer/           # 审稿人：结构检查 + 跨章一致性
│   ├── book-proofreader/        # 三校校对：文字/交叉引用/可读性
│   └── book-compiler/           # 统稿：修复 + 去重 + 终稿编译
├── agents/
│   ├── book-planner.md          # 规划 Agent
│   ├── book-writer-p1.md        # Writer Part 1（Ch01-03）
│   ├── book-writer-p2a.md       # Writer Part 2A（Ch04-05）
│   ├── book-writer-p2b.md       # Writer Part 2B（Ch06-07）
│   ├── book-writer-p3.md        # Writer Part 3（Ch08-10）
│   ├── book-writer-p45.md       # Writer Part 4-5（Ch11-16）
│   ├── book-reviewer-agent.md   # 自动审稿人
│   ├── book-verifier.md         # 结构验证器
│   └── book-editor-in-chief.md  # 统稿主编
├── commands/
│   └── start.md                 # /source-code-book:start 启动命令
├── docs/
│   └── workflow-experience.md   # 工作流经验总结
├── CHANGELOG.md
├── CONTRIBUTING.md
├── LICENSE
├── README.md
└── SECURITY.md
```

## 管线流程

```
┌─────────────────────────────────────────────────┐
│  1. 克隆 + 分析                                  │
│     git clone, wc -l, find, understand skill    │
└──────────────────┬──────────────────────────────┘
                   ▼
┌─────────────────────────────────────────────────┐
│  2. 规划 (book-planner skill)                    │
│     BOOK_PLAN.md, STYLE_GUIDE.md,               │
│     EDITORIAL_PLAN.md                           │
└──────────────────┬──────────────────────────────┘
                   ▼
┌─────────────────────────────────────────────────┐
│  3. 初稿 (5 writers 并行)                        │
│     p1 → Ch01-03                                │
│     p2a → Ch04-05                               │
│     p2b → Ch06-07                               │
│     p3 → Ch08-10                                │
│     p45 → Ch11-16                               │
└──────────────────┬──────────────────────────────┘
                   ▼
┌─────────────────────────────────────────────────┐
│  4. 三审                                         │
│     初审: book-reviewer-agent（逐章结构检查）      │
│     复审: reviewer-consistency（跨章一致性）       │
└──────────────────┬──────────────────────────────┘
                   ▼
┌─────────────────────────────────────────────────┐
│  5. 返工 (writers 根据 review 报告修改)           │
└──────────────────┬──────────────────────────────┘
                   ▼
┌─────────────────────────────────────────────────┐
│  6. 验证 (book-verifier 自动结构检查)             │
└──────────────────┬──────────────────────────────┘
                   ▼
┌─────────────────────────────────────────────────┐
│  7. 三校 (并行启动)                               │
│     一校: 文字校对（proofread-1.md）               │
│     二校: 交叉引用（proofread-2.md）               │
│     三校: 可读性（proofread-3.md）                 │
└──────────────────┬──────────────────────────────┘
                   ▼
┌─────────────────────────────────────────────────┐
│  8. 统稿 (book-editor-in-chief)                  │
│     P0/P1/P2 修复 + 内容去重 + 风格统一            │
└──────────────────┬──────────────────────────────┘
                   ▼
┌─────────────────────────────────────────────────┐
│  9. 附录 + 终稿                                  │
│     book-final.md（含 4 个附录）                   │
└─────────────────────────────────────────────────┘
```

## 章节结构模板

每章严格遵循：

```
> 开头隐喻/名言

\`\`\`mermaid
（架构图/流程图/状态图）
\`\`\`

## 技术深潜（代码引用带 file:line）
### 设计决策框（决策/备选/权衡/理由）

## 停下来想一想（2-3 个反思问题）
## 可迁移的设计原则（3-5 条可迁移原则）
```

## 经验教训（来自 Hermes 实践）

我们在实际工作中学到了这些教训，已内建到插件中：

1. **必须先研究参考书** — 不先读参考书就写，风格会偏移
2. **Mermaid 是唯一选择** — 禁止 ASCII 图，所有图必须可渲染
3. **Main Agent 不干活** — Main 只负责协调，所有工作委派给 subagent
4. **风格指南必须在写之前创建** — 不统一术语和结构，各章风格会分裂
5. **内容重合必须提前处理** — 每个概念指定一个主章节，其他只做交叉引用
6. **三校并行启动** — 一校/二校/三校/封面/序言可以同时进行
7. **Appendix 必须在正文之后** — 统稿时容易拼错位置
8. **进度必须及时反馈** — 不能让 main agent 干等所有 subagent 跑完才汇报

## 测试

```bash
# 本地测试插件
claude --plugin-dir /path/to/source-code-book-plugin

# 验证插件结构
ls -la .claude-plugin/plugin.json
ls -la skills/*/SKILL.md
ls -la agents/*.md
ls -la commands/start.md
```

## License

MIT — see [LICENSE](LICENSE)

## Contributing

See [CONTRIBUTING.md](CONTRIBUTING.md)

## Security

See [SECURITY.md](SECURITY.md)

## Changelog

See [CHANGELOG.md](CHANGELOG.md)
