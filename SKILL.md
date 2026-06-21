---
name: git-safety-guardian
description: >-
  Enforces a human-approval gate before any repository-changing git operation —
  commit, commit --amend, push, force-push, merge, rebase, cherry-pick,
  reset --hard, branch -D, tag / tag -d, and stash pop / apply. Use this skill
  whenever you are about to run a git write command, the moment a coding task
  wraps up and a commit feels natural, when the user says things like "commit",
  "push", "merge", "ship it", "save my work", or right after tests pass — any
  moment Claude might be tempted to commit or push on its own. It requires
  explicit, unambiguous, per-operation approval and treats silence or a vague
  "ok / sure / proceed" as a refusal, not a yes. Trigger it proactively at the
  end of coding work even if the user never mentions git; stopping accidental or
  unauthorized git writes is its entire job. Read-only git (status, diff, log)
  is never blocked.
---

# Git Safety Guardian

## Why this skill exists

Commits and pushes are the two git actions that are easiest to fire reflexively
and hardest to walk back. A commit silently bakes in whatever is staged —
including half-finished work, debug prints, or secrets. A push hands that state
to a shared remote where teammates, CI, and sometimes production immediately act
on it. Once history leaves the local machine, "undo" means force-pushing over
other people's work, which is its own hazard.

So the governing assumption here is humility about state:

- The code may contain bugs the tests didn't catch.
- The tests may be incomplete or may not have run.
- The user may not intend to commit *everything* that's currently changed.
- A push may touch production or a branch other people depend on.
- **Silence is not approval.** Neither is a task feeling "done."

This skill outranks convenience, speed, and workflow tidiness. It is better to
stop and ask than to guess and be wrong, because the cost of a wrong git write
is paid by the human, often in a different and more painful currency than the
few seconds an extra confirmation costs.

## What is protected

Treat every one of these as a **protected operation** — a git command that
changes the repository or its history and therefore needs the approval workflow
below before it runs:

- `git commit`, `git commit --amend`
- `git push`, `git push --force`, `git push --force-with-lease`
- `git merge`
- `git rebase`
- `git cherry-pick`
- `git reset --hard`
- `git branch -D` (force-delete a branch)
- `git tag`, `git tag -d`
- `git stash pop`, `git stash apply`

Never run a protected operation automatically — not as a "finishing touch," not
because it seems obviously what the user wants, not because a similar action was
approved earlier. Each one waits for the workflow.

## What is always fine (do not gate these)

Read-only and inspection commands never change history and must **not** be
blocked or wrapped in confirmations — gating them would make the assistant
useless and train the user to rubber-stamp prompts. Run these freely:

`git status`, `git diff` (any form), `git log`, `git show`, `git branch`
(listing), `git remote -v`, `git stash list`, `git fetch`, `git blame`,
`git describe`, `git rev-parse`, and similar read-only queries.

Staging (`git add`) and creating a branch (`git switch -c` / `git checkout -b`)
are low-risk and don't need the full gate — but they are often the lead-in to a
commit, so once something is staged, hold at the commit boundary.

## The core rule

**Every protected operation requires a fresh, explicit, per-operation approval
from the user.** Approval for one commit is not approval for the next one.
Approval given in an earlier conversation, or for a similar-looking change, does
not carry over. Each protected action starts its approval sequence from zero.

---

## Commit approval workflow

Before any commit, walk these four steps in order. Each gate is a real stop —
present the information, ask the question, and wait for the user's reply before
moving on.

### Step 1 — Review

Show the user what they're about to commit so the decision is informed, not
blind:

- Modified files
- New (untracked) files that would be included
- Deleted files
- A short, high-level summary of what changed and why
- Test status (did tests run? did they pass? or were they not run — say so)

### Step 2 — Commit proposal

Propose, don't impose:

- A proposed commit **title**
- A proposed commit **description** (body), if the change warrants one

Then ask:

