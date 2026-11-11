# ticket-router

A Claude Code plugin that triages Jira tickets and delegates each one to the model and effort level that fit its complexity — so simple tickets don't burn Opus tokens, and hard ones don't get a shallow fix.

## How it works

```
/ticket-router:resolve-ticket PROJ-123
        │
        ▼
 Triage (Haiku)  ── fetches the ticket via the Atlassian MCP server
        │           and scans the repository for affected files
        ▼
 PROJ-123 → medium (baseline high): clear acceptance criteria, 2 files affected
        │
        ▼
 Subagent ticket-medium (Sonnet, medium effort) ── implements, runs tests, reports back
```

Only the subagent that matches the ticket does the heavy work. Triage itself runs on the cheapest model.

| Level      | Model  | Effort | Behavior |
|------------|--------|--------|----------|
| `low`      | Haiku  | low    | Smallest direct change, no refactoring |
| `medium`   | Sonnet | medium | Explores the flow, plans before editing |
| `high`     | Opus   | high   | Written plan, full test and lint pass |
| `critical` | Opus   | max    | Risk analysis, new tests, self-review of the diff |

## Triage criteria

The Jira priority sets a **baseline**:

| Jira priority                | Baseline   |
|------------------------------|------------|
| Lowest, Low                  | `low`      |
| Medium                       | `medium`   |
| High                         | `high`     |
| Highest, Critical, Blocker   | `critical` |

The baseline is then adjusted by **at most one level** based on:

- **Description** — clear with acceptance criteria lowers it; vague or open-ended raises it.
- **Type** — a bug with a stack trace or clear repro lowers it; a new feature or refactor raises it.
- **Size** — many acceptance criteria or high story points raise it.
- **Affected files** — 1–2 files lowers it; 6+ files or multiple modules raise it.
- **Risk floor** — tickets touching database migrations, authentication, payments or a public API are never `low`.

Priority measures urgency, not complexity — that's why the description and the codebase have the final word.

## Requirements

- [Claude Code](https://code.claude.com)
- A Jira Cloud site you can sign in to

## Installation

In a Claude Code session:

```
/plugin marketplace add iscarvalho/ticket-router
/plugin install ticket-router@italo-plugins
```

Restart the session after installing.

### First use: connect to Jira

The plugin registers the official [Atlassian MCP server](https://support.atlassian.com/atlassian-rovo-mcp-server/docs/getting-started-with-the-atlassian-remote-mcp-server/) automatically, but you need to sign in once:

1. Run `/mcp`
2. Select **atlassian** (listed under the plugin)
3. Sign in to Atlassian in your browser

If you skip this, the command stops and tells you to connect before doing anything.

## Usage

Open Claude Code **at the root of the repository the ticket belongs to**, then run:

```
/ticket-router:resolve-ticket PROJ-123
```

To skip triage and force a level:

```
/ticket-router:resolve-ticket PROJ-123 --level high
```

When the subagent finishes, you get a report with a summary, changed files, test results and open items. Review the diff before committing.

## Project structure

```
ticket-router/
├── .claude-plugin/
│   ├── plugin.json          # Plugin manifest
│   └── marketplace.json     # Makes this repo installable as a marketplace
├── .mcp.json                # Atlassian MCP server
├── agents/
│   ├── ticket-low.md
│   ├── ticket-medium.md
│   ├── ticket-high.md
│   └── ticket-critical.md
└── commands/
    └── resolve-ticket.md    # Triage and delegation
```

## Customizing

- **Change a level's model or effort** — edit the `model` and `effort` fields in the matching file under `agents/`.
- **Tune the triage** — edit the priority table and signals in `commands/resolve-ticket.md`.

## Local development

```bash
claude --plugin-dir /path/to/ticket-router
```

Restart the session after editing any plugin file. Use `claude --debug --plugin-dir ...` if the plugin doesn't load.

## License

MIT
