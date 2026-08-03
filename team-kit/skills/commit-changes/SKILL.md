---
name: commit-changes
description: >-
  Commit the current working changes across this multi-repo project the way it
  is done here. Trigger whenever the user asks to commit work — "commit", "commit
  my changes", "make commits", "закоміть", "зроби коміти", "закоммить зміни" — even
  if they don't name a specific file or repo. The project is split across several
  sibling git repositories (SXG-slingshot-common, SXG-content-model,
  SXG-slingshot-custom, …), so commits usually span more than one repo. This skill
  discovers which repos changed, commits each with a clean Conventional Commits
  message, refuses to silently bundle unrelated pre-existing changes, and always
  reports the commit message plus the exact files committed. It does not push.
allowed-tools:
  - Bash(git status *)
  - Bash(git -C * status *)
  - Bash(git diff *)
  - Bash(git -C * diff *)
  - Bash(git log *)
  - Bash(git -C * log *)
  - Bash(git add *)
  - Bash(git -C * add *)
  - Bash(git commit *)
  - Bash(git -C * commit *)
  - Bash(git show *)
  - Bash(git -C * show *)
  - Bash(git rev-parse *)
  - Bash(git -C * rev-parse *)
  - Bash(git reset --soft *)
  - Bash(git -C * reset --soft *)
---

# Commit changes

The goal is reliable, honest commits across this project's several git repos,
with a clear report back to the user of *what message* and *which files* landed.

Two things make this project different from a single-repo commit:
1. Work often touches **multiple sibling repos** under the project's parent
   directory (e.g. `SXG-slingshot-common`, `SXG-content-model`,
   `SXG-slingshot-custom`). Each is its own repo with its own branch and status.
2. Those repos frequently carry **pre-existing uncommitted work that is not
   yours** (large model rewrites, whole-file staged adds). Sweeping that into a
   commit misrepresents the changeset — so surface it, don't bundle it blindly.

## Workflow

### 1. Find the repos with changes

The files you edited this session tell you which repos are involved. For each
candidate repo, check its branch and status:

```
git -C <repo> rev-parse --abbrev-ref HEAD
git -C <repo> status --short
```

(If `git -C` is blocked in this environment, `cd` into the repo instead.)

If a repo is on the default/main branch (`main`, `master`, `SXG_3.4.2`, …),
create a working branch before committing, per the usual git rules. Repos here
are normally already on a dev branch (e.g. `Cofco_DEV_Dykalchuk`).

### 2. Read each repo's diff before staging

Read what actually changed — enough to describe *intent*, not just mechanics:

```
git -C <repo> diff --stat
git -C <repo> diff -- <path>
git -C <repo> log --oneline -15     # match the repo's existing message style
```

While reading, **scan the diff for secrets** — API keys, tokens, passwords,
private keys, `.env` values. If you find one, do not stage or commit it: stop,
point at the file, and let the user remove it or confirm a false positive.

Stage **only the files that belong to the change you were asked to commit**, by
path (`git -C <repo> add <path> …`) — never `git add -A` / `git add .`, so
unrelated work is never swept in. If the user already staged exactly what they
want, respect it. If a repo also holds changes that are large, that you did not
make, or that look like someone else's in-progress work (thousands of reordered
lines, a file staged as a brand-new whole-file add), **stop and ask the user**
how to handle that repo instead of including it. Confirming is cheap; committing
foreign WIP under your message is hard to undo and misleading.

### 3. Write the message, commit with `-F`

Write **one logical commit per repo** in Conventional Commits form (see below).
If several unrelated concerns are mixed in one repo, say so and suggest splitting
rather than lumping them.

Pass the message via a **temp file**, not inline. The Bash tool runs Git Bash
while the shell is PowerShell-flavored, and inline heredocs (`@'...'@`, `<<EOF`)
can get mangled — you can end up with stray characters in the subject. A message
file plus `git commit -F` is the one reliable way:

1. Write the message to the scratchpad, e.g. `<scratchpad>/commit-msg-<repo>.txt`.
2. Optionally validate its shape with the sibling skill's checker:
   ```bash
   bash ~/.claude/skills/git-commit/scripts/validate-commit-msg.sh < <scratchpad>/commit-msg-<repo>.txt
   ```
   It prints `ERROR:` and exits non-zero on a hard violation (unknown type, bad
   structure, trailing period, subject > 72, missing blank line before body). Fix
   and re-run until it passes. Form only — you still own whether it's *accurate*.
3. Stage the intended paths, then commit:
   ```bash
   git -C <repo> add <path> …
   git -C <repo> commit -F <scratchpad>/commit-msg-<repo>.txt
   ```

If a commit lands with a broken message, `--amend` is blocked by a hook here, so
undo and redo the *just-created, unpushed* commit —
`git -C <repo> reset --soft HEAD~1`, then commit again with `-F`.

### 4. Report back — the required output

This report is the whole point of the skill — never skip it. After committing,
for **each** repo gather and present the facts:

```
git -C <repo> rev-parse HEAD                                   # hash
git -C <repo> log -1 --pretty=%B                               # full message
git -C <repo> show --stat --name-only --pretty=format: HEAD    # committed files
```

Present per repo, compact, e.g.:

```
SXG-slingshot-common → 711ae280
  feat(history): add missing sxcfc/sx property labels
  files:
    web/.../messages/cofco-history_uk.properties
    web/.../messages/documentHistory.properties
```

### 5. Do not push

Stop after committing. Only run `git push` if the user explicitly asks.

## Conventional Commits format

```
<type>(<optional scope>): <subject>

<optional body — the *why*, wrapped ~72 cols>
```

English, imperative, lowercase subject, no trailing period, ≤ 50 chars ideally
(hard cap ~72). Infer **scope** from the diff paths (`history`, `model`,
`labels`, `ui`); omit when no single scope fits. Add a **body** only when the
motivation or a tradeoff isn't obvious from the subject.

| type | when |
|------|------|
| `feat` | new feature / user-facing capability |
| `fix` | bug fix |
| `docs` | documentation only |
| `style` | formatting/whitespace, no logic change |
| `refactor` | neither fixes a bug nor adds a feature |
| `perf` | performance improvement |
| `test` | adding or fixing tests |
| `build` | build system, dependencies |
| `ci` | CI configuration and scripts |
| `chore` | maintenance/tooling not touching src or tests |
| `revert` | reverting a previous commit |

**Example — localization fix, no body:**
```
fix(labels): correct "Дотаткові" typo in card-labels
```

**Example — feature with body:**
```
feat(history): add missing sxcfc property labels

Card properties were shown as raw keys (sxg-custom-history.sxcfc_eStampCOFCO)
in the action history; add the missing labels sourced from the content model.
```

## When to stop (no commit)

Stop before committing, don't retry silently, and explain plainly:
- **Nothing to commit** — no changes in a repo; say so rather than an empty commit.
- **A secret is in the diff** — point at the file; wait for the user.
- **A hook rejected the commit** — report the hook output; the commit did not
  happen. Don't bypass with `--no-verify` unless the user explicitly asks.

## Guardrails & notes

- Stays local and non-destructive: no `git push`, no `--amend`, no `--no-verify`,
  no git-config changes — each is a separate, explicit user decision. (The
  `reset --soft` message-fix above is the one narrow exception, and only on a
  commit you just created this run and haven't pushed.)
- Never invent changes that aren't in the diff.
- One logical commit per repo per invocation.
- Don't add `Co-Authored-By` or tool attribution unless the user asks for it.
