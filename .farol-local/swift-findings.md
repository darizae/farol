# Farol Swift and SwiftUI contribution findings

Personal contributor notes for `darizae/farol`, not an upstream source file. The notes branch is visible to others if the fork is public; do not put secrets or personal information here.

**Local location:** `<repo-root>/.farol-local/swift-findings.md`  
**Online backup:** fork-only `personal/farol-notes` branch  
**Used by:** `.agents/skills/farol-contributor/SKILL.md` and `.claude/skills/farol-contributor/SKILL.md`

Update this file only with observed, review-backed facts or clearly labeled hypotheses. Read it before every Farol task. Do **not** add this file to a feature branch or upstream PR.

## Confirmed lessons from PR #337, November 20, 2026

PR: https://github.com/snowztech/farol/pull/337  
Maintainer commit 1: https://github.com/snowztech/farol/commit/c2789935da817058bf430aa51d7f74ea01001343  
Maintainer commit 2: https://github.com/snowztech/farol/commit/989370310f75652d1f2e6a00948ea4c96b001f66

### 1. Model a transient selection separately from the active item

The original file-path PR stored a clicked row in `Files.Selection.paths` after an ordinary click. The row styling in `FileRow` used `marked || (active && !hasSelection)`, meaning any nonempty multi-selection suppressed the active-file highlight. Opening a different file through Find in Files or changing tabs left the previously clicked row highlighted; expanding a folder could also create stale selection highlighting.

**Maintainer's actual fix** in `Files.Selection.click`:

- An ordinary click makes `paths` empty while saving the clicked path as `anchor`.
- A Shift-click selects the visible range between `anchor` and the clicked item (or falls back to the clicked path if there is no valid anchor).
- A Command-click with empty selection first inserts the prior `anchor` if it is different, then toggles the clicked path, and updates `anchor`.
- A later plain click clears a multi-selection, so it is always possible to leave selection mode.

**Design rule:** Temporary selection sets, selection anchors, active/open items, and currently focused controls are different kinds of state. Name and test them separately. Verify the interaction sequence, not only a single isolated click. Check updates caused by search, tabs, keyboard navigation, and programmatic state changes as well as direct row interaction.

**Regression sequence worth testing:** Plain-click A; navigate externally to B; B should remain the active highlighted file, not A. Plain-click A; Shift-click D; range A–D should be selected. Plain-click D; selection should clear. Command-click B; D and B should enter selection together. Include folder expand/collapse and right-click context menu behavior in GUI checks.

### 2. Do not wrap an existing utility just to expose the same behavior

The PR added `Files.escapePathsForShell(_:)` solely to call `paths.map(NewTask.shellQuoted).joined(separator: " ")`. `NewTask.shellQuoted` already performed safe single-quote escaping, but its access level was internal.

**Maintainer's actual fix:** Remove `Files.escapePathsForShell`, make `NewTask.shellQuoted` public, and directly use `pathsToCopy.map(NewTask.shellQuoted).joined(separator: " ")` in the UI. Update the test to call the same existing helper directly. He removed the wrapper-only empty-input assertion.

**Design rule:** Search for existing helpers and their visibility before introducing new APIs. Prefer direct reuse and a justified access-level adjustment over a forwarding abstraction with no additional semantics. Do not create a second shell-quoting implementation.

### 3. Keep end-to-end regression checks for command behavior

The PR's test executed `/bin/sh` with `printf '%s\\0'` and compared decoded arguments to the original paths. The maintainer retained that test while replacing the wrapper call with `NewTask.shellQuoted`. This was an explicitly praised aspect of the PR.

**Design rule:** For shell quoting and subprocess behavior, test the actual process boundary and values with spaces, embedded quotes, and special characters. Unit tests for a string-construction helper alone do not establish that shell arguments survive round-tripping.

### 4. SwiftUI correctness depends on interactions across the feature

The first implementation was locally usable, but a cross-navigation state error remained. A passing test for `Files.Selection` did not prove that the file tree correctly highlighted the currently open file when another UI entry point changed it.

**Design rule:** For every new SwiftUI state variable, identify who owns it, what resets it, what causes a view update, and what depends on its emptiness. Check interaction sequences from outside the component. Test both selected and unselected states, transitions back to neutral, and behavior across navigation.

### 5. Avoid premature “done” claims and unrelated changes

Earlier iterations of this contribution had UI multiselect logic that was not fully wired, an initializer mismatch, redundant quoting, extraneous comments, and removal of an existing overwrite assertion. The maintainer's later review uncovered a distinct stale-highlight case.

**Process rule:** Build with the actual macOS toolchain; run `make test`, `make lint`, and `make run` after final edits; inspect changed files and removed tests; test the app interaction when GUI access exists. A tool invocation or successful build is not evidence of a manually verified UI flow. State limitations explicitly. Do not add inline comments or unrelated documentation for this user's feature branches.

## Applying these findings to upcoming work

### Issue #336 — Project groups versus sessions

**Confirmed source context:** `ProjectHeader` currently uses 12 pt muted text and session names use 13 pt. The project header also tracks `folded` and `holdsSelection`, and folded groups can display an aggregate activity lamp. `DESIGN.md` assigns prominent sidebar colors to agent status rather than project identity.

**Derived test requirement:** After changing header typography or grouping surfaces, check that project focus, the selected child session, the header's `holdsSelection` state, and hover emphasis remain visually distinct. Test selection driven by keyboard or navigation from another component, not only by clicks. A folder/group header must not acquire the session's active appearance accidentally. Keep collapsed groups with an active descendant legible, and preserve drag/drop, hover buttons, and dark/light themes.

**Implementation preference:** Small, theme-aware typography/spacing changes first; no extra persistent selection state or per-project color scheme. Use allowed font sizes from `scripts/check-style.py`.

### Issue #334 — Stale remote branches

**Confirmed source context:** `History.fetch` already invokes `git fetch --all --prune`, `History.branches` reads remote-tracking refs, and tests cover a successful prune. `History.deleteRemoteBranch` deletes a branch on a server via `git push ... --delete`. `GraphModel` has reload/snapshot/watch logic.

**Derived test requirement:** Compare the actual remote ref set to `GraphModel.remote` and displayed graph entries before and after fetch. Simulate a deleted branch using a **temporary bare remote**. Verify error/refresh transitions and switching roots; do not conflate a local remote-tracking ref with the server's real branch. If Git has already pruned the ref, don't assume another Git deletion command is needed. Preserve explicit confirmation before real server deletion.

**Implementation preference:** Diagnose source-of-truth versus cached view state and async reload ordering before making changes. Reuse existing History methods. Add a regression test for the actual failure and stop if it cannot be reproduced.

## Future review findings

Add dated entries with links to the exact upstream review, change, test, or commit. Clearly label what the maintainer said, what the patch actually changed, and which recommendations are our own inference. Update the fork-only `personal/farol-notes` branch, not a code-review branch.
