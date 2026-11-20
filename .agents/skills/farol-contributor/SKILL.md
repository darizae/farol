---
name: farol-contributor
description: Develop and verify focused Swift/macOS contributions to snowztech/farol using upstream conventions, personal Swift review findings, safe local Git branches, actual test results, and fork-only pushes without PR creation. Use whenever modifying Farol's Swift code, tests, Git graph, or sidebar UI.
---

# Farol contributor

Use this skill for every contribution to `snowztech/farol` from `darizae/farol`.

## Read these before doing any work

1. Confirm the repository root with `git rev-parse --show-toplevel` (normally `~/repos/farol`).
2. Read the current repository `AGENTS.md`, `CONTRIBUTING.md`, `DESIGN.md`, `ROADMAP.md`, and the issue with its comments. These are authoritative if they differ from this skill.
3. **Read the entire personal findings file at `<repo-root>/.farol-local/swift-findings.md` on every invocation.** It is linked relative to either installed skill location at [../../../.farol-local/swift-findings.md](../../../.farol-local/swift-findings.md). Do not merely assume it is loaded. If absent, stop implementation and restore it from the owner's downloaded file or `darizae/farol` branch `personal/farol-notes`; never silently invent its contents.
4. Inspect the affected source, call sites, UI state, asynchronous flows, helpers, existing tests, and nearby design patterns. Read them before proposing a change.

## Branches, permissions, and private knowledge

- Confirm `origin` is `darizae/farol`, and `upstream` is `snowztech/farol`. Never push to `upstream`.
- Begin each issue from a freshly fetched `upstream/main`: `git fetch upstream main`. The maintainer's post-review fixes to PR #337 are already merged, so **do not** branch from the old `feat/copy-file-paths` or stale fork `main`.
- One issue per branch. Inspect `git status --short` before switching; do not stash, reset, overwrite, cherry-pick, rebase or force-push another person's work to get around a dirty checkout.
- The installed skill files `.agents/skills/farol-contributor/SKILL.md` and `.claude/skills/farol-contributor/SKILL.md` and the personal file `.farol-local/swift-findings.md` are **local-only on code branches**. Ignore them using this checkout's `$(git rev-parse --git-path info/exclude)`, not a tracked `.gitignore`. Before any feature commit verify none appears in `git diff --cached --name-only`.
- To back up or update the notes online, use a **dedicated fork-only branch** named `personal/farol-notes` in `darizae/farol`. Put the notes and skill in that branch only, never in an issue branch or upstream PR. Prefer a separate, temporary Git repository or worktree to publish it, with exact branch and remote verification. If it does not exist, create it as an orphan branch so it contains only these personal contribution documents. If it exists, update it normally without rewriting history. Never make or merge a PR from this branch.
- These are distinct requirements: **ignored/untracked in a feature checkout** and **tracked only in a separate fork branch**. A file cannot be simultaneously untracked and backed up in the same Git branch.
- You may edit, test, make focused commits and push successful issue branches to `origin`. **Never create a PR (even a draft), merge, post GitHub comments, or push a feature to `upstream`.** The user reviews code before PR creation.

## Diagnose, change, and self-review

