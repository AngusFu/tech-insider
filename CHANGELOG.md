# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [1.0.0] - 2026-04-14

### Added
- Initial release of source-code-book plugin
- 5 skills: book-planner, book-writer-template, book-consistency-reviewer, book-proofreader, book-pipeline (管线编排)
- 11 agents: book-planner (通用动态章节), 5 writers (foundation/core-loop/core-system/tools/integration), book-chapter-reviewer (逐章结构审查), book-technical-reviewer (技术事实核对), book-verifier, book-editor-in-chief, book-preface-writer
- `/source-code-book:make-book` command for full pipeline launch
- Complete documentation: README.md, workflow-experience.md
- `.work/` hidden directory for intermediate files (review/proofread/verification reports)
- MIT License
