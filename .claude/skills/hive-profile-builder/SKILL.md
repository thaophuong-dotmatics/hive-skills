---
name: hive-profile-builder
description: "Thao's hive workflow — project setup: scan the repo and write PROJECT-PROFILE.md at the repo root, a one-screen router the hive skills read for docs, commands, app access, review lenses, and tracker info."
disable-model-invocation: true
---

# Hive Profile Builder

Scan the repo. Write `PROJECT-PROFILE.md` at the repo root — committed, not gitignored.

A **router, not a new CLAUDE.md**:
- Existing doc answers a question → link it.
- No doc covers it → inline (typically app launch and login).

**Already exists?** Refresh in place:
- Re-verify every pointer, update stale ones.
- **Manual content** = text inside the five existing sections that isn't one of the required pointers/fields — keep it verbatim unless the field it's attached to needs refreshing. Never create or preserve an extra top-level `##` heading beyond the five-entry contract; a legacy stray heading's text moves under whichever entry it belongs to, unchanged, rather than staying a sixth heading.
- Never duplicate or append a second copy.

**Re-run when:** docs reorganize, or a hive skill hits a stale pointer. `/hive-master` kickoff is a natural checkpoint.

## Procedure

1. Read `CLAUDE.md`, `README.md`, any `CONTEXT.md`/`CONTEXT-MAP.md`, `docs/`, ADR directories, CI config. Find what's documented where.
2. Exists → refresh in place (per above). Else → create. Either way, write/update the five entries below — a few lines of pointers each, inline prose only for gaps.
3. **Resolve undiscoverable required fields before publication.** Show the user the draft and ask them to fill anything undiscoverable (usually login flow, where test credentials live). Do not commit an incomplete required entry — wait for the human's answer, then continue to step 4.
4. **Commit and push it before any hive role is dispatched.** A cloud Lead clones from git; an uncommitted profile does not exist for it.
   - **Never on `main`.** You run before any unit branch exists, so a fresh repo has you standing on `main` — cut a dedicated branch first: **initial setup** creates `chore/project-profile` from current `main`; **a refresh** creates or resets a clean `chore/project-profile-refresh` from current `main` — never continue new work on a branch already merged, which may be stale, behind, or deleted remotely. Either branch already carries unrelated or unmerged work → stop and report the collision rather than overwriting it. Never mix this commit into an unrelated ticket branch. Law *H* has no carve-out for setup commits.
   - Record the branch and worktree you started from before switching to the setup branch — you'll return to it at the end.
   - Guarded steps, not one compact chain — a bare `;`/`&&` sequence lets a failed step continue into the next one:
     ```bash
     git add PROJECT-PROFILE.md || exit 1
     if [ -f docs/review-lenses/security.md ]; then git add docs/review-lenses/security.md || exit 1; fi
     if ! git diff --cached --quiet; then
       git commit -m "PROJECT-PROFILE: <initial profile|refresh>" || exit 1
     fi
     git push -u origin HEAD || exit 1
     ```
     Stage the security rubric by its exact path, never the whole `docs/review-lenses/` directory — that folder may hold other lens files not part of this write, and the no-mixing-unrelated-work rule applies to staging, not just branches. A freshly cut branch has no upstream, so a bare `git push` fails and the profile is committed locally and never seen. Any failure above (staging, hook, signing, identity, push) is a blocker — stop and report it.
     - **Nothing staged (a refresh found no changes)** → verify the existing `PROJECT-PROFILE.md` is already reachable from current `main`. Reachable → skip the PR/merge step entirely, restore your original checkout, and return `ready · profile current · no changes`, never opening a no-op PR. Not reachable → that's a publication inconsistency (the working copy is current but never made it to `main`) — stop and report it, don't silently re-push.
   - **Then get it merged — this is a hard precondition, not a nicety, and "PR opened" does not satisfy it.** A profile sitting on `chore/project-profile` is invisible to every role, because **every unit's root ultimately parents off `main`** (`/hive-master` §8.1) — there's no alternate base a hive role can be handed instead. Open a PR and have the human merge it **before kickoff**. **No hive role dispatches until `PROJECT-PROFILE.md` is reachable from `main`** — an open, unmerged PR is not yet reachable.
   - **You may return before the human merges — that's not the same as being done.** Once the PR is open, return `blocked · pending profile merge · <PR-url>`, restoring your original branch/worktree first. On re-entry, verify `PROJECT-PROFILE.md` is reachable from `main` (re-fetch and check, never assume from memory) before reporting ready: return `ready · profile merged · pr:<PR-url> · head:<sha>` — the same PR, now merged, and the sha it merged at — never report setup complete merely because the PR exists.
   - When finished (merged, or returning blocked pending merge), return to the exact branch and worktree you recorded above and verify it's checked out — don't rely on `git switch -` alone, which can restore the wrong branch or fail across worktrees; never leave the checkout stranded on the setup branch, since the next skill cuts its branch off whatever HEAD you leave behind.
   - *Naming carve-out: this runs before any tracker item exists, so the branch and subject carry no `LP-`/`LAK-` keys and no role tag — exempt from Law *D*'s naming by design, and the only hive commit that is.*
   - **Never a Conventional-Commits subject** (`feat(...)`, `chore(...)`) — same rule every hive role obeys.
   - On a mid-run refresh, push before reporting the refresh to the caller.

