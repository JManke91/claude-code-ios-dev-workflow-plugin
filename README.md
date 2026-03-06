# iOS Dev Workflow Plugin

A Claude Code plugin providing iOS development workflow tools including Jira ticket integration, branch code reviews, and iOS API research capabilities.

## Features

### Commands (User-Invokable Skills)

#### `/ios-dev-workflow:ticket`
Transforms a Jira ticket into a plannable feature with automated branch creation and implementation planning.

**Usage:**
```
/ios-dev-workflow:ticket PROJECT-123
/ios-dev-workflow:ticket https://your-domain.atlassian.net/browse/PROJECT-123
/ios-dev-workflow:ticket PROJECT-123 path/to/reference.md
```

**Features:**
- Parses Jira ticket IDs from URLs or direct input
- Fetches full ticket details via Jira MCP
- Creates properly named feature branches
- Spawns agent teams for feature planning
- Supports reference blueprint files

#### `/ios-dev-workflow:review-branch`
Comprehensive code review of the current branch against Jira ticket requirements.

**Usage:**
```
/ios-dev-workflow:review-branch
```

**Features:**
- Fetches diff against base branch (configurable, defaults to `origin/develop`)
- Validates acceptance criteria from Jira ticket
- Checks architecture compliance
- Verifies build success
- Reviews API correctness
- Generates detailed review markdown document

### Agents

#### `ios-api-researcher`
Elite iOS API researcher for finding optimal APIs, frameworks, and system capabilities.

**Use cases:**
- Finding modern replacements for deprecated APIs
- Discovering the best approach for new features
- Understanding API availability across iOS versions
- Comparing different Apple frameworks

**Invoked automatically** by other commands/agents when API research is needed.

#### `code-reviewer`
Expert code reviewer specializing in iOS/Swift code quality, security, and best practices.

**Capabilities:**
- Security vulnerability detection
- Performance analysis
- Architecture compliance checking
- Swift concurrency correctness
- SOLID principles enforcement

## Requirements

### MCP Servers
- **Jira MCP**: Required for ticket and review-branch commands
- **Context7 MCP**: Required for ios-api-researcher agent

### Project Configuration
For best results, your project should have:
- An `ARCHITECTURE.md` or similar architecture documentation
- A `CLAUDE.md` with project-specific instructions
- Git repository with a defined base branch (e.g., `develop` or `main`)

## Installation

### Local Testing
```bash
claude --plugin-dir ./ios-dev-workflow-plugin
```

### From Marketplace (when published)
```
/plugin install ios-dev-workflow
```

## Customization

### Adapting to Your Project

1. **Base Branch**: The commands default to `origin/develop`. Modify the commands if your main branch differs.

2. **Branch Naming**: The ticket command uses `feature/<TICKET-ID>-<description>` format. Adjust in `commands/ticket.md` if needed.

3. **Architecture Docs**: The agents reference architecture documentation. Ensure your project has equivalent docs or update the agent prompts.

4. **Minimum iOS Version**: The ios-api-researcher considers iOS 17.0+ by default. Adjust in the agent definition for different targets.

## Directory Structure

```
ios-dev-workflow-plugin/
├── .claude-plugin/
│   └── plugin.json          # Plugin manifest
├── commands/
│   ├── ticket.md            # Jira ticket workflow
│   └── review-branch.md     # Branch code review
├── agents/
│   ├── ios-api-researcher.md  # API research agent
│   └── code-reviewer.md       # Code review agent
└── README.md
```

## License

MIT