> "Do you want to create this commit?"

Wait for explicit approval.

### Step 3 — Message confirmation

Ask:

> "Please confirm the commit message exactly as written."

Wait for explicit approval. This second gate exists because the *intent* to
commit and the *exact wording* that lands in permanent history are two separate
decisions — the user may approve committing but want to reword.

### Step 4 — Final confirmation

Ask:

> "Final confirmation: execute git commit now?"

Wait for explicit approval.

Only after all three approvals (Steps 2, 3, and 4) may the commit run.

---

## Push approval workflow

A push is higher-stakes than a commit because it leaves the local machine. Walk
these three gates before any push.

### Step 1 — Push review

Present:

- The current branch
- The remote target (e.g. `origin`) and the branch it pushes to
- The exact commits that would be pushed (list them)

Then ask:

> "Do you want to push these commits?"

Wait for approval.

### Step 2 — Branch confirmation

Ask:

> "Please confirm the target branch."

Wait for approval. Pushing to the wrong branch — especially a default/protected
branch — is a common and costly mistake, so the destination gets its own gate.

### Step 3 — Final confirmation

Ask:

> "Final confirmation: execute git push now?"

Wait for approval.

Only after all three approvals may the push run.

---

## Force push — critical risk

`git push --force` and `--force-with-lease` can **erase commits that already
exist on the remote**, including work pushed by other people. This is the single
most destructive routine git operation, so it gets the strongest treatment.

Before a force push, first **explain the consequences in plain terms**:

- What history will be overwritten and what will be lost
- Which branch is affected and who else may depend on it
- That recovering the overwritten commits may be difficult or impossible

Then require **three separate confirmations**, the same shape as a normal push
but with the consequences spelled out first.

If anything is unclear — you're not certain which branch is targeted, whether
the remote has commits you'd clobber, or whether the user grasps the impact —
**refuse the force push** and say why. A refusal here is a feature, not a
failure.

---

## What does NOT count as approval

The whole skill collapses if vague acknowledgement is read as a green light.
These are **not** approvals — they're the user nodding along, not authorizing a
specific irreversible action:

> ok · sure · continue · proceed · do it · looks good · go ahead · fine · yep

They're ambiguous because they don't name the operation and often just
acknowledge what you said. When you get one of these in place of a clear yes,
treat it as **not yet approved**: restate exactly what you're about to do and
ask the specific question again.

If there is any uncertainty about whether the user truly approved *this*
operation: **do not commit, do not push.** Ask. The downside of asking once more
is seconds; the downside of guessing wrong is the user's time, history, or
production state.

## What DOES count as approval

So the skill doesn't become paralysis, here's what a real approval looks like —
an unambiguous instruction tied to the specific operation in front of you:

- "Yes, create that commit."
- "Confirmed, use that message."
- "Yes, push to main now."
- "Go ahead and force-push my feature branch — I understand it overwrites the
  remote."

The test: would a reasonable person read this as the user authorizing *this
exact action*, knowing what it does? If yes, proceed. If it's a generic "ok"
that could just mean "I heard you," it's Step-1 territory again.

---

## Never auto-perform these

Do not commit:

- because a task finished
- because tests passed
- because a previous conversation seemed to imply approval
- because a similar commit was approved before

Do not push:

- because you just committed
- because a merge or deploy completed
- because the branch was approved for a push earlier

Every commit and every push starts a new approval sequence. No exceptions, no
inherited permissions.

## Preferred end-of-task behavior

When coding work is done, the natural-feeling next move is to commit and push.
Resist it. Instead:

1. **Stop.**
2. Summarize what you did.
3. Show the changed files.
4. Suggest a commit message.
5. Wait for instructions.

Leaving the repository in a clean, reviewed, *uncommitted* state is the correct
finished position. Handing the user a clear summary and a ready-to-go message —
and then waiting — respects that the commit and the push are theirs to make.
