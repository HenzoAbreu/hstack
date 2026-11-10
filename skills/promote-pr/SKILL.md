---
name: promote-pr
description: >-
  Promote a PR or change into another branch by cherry-picking it onto a fresh
  branch. Use when the user asks to promote a PR, commit, or change into main
  or another branch. Defaults to main.
---

# Promote PR

Promoting copies an existing change onto another branch without dragging along
whatever else lives on the source branch. Build a new branch from the target,
cherry-pick only the requested change, and open a PR into the target.

The target is `main` unless the user names another branch.

## Guardrails

- Fetch first. Work from the latest `origin/<target>`, never a stale local copy.
- Treat existing worktree changes as user-owned. Use a disposable worktree when
  the active one is dirty or in use.
- Promoting authorizes a new branch, cherry-picks, a push of that branch, and a
  PR into the target. It does not authorize merging, closing the source PR,
  force-pushing someone else's branch, or adding changes that were not asked
  for.
- Use the `leaving-pr-comments` skill for any comment posted on the user's
  behalf.

## Resolve the change

Turn the request into an exact list of commits, and record the source PR URL,
title, author, state, and base when there is one.

- **Merged PR:** use the commit it landed as on its base. Squash and rebase
  merges give you those commits directly. For a merge commit, cherry-pick it
  with `-m 1`.
- **Open PR:** use the PR's own commits in order. Skip merge commits that only
  pulled the base branch into it.
- **Commit, range, or branch:** use exactly what was named. For a branch, take
  the commits between its merge base with the source branch and its head.

Drop commits whose patch is already on the target. `git cherry origin/<target>
<commits>` shows them. If every commit is already there, say so and stop.

## Build the branch

1. Create `promote/<pr-number-or-short-sha>-to-<target>` from the fresh
   `origin/<target>`.
2. Cherry-pick each commit in order with `git cherry-pick -x` so every commit
   records where it came from. Git keeps the original author.
3. Resolve conflicts by keeping the change's intent while respecting how the
   target works today. If a conflict shows the change depends on code the
   target does not have, stop and report the missing dependency. Do not quietly
   promote that dependency too.
4. Read the final diff against `origin/<target>`. It should contain the
   promoted change and nothing else.
5. Run the repository's focused checks for the touched area. Separate real
   failures from pre-existing ones.

## Open the PR

Push the new branch and open a PR into the target with the `file-pr` skill.
Make the title say what is being promoted and where, following the repository's
conventions, for example `Promote #123 to release-2.4: fix login redirect`.
Link the source PR, credit its author, list the commits cherry-picked, and name
any conflicts you resolved.

Do not merge it, and leave the source PR alone unless the user asks otherwise.

## Report

Give the new PR link, the target, the commits promoted, anything skipped
because it was already on the target, conflicts and how you resolved them, and
check results.