1. Write down the intended behavior, the current observed behavior, and the smallest solution. For a bug, reproduce with evidence before modifying deletion semantics or inventing a fix.
2. Identify the *source of truth* for every state variable. Distinguish persistent application state, temporary multi-selection, focused/active item, and cached/snapshot state. Check how all entry points update that state (click, modifier-click, keyboard shortcuts, search, tab switching, folder collapse/expand, repository change).
3. Prefer existing helpers over forwarding wrappers. Check existing utilities and access levels before creating a new abstraction. A narrow visibility change is better than a redundant public wrapper when appropriate.
4. Follow current Swift/SwiftUI API patterns, property wrappers, modifier order, initializer signatures, concurrency, UI responsiveness and the `Palette` design system. Avoid expensive per-row computations and extra subprocesses on the main thread.
5. Keep the patch small. Preserve unrelated files, tests, comments, APIs, gestures and the user's work. Do not add new inline comments or explanatory documentation comments unless the user explicitly authorizes them. Do not create summary or PR-description files in the repository.
6. Test both a feature's positive behavior and its **negative/interleaved transitions**. A component can pass unit tests and still have the wrong UI behavior after another entry point acts. Where possible, add a regression test that fails before and passes after the fix.
7. For process quoting, Git and shell code, test real command behavior in disposable environments. Never perform destructive branch tests on a real remote. Reuse `NewTask.shellQuoted` for existing shell quoting rather than recreating it.
8. For user-visible changes, add one concise bullet under `Unreleased` in `CHANGELOG.md`. Follow `DESIGN.md` for UI: only approved type sizes, theme-relative neutral colors in the sidebar, and intact agent-status lamp meaning.

## Required actual verification

- After **the final edit**, run `make test` and `make lint`. Investigate failures rather than calling them passes. Run `git diff --check` and `git diff --cached --check`, and inspect `git status --short`, `git diff`, `git diff --cached`, and the comparison to `upstream/main` before staging or committing.
- For app changes, quit the running development app and rebuild:

  ```sh
  osascript -e 'tell application id "dev.farol.Farol.debug" to quit' 2>/dev/null || true
  make run
  ```

  A successful `make run` proves build and launch only, not that the interaction was tested. Verify the affected UI manually when GUI control exists. Otherwise state clearly which manual Mac checks the user must perform. A restarted instance is essential; `open` can activate an older build.
- For sidebar changes, check at least two projects, expanded and collapsed, multiple sessions, light and dark themes, narrow sidebar/long names, session selection via clicks **and external navigation**, drag/drop and hover. Capture before/after screenshots outside the repo when possible.
- For Git-graph changes, reproduce with temporary local/bare Git repositories, distinguish `refs/remotes` from actual server branches, and verify graph state after fetch and other refresh paths. Use the Farol Inspector (`Option-Command-I`) for Git calls and responsiveness if accessible.
- Review staged and committed diffs for unintended regressions or removed assertions. Verify each new test tests behavior, not just a helper that happens to exist. Run `git diff --check upstream/main...HEAD` after committing.
- Do not assert success for an unperformed check or a UI action not actually exercised.

## Commit, push, stop

- If a supported fix or improvement passes required checks, make focused one-line Conventional Commit(s), no body or co-author trailer. Push **the issue branch only** to `origin` using `git push -u origin <issue-branch>`; verify the remote and commit SHA. Never commit private notes/skill files in the issue branch.
- If a bug cannot be reproduced or a safe solution cannot be validated, report the evidence and stop without inventing a change or a speculative commit.
- Final handoff: issue number and branch, root cause or design rationale, changes made, regression coverage, commands and *actual* pass/fail outputs, UI testing performed versus not performed, remaining concerns, commit SHA, branch URL, and confirmation **“No PR created.”**

## Issue emphasis

### Issue #336: project/session hierarchy

Look at `SidebarView`, `ProjectHeader`, `SessionRow`, collapsed headers, and `holdsSelection`. The current concern is the parent/child hierarchy: muted 12 pt project labels versus 13 pt sessions. Follow the typography scale (13 and 15 are allowed; 14 is not), existing neutral theme colors, lamp semantics, and hover/disclosure conventions. Check that cosmetic changes cannot make an unrelated session look active or hide the active descendant when a group is folded.

### Issue #334: stale remote-tracking branches

`History.fetch` already executes `git fetch --all --prune`, `History.branches` reads `refs/remotes`, and tests already cover a successful prune. `History.deleteRemoteBranch` pushes a server-side deletion and is not an alternative to local pruning. Distinguish Git operation failures, successful pruning with stale `GraphModel` UI state, root/snapshot changes, and discoverability of the fetch action. Do not perform server deletion without an explicit confirmation. A no-op after successful reproduction-free diagnosis is better than a speculative implementation.
