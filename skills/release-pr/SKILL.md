---
name: release-pr
description: >-
  Combine several PRs or changes into one release PR with a changelog, then
  close the PRs it replaces. Use when the user asks for a release PR, or to
  group, bundle, or release a list of PRs or changes together.
---

# Release PR

A release PR replaces a list of PRs or changes with one PR into the target.
It carries a changelog, gets a real title, and closes the PRs it replaces.
Tests run after it is open, not before.

The target is `main` unless the user names another branch.

## Guardrails

- Fetch first. Work from the latest `origin/<target>`.
- Treat existing worktree changes as user-owned. Use a disposable worktree when
  the active one is dirty or in use.
- Asking for a release PR authorizes a new branch, cherry-picks, a push of that
  branch, the release PR, and closing the open PRs it replaces. It does not
  authorize merging the release PR, closing PRs the user did not list, or
  deleting their branches.
- Do not run the test suite before the release PR is open. Resolving conflicts
  is fine. Testing waits until the PR exists.
- Use the `leaving-pr-comments` skill for every comment posted on the user's
  behalf.

## Resolve the list

For each item, record its URL, title, author, state, labels, and the commits
that make it up. Use the same rules as `promote-pr`: a merged PR's landed
commit, an open PR's own commits without base merges, or the exact commits
named.

Keep the order the user gave. When they gave no order, use merge time for
merged PRs and then creation time for open ones. Drop commits already on the
target with `git cherry`, and note those items as already released.

If an item can't be found, or is closed without merging, stop and ask instead
of guessing.

## Build the branch

1. Create `release/<version-or-date>` from the fresh `origin/<target>`. Use a
   version only when the user or the repository's release convention gives
   one. Otherwise use the date, for example `release/2026-11-10`.
2. Cherry-pick every commit, item by item, with `git cherry-pick -x`. Git keeps
   each original author.
3. Resolve conflicts by keeping both changes' intent. If two items truly
   disagree, or one depends on code that is neither on the target nor in the
   list, stop and report it.
4. Read the final diff against `origin/<target>`. It should contain the listed
   changes and nothing else.

## Write the title and changelog

The title follows the repository's conventions and says what ships, for example
`release: 2.4.0` or `release: faster sync and login fixes`. Lead with the
version when there is one.

Open the description with one or two plain sentences on what the release
delivers. Then add the changelog. Group entries by kind and skip empty
groups. One line per item, written for a reader rather than copied from a
commit subject, ending with the source PR link and author:

```markdown
## Changelog

### Features
- Sync finishes in half the time on large workspaces (#412, @alice)

### Fixes
- Login no longer loops after a session expires (#418, @bob)

### Other
- Bump the Node runtime to 22 (#420, @carol)
```

List already-released items under a short "Already on <target>" note. Name any
conflicts you resolved. End with the model and harness line from `file-pr`.

## Open it, close the old PRs, then test

1. Push the branch and open the release PR into the target. Follow `file-pr`
   for the title rules, the secrets check, and opening a real PR rather than a
   draft. The changelog replaces the usual problem-first description.
2. Close each listed PR that is still open. Leave a short comment that links
   the release PR, for example `Shipping in #430.` Merged PRs can't be closed,
   so leave them alone. Delete no branches.
3. Now run the tests. Run the repository's test suite on the release branch
   and watch the PR's CI. For a failure, decide whether it comes from the
   release or was already there. Fix release failures on the release branch,
   push, and run again. If the user asked you to babysit it, continue with
   `babysit-pr`.

Do not merge the release PR.

## Report

Give the release PR link and title, the PRs it includes and which ones you
closed, items skipped as already released, conflicts and how you resolved
them, and test and CI results, including what failed and what you fixed.
