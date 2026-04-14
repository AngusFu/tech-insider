# Contributing

## Development

```bash
# Clone the repo
git clone https://github.com/example/source-code-book-plugin.git
cd source-code-book-plugin

# Test locally with Claude Code
claude --plugin-dir .

# In Claude Code, run:
/source-code-book:start https://github.com/example/repo --title "Test Book"
```

## Plugin Structure

```
source-code-book-plugin/
├── .claude-plugin/
│   └── plugin.json          # Plugin manifest
├── skills/                  # Skill definitions
│   └── <skill-name>/
│       └── SKILL.md
├── agents/                  # Agent definitions
│   └── <agent-name>.md
├── commands/                # Slash commands
│   └── <command-name>.md
├── docs/                    # Documentation
├── LICENSE
├── README.md
└── CHANGELOG.md
```

## Adding a New Skill

1. Create `skills/<skill-name>/SKILL.md`
2. Add YAML frontmatter with `name`, `description`, `user-invocable`, `allowed-tools`
3. Write actionable instructions in the body

## Adding a New Agent

1. Create `agents/<agent-name>.md`
2. Add YAML frontmatter with `name`, `description`, `tools`
3. Write the agent's system prompt in the body

## Adding a New Command

1. Create `commands/<command-name>.md`
2. Add YAML frontmatter with `name`, `description`, `argument-hint`
3. Use `$ARGUMENTS` to capture user input

## Submitting Changes

1. Fork the repository
2. Create a feature branch
3. Test with `claude --plugin-dir .`
4. Submit a pull request
