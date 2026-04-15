# Security

## Plugin Safety

This plugin:
- Only reads and writes files within the project directory
- Executes `git clone` to fetch source code
- Runs basic shell commands (`find`, `wc`, `grep`) for code analysis
- Does not send code or data to external services beyond the LLM API calls

## Agent Permissions

Agents in this plugin have the following tool restrictions:
- **Writer**: `Read, Write, Edit, Grep, Glob, Bash` — write chapters, run code analysis commands
- **Reviewers**: `Read, Write, Grep, Glob, Bash` — read chapter files, write reports
- **Verifiers**: `Read, Write, Grep, Glob, Bash` — automated structural checks
- **Editor-in-Chief**: `Read, Write, Edit, Grep, Glob, Bash` — fix issues in chapter files
- **Final Reviewer**: `Read, Write, Grep, Glob, Bash` — read compiled book, write report

No agent has `Agent` tool access — teammates in Agent Teams cannot spawn other agents.

## External Dependencies

- The `book-planner` skill may reference the `understand` skill if available (optional external plugin)
- No network access beyond git clone operations

## Reporting Issues

If you find a security issue, please open an issue in the repository.
