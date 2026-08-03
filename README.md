# team-tools

A Claude Code plugin marketplace for the team.

## Installation

In Claude Code, add this marketplace, then install the plugin:

```
/plugin marketplace add ydikalchuk-lab/team-tools
/plugin install team-kit@team-tools
```

To pull in later updates:

```
/plugin marketplace update team-tools
```

## Plugins

| Plugin | Description |
|--------|-------------|
| [team-kit](./team-kit) | `commit-changes` and `prompt-improve` skills |

### team-kit

- **commit-changes** — commits working changes across the project's sibling git
  repos with clean Conventional Commits messages, refuses to bundle unrelated
  pre-existing work, and reports exactly what landed. Does not push.
- **prompt-improve** — turns a raw, vague task draft into a clear, structured
  prompt (role, context, task, constraints, response format) with a before/after.

See [team-kit/README.md](./team-kit/README.md) for details.

## Repository layout

```
team-tools/
├── .claude-plugin/
│   └── marketplace.json     # marketplace registry
└── team-kit/                # plugin
    ├── .claude-plugin/
    │   └── plugin.json
    ├── README.md
    └── skills/
        ├── commit-changes/
        └── prompt-improve/
```
