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
# Clone the plugin to a central location (one-time setup)
git clone git@github.com:JManke91/claude-code-ios-dev-workflow-plugin.git ~/.claude-plugins/ios-dev-workflow

# Use from any project
claude --plugin-dir ~/.claude-plugins/ios-dev-workflow

To update the plugin later:
cd ~/.claude-plugins/ios-dev-workflow && git pull
```

### Via plugin command
```
/plugin install https://github.com/JManke91/claude-code-ios-dev-workflow-plugin
```
This clones and registers the plugin automatically:
1. Claude code clones the repository to a cache directory (~/.claude/plugins/cache)
2. The plugin is registered in your settings (by default at user scope, meaning it works accross all projects)
3. Commands become immediately available with the namespace prefix (e.g. /ios-dev-workflow:ticket)

### Keeping the Plugin Updated

  After installation, enable auto-update to receive updates automatically:
  1. Run `/plugin`
  2. Go to **Marketplaces** tab
  3. Select **ios-dev-workflow**
  4. Choose **Enable auto-update**

Or manually update anytime with:
```
/plugin marketplace update ios-dev-workflow
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
