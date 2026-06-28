---
name: git-workflow
description: MUST USE when making significant or multi-step code changes. Guides incremental commits to a dedicated feature branch with descriptive messages. Never pushes to remote.
---

# Git Workflow for Significant Changes

Track substantial or multi-file changes with Git to enable easy review, iteration, and reversion.

**Note:** Git commands run on the host filesystem, not inside Docker containers.

## Preserve Existing Work

Before starting:

```bash
git status
```

- If uncommitted changes exist, create a snapshot commit to preserve them **without removing them from the working tree**:
  ```bash
  git add -A
  git commit -m "WIP: snapshot of current work before AI changes"
  ```
  Then create your AI branch from this commit.

**Why a commit instead of a stash?** Stashing removes changes from the working tree, meaning the AI would see outdated code and potentially overwrite the user's latest work. A snapshot commit keeps the latest code visible and intact while providing a safe restore point.

**Inform the user:** After creating a snapshot commit, explicitly tell the user about it:
> "To preserve your uncommitted changes before starting this task, I've created a snapshot commit: `abc1234` ('WIP: snapshot of current work before AI changes')."

## Use a Feature Branch

For non-trivial work, create a branch:

```sh
git checkout -b ai/<descriptive-name>
```

## Commit Incrementally

Commit logical chunks as you progress:

```sh
git add <files>
git commit -m "<concise description of the change>"
```

- Explain the **why**, not just the what
- Keep subject lines under 72 characters
- Use present tense, imperative mood (e.g., "Add validation to user routes")

## Critical Guardrail: No Remote Pushes

**Never push to remote under any circumstances.** This includes `git push`, `git push --force`, tags, or any automated push. All commits remain local for developer review.