## Before return

- `PROJECT-PROFILE.md` exists at the repo root, tracked by git, title + all five headings present exactly once in order.
- Entry 1 carries the exact `Bootstrap skill:` line (below), and the exact `Global-ID namespaces:` line — every directory where numbered identities are allocated, or `none`.
- Every pointer resolves to an existing file or documented location; commands verified against repo docs or CI config.
- Entry 2 names the E2E CI surface (the required check, the checks its dispatch raises alongside, and how it is dispatched) with a local command only where no CI check exists, or `none`.
- Entry 3 covers launch, URL/port, login flow, and a credential-location pointer — never credentials themselves.
- Entry 4 carries `Automated PR review: <Cursor Bugbot | none>`.
- Manual content within the five entries is unchanged; no sixth top-level heading exists.
- One of: changes pushed and the profile verified reachable from `main` (`ready · profile merged · pr:<PR-url> · head:<sha>`); changes pushed and you're returning `blocked · pending profile merge · <PR-url>`; or nothing was staged and the existing profile was verified already current on `main` (`ready · profile current · no changes`). Never reporting ready off an open, unmerged PR, and never opening a no-op PR when nothing changed.
- You've returned to the branch/worktree you started from.

## The five entries

Five headings, fixed order — a contract. Consumers match by name; `/hive-scout` (entry 3) and `/hive-architect` (entry 1) match by number. Keep exact titles and order on every write and refresh. File opens with a title line:

```text
# PROJECT-PROFILE — <repo>

## 1. Docs pointers
## 2. Commands
## 3. App access for verification
## 4. Review lenses
## 5. Tracker
```

1. **Docs pointers** — glossary, architecture docs, ADR directories, repo rules, troubleshooting/runbook docs (where project-specific operational problems are recorded). Links for all of these — the exact `Bootstrap skill:` and `Global-ID namespaces:` lines below are the only non-link fields this entry carries.
   - Also note the repo's **bootstrap skill**, if one exists — check for it like any repo convention.
     - *Why:* it creates the ticket + a dedicated worktree/branch for new work *before* any hive skill runs.
   - Exact required line, kept present on every refresh — `/hive-architect` and `/hive-master` branch on it, not on prose:
     ```text
     Bootstrap skill: /<skill-name>
     ```
     or
     ```text
     Bootstrap skill: none — /hive-architect creates the worktree/branch itself.
     ```
   - **Global-ID namespaces** — every directory where numbered identities are allocated (ADR dir, migrations dir, any other numbered series), comma-separated on one line, or `none`:
     ```text
     Global-ID namespaces: docs/adr, backend/migrations
     ```
     - *Why:* these are `check-drift`'s inputs — a number is picked from a branch-local view, so two branches take the same one and the files merge with no conflict (`/hive-ticket-lead` §10.1). An unlisted directory is never checked.
