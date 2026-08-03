# team-kit

Team commit-changes and prompt-improve skills for Claude Code.

## Skills

### commit-changes

Commits the current working changes across this multi-repo project the way it is
done here. The project is split across several sibling git repositories
(`SXG-slingshot-common`, `SXG-content-model`, `SXG-slingshot-custom`, …), so
commits usually span more than one repo.

The skill:

- discovers which repos changed,
- commits each with a clean [Conventional Commits](https://www.conventionalcommits.org) message,
- refuses to silently bundle unrelated pre-existing changes,
- always reports the commit message plus the exact files committed,
- does **not** push.

Trigger it by asking to commit work — "commit", "commit my changes", "make
commits", "закоміть", "зроби коміти", "закоммить зміни".

### prompt-improve

Turns a raw, vague task draft ("почини логін", "зроби нормально", "оптимізуй
тут") into a clear, structured prompt with role, context, task, constraints, and
response format, and shows a before/after comparison.

It **does not execute** the task in the draft — it improves the wording so the
resulting prompt can be handed to an agent.

Argument: `[чернетка завдання]` — the draft task to improve.

## Installation

Add this directory as a Claude Code plugin, then invoke a skill by name (e.g.
`/commit-changes`) or let Claude trigger it automatically from the descriptions
above.

## Layout

```
team-kit/
├── .claude-plugin/
│   └── plugin.json
└── skills/
    ├── commit-changes/
    │   └── SKILL.md
    └── prompt-improve/
        └── SKILL.md
```
