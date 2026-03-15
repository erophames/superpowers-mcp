# superpowers-mcp

An MCP server that makes [superpowers](https://github.com/obra/superpowers) skills available to any LLM that supports the [Model Context Protocol](https://modelcontextprotocol.io).

Superpowers is a skills library for Claude Code that enforces disciplined workflows -- TDD, systematic debugging, brainstorming, planning, and more. This server exposes those skills to any MCP-compatible client: Cursor, Windsurf, Gemini, Kilo Code, Claude Desktop, or your own app.

## Quick Start

### 1. Clone and build

```bash
git clone https://github.com/erophames/superpowers-mcp.git
cd superpowers-mcp
npm install
npm run build
```

### 2. Run setup

```bash
node build/index.js
```

On first run in a terminal, the setup wizard clones the superpowers repository and saves the configuration.

### 3. Configure your MCP client

Add to your client's MCP server configuration:

```json
{
  "mcpServers": {
    "superpowers": {
      "command": "node",
      "args": ["/absolute/path/to/superpowers-mcp/build/index.js"]
    }
  }
}
```

Replace `/absolute/path/to/` with the actual path where you cloned the repository.

<details>
<summary>Client-specific config file locations</summary>

| Client | Config Location |
|--------|----------------|
| Claude Desktop | `~/Library/Application Support/Claude/claude_desktop_config.json` |
| Claude Code | `~/.claude.json` or project `.mcp.json` |
| Cursor | Cursor Settings > MCP |
| Windsurf | `~/.codeium/windsurf/mcp_config.json` |

</details>

## What It Exposes

The server registers skills through all three MCP primitives:

### Tools

| Tool | Description |
|------|-------------|
| `list_skills` | List all available skills with descriptions and file lists |
| `use_skill` | Load a skill by name -- returns the full skill content as instructions to follow (optional guardrail enforcement) |
| `get_skill_file` | Load a supporting file from a skill (reference docs, prompt templates, scripts) |
| `recommend_skills` | Recommend top skills for a task using semantic ranking + workflow policy boosts |
| `compose_workflow` | Build an ordered multi-skill workflow for a goal |
| `validate_workflow` | Validate selected skills against required workflow guardrails |
| `semantic_search_skills` | Semantic search across `SKILL.md` and supporting files |

### Prompts

Each skill is registered as an MCP prompt named `superpowers:{skill-name}`. Clients that support prompt selection will show these in their UI. Selecting a prompt injects the full skill content into the conversation.

### Resources

Each skill's `SKILL.md` and supporting files are registered as resources:

```
superpowers://skills/brainstorming/SKILL.md
superpowers://skills/test-driven-development/testing-anti-patterns.md
superpowers://skills/systematic-debugging/root-cause-tracing.md
```

A resource template `superpowers://skills/{skillName}/{fileName}` enables dynamic access to any file.

## Available Skills

| Skill | What it does |
|-------|-------------|
| `brainstorming` | Collaborative design through questions, approaches, and incremental validation |
| `writing-plans` | Bite-sized implementation plans with exact file paths and TDD steps |
| `executing-plans` | Batch execution of plans with review checkpoints between tasks |
| `subagent-driven-development` | Fresh subagent per task with two-stage code review |
| `dispatching-parallel-agents` | Distribute independent tasks to concurrent agents |
| `test-driven-development` | RED-GREEN-REFACTOR cycle -- write failing test first, always |
| `systematic-debugging` | 4-phase root cause analysis: investigate before you fix |
| `verification-before-completion` | Run the command, read the output, then claim the result |
| `requesting-code-review` | Dispatch code review and act on severity-categorized feedback |
| `receiving-code-review` | Technical rigor when receiving feedback -- verify, don't blindly agree |
| `using-git-worktrees` | Isolated git worktrees for parallel feature development |
| `finishing-a-development-branch` | Structured options for merge, PR, or cleanup when done |
| `writing-skills` | Framework for creating, testing, and deploying new skills |
| `using-superpowers` | How the skill system works and when to invoke skills |

## Usage Examples

Once connected, ask your AI assistant:

- "What superpowers skills are available?" (calls `list_skills`)
- "Use the brainstorming skill to help me design a caching layer."
- "Load the TDD skill and follow it to implement this feature."
- "Read the anti-patterns file from the test-driven-development skill."
- "Recommend the best skills for implementing this MCP feature."
- "Compose a workflow for debugging flaky tests."
- "Validate this workflow: brainstorming, writing-plans, test-driven-development."
- "Search skills for root cause tracing techniques."

## Configuration

### Environment Variables

| Variable | Description |
|----------|-------------|
| `SUPERPOWERS_SKILLS_DIR` | Override the skills directory path directly |

### Skill Discovery Order

1. `SUPERPOWERS_SKILLS_DIR` environment variable
2. Saved directory from the setup wizard (persisted in `~/.config/superpowers-mcp`)
3. Claude Code plugin cache (`~/.claude/plugins/cache/claude-plugins-official/superpowers/*/skills/`)

### Auto-Updates

When skills are sourced from a git repository, the server checks for updates once per day on startup and pulls new changes automatically.

## Development

```bash
npm install
npm run build
npm test
```

| Script | Description |
|--------|-------------|
| `npm run build` | Compile TypeScript |
| `npm run dev` | Watch mode compilation |
| `npm test` | Run all 58 tests |
| `npm run test:watch` | Watch mode tests |
| `npm start` | Run the server |

### Architecture

```
src/
  index.ts                  Entry point, stdio transport, setup and update orchestration
  server.ts                 McpServer creation, composes all registrations
  config.ts                 Persistent configuration
  update.ts                 Daily auto-update check
  git.ts                    Git operations (clone, pull, fetch) via execFile
  cli/setup.ts              Interactive setup wizard
  skills/
    types.ts                Skill, SkillFile, SkillMetadata interfaces
    discovery.ts            Directory scanning, YAML frontmatter parsing, version resolution
  tools/register.ts         list_skills, use_skill, get_skill_file
  prompts/register.ts       One MCP prompt per skill
  resources/register.ts     Static resources per file + resource template for dynamic access
```
### Tests

[![Node.js CI](https://github.com/erophames/superpowers-mcp/actions/workflows/node.js.yml/badge.svg)](https://github.com/erophames/superpowers-mcp/actions/workflows/node.js.yml)[![Node.js CI](https://github.com/erophames/superpowers-mcp/actions/workflows/node.js.yml/badge.svg)](https://github.com/erophames/superpowers-mcp/actions/workflows/node.js.yml)

## License

MIT
