# Git Safety Guardian

[![skills.sh](https://skills.sh/b/3bsalam-1/git-safety-guardian)](https://skills.sh/3bsalam-1/git-safety-guardian)

A Claude skill that puts a human-approval gate in front of every
repository-changing git operation. It exists to stop accidental or
unauthorized commits, pushes, merges, rebases, force-pushes, branch deletions,
and other history-altering actions — the assistant must get explicit,
per-operation approval before any of them run.

## What it does

- Treats `commit`, `push`, `force-push`, `merge`, `rebase`, `cherry-pick`,
  `reset --hard`, `branch -D`, `tag`/`tag -d`, and `stash pop`/`apply` as
  **protected operations** that never run automatically.
- Requires a **triple-confirmation** workflow before a commit or push.
- Treats vague acknowledgement ("ok", "sure", "go ahead") as **not** approval.
- Spells out the consequences and demands extra care before a **force push**.
- Leaves finished work in a clean, reviewed, *uncommitted* state with a
  suggested message — and waits.

Read-only git (`status`, `diff`, `log`, `fetch`, …) is never gated.

## Why

Commits and pushes are the git actions easiest to fire reflexively and hardest
to undo. Once history leaves the local machine, "undo" usually means
force-pushing over other people's work. This skill trades a few seconds of
confirmation for protection against mistakes the human pays for.

## Install

Install with the [skills](https://skills.sh) CLI (works for Claude Code and
other supported agents):

```sh
npx skills add 3bsalam-1/git-safety-guardian
```

Or, to install globally for all your projects:

```sh
npx skills add 3bsalam-1/git-safety-guardian --global
```

Prefer to do it by hand? Copy the `git-safety-guardian/` folder into your Claude
skills directory. Either way, the skill activates automatically whenever a git
write operation is imminent.

## Contents

- `SKILL.md` — the skill definition (frontmatter + behavior).

## License

MIT
