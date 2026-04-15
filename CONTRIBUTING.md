# Contributing

## Development

```bash
# Clone the repo
git clone https://github.com/AngusFu/tech-insider.git
cd tech-insider

# Option 1: Local testing
claude --plugin-dir .

# Option 2: Via marketplace (closer to real installation)
/plugin marketplace add .
/plugin install tech-insider@tech-insider-marketplace

# Run in Claude Code
/tech-insider:make-book https://github.com/AngusFu/tech-insider --title "Test Book"
```

## Plugin Structure

```
tech-insider/
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
2. Add YAML frontmatter with `name`, `description`, `user-invocable`
3. Write actionable instructions in the body

## Adding a New Agent

1. Create `agents/<agent-name>.md`
2. Add YAML frontmatter with `name`, `description`, `allowed-tools`
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