2. **Commands** — test, lint, dev server, per area. Point at CLAUDE.md if it lists them.
   - Include a dead-code command where the stack has one (`npx knip` / `ts-prune` for TS, `vulture` for Python).
   - *Why:* the Inspector's code-shape lens runs it over touched packages.
   - **E2E: record the CI surface** — the workflow file, the required check name, and how it is dispatched (`workflow_dispatch` + its inputs, marker commit, label). Record a local command **only** for a repo carrying no CI E2E check at all; `none` where the repo has no suite.
   - **Name the dispatch's own byproduct checks too:** the checks the workflow raises beside the required one (`Trigger E2E Tests` / `PR E2E Tests` beside `Luma SDLC E2E Tests`). Fire the workflow once on a scratch PR and read `gh pr checks` if the answer is not obvious from the YAML. They become `hive-dispatch e2e --e2e-sibling`. Anything undeclared is treated as an unrelated check and gates the E2E dispatch, which is correct for everything the workflow did **not** create.
   - *Why:* the gate is satisfied by that check and never by a local run (`/hive-master` §12) — a profile offering a local command where a check exists is what sends a gate to the wrong surface. An unnamed byproduct is the other half of the same defect: the E2E gate ends up blocking on a check its own dispatch created.
3. **App access for verification** — how to launch, URL and port, how to log in, pointer to where test credentials live (never the credentials themselves).
   - Multiple instances runnable at once → record how (port/env), so parallel DOD checks are possible.
   - No existing doc usually covers this one — write inline.
4. **Review lenses** — review dimensions that matter for this stack, each pointing at the doc defining the rule (e.g. tenant isolation → the database guide).
   - Consumers: `/hive-inspector` picks from this list; `/hive-panel` uses the same entry as its roster.
   - External input surface present → instantiate the security rubric:
     1. Copy `references/security-lens-rubric.md`, bundled in **this skill's own directory** (`/hive-profile-builder`, not the project repo), to `docs/review-lenses/security.md` in the project repo.
     2. Fill its per-stack section.
     3. Point the security lens at it.
   - **Can't tell whether the repo accepts external input** — inspect routes, webhooks, upload paths, message consumers, CLI arguments, and public APIs. Still uncertain → instantiate the rubric anyway; never omit the security lens on an assumption.
   - Frontend-heavy repos may add a short UI-slop lens file — generic gradient themes, default component-library styling, stock "Welcome to X" heroes, placeholder-service images. All are findings; cite the design system instead.
   - **Declare the automated reviewer on its own line: `Automated PR review: <Cursor Bugbot | none>`.** An automated PR reviewer is a review lens like any other, so it is declared here rather than under a heading of its own — the five entries are a fixed contract and never grow a sixth.
     - **`none` is a legitimate answer, not a gap.** Company repos have Cursor Bugbot; personal and sandbox repos have none, by choice. Write `none` plainly; never leave the line off, and never imply a repo without one is misconfigured.
     - Consumers branch on it — `/hive-ticket-lead` (per-ticket checkpoint), `/hive-gate-lead` (checkpoint-2 loop), `/hive-master` (E2E ordering, completion, wrap-up), `/hive-pr-bugbot-triage` (entry guard). Declared → the bugbot loop is a **hard gate, unchanged**. `none` → the loop does not run, and every consumer **records** `bugbot: none in profile` rather than skipping silently, exactly as a missing E2E command is recorded (`/hive-master` §10).
     - *Why the record, not a skip: a silently omitted check and a check that failed unnoticed look identical afterwards. Naming the omission is what keeps them apart.*
5. **Tracker** — declare exactly one of the three modes (Law *D*), never a mix:
   ```text
   Jira: <project key + conventions | none>
   Linear: <team key + conventions | none>
   GitHub Issues: <owner/repo, label conventions | none>
   ```
   - **Real tracker** (the corpus term — see `/hive-issue-planner`'s tracker-mode split) → **every tracker this repo actually uses** is declared and created in (Law *D*). Two declared → both keys on every ticket. **One declared is legitimate**: a repo running on Jira alone declares `Linear: none`, and that is an answer, not a gap. Ask which trackers this work really uses rather than assuming two; a declared tracker nobody posts to blocks kickoff over nothing.
   - **GitHub-issues** — the human's own personal/tooling repo with no Jira/Linear project for this work. `Jira: none` and `Linear: none` are correct here, not gaps — do not chase a Jira/Linear key that doesn't exist. `GitHub Issues:` names the repo (usually this one) and any label conventions to reuse (e.g. `ready-for-agent`, `phase-N`, matching the triage-label doc where one exists).
   - **Rehearsal/local-files run** (no tracker at all — `tickets.md`/`spec.md` only) → `none` on all three is the correct, legitimate declaration; nothing here overrides that mode.
   - **Never infer the mode from the repo's owner alone** — ask, if it isn't already obvious from an existing profile or the human's stated intent for this repo.
