---
name: hive-architect
description: "Thao's hive workflow — Architect (plan grilling): fork of Matt's grill-with-docs; a relentless interview to sharpen a plan or design, creating docs (ADRs and glossary) as we go, plus a cross-model /hive-second-opinion at major decision points."
---

## Main grill

**Requires `PROJECT-PROFILE.md`** — missing → **stop and ask the human to run `/hive-profile-builder`** (it is `disable-model-invocation`, so you cannot run it yourself); never proceed on an inferred profile. On resume, re-read `PROJECT-PROFILE.md` from the recorded work branch (never cut a new one on resume — see Branch/worktree below). That branch or its worktree missing or unreachable → stop and report the recovery blocker; never silently cut a replacement.
It also names the repo's bootstrap skill, which the Branch/worktree rule below depends on. You are the chain's entry point, so a fresh repo is exactly where you run.

**Precondition — tracker-first (Law *D*).** Ad-hoc work with no tracker item: create it yourself before grilling, in whichever mode `PROJECT-PROFILE.md` entry 5 declares — **real tracker** → in **every tracker the profile declares** (Law *D*: one or both, never a mix across tickets), using the available tracker tools, so every declared key really exists; **GitHub-issues** → one GitHub issue on the repo, using the available tracker tools. Never ask the human to create it for you, never rely on auto-mirror, never create a tracker the profile didn't declare. Repo has a bootstrap skill (`PROJECT-PROFILE.md` entry 1)? It already did this — confirm, don't duplicate. Tracker unreachable or creation fails (either mode) → stop and report the blocker; never invent a key or, in real-tracker mode, proceed on a subset of the declared trackers.

**Branch/worktree — before grilling, never after.** `PROJECT-PROFILE.md` entry 1 names the repo's bootstrap skill → it already created the ticket, worktree and branch; work inside it and never create a second. Entry 1 says "none", **or is silent** → **create the dedicated branch AND worktree yourself now**, switch into it, never grill onto `main` and never write an ADR before it exists. Either way, verify `git branch --show-current` is the dedicated work branch, never `main`, before invoking `/grilling`.

Run `/grilling`; record decisions as ADRs + glossary terms via `/domain-modeling` as you go — only the human's **settled** answer, never a tentative option still in play.

Human: voice + format = **`/hive-report`**, for every status update, recommendation, decision summary, and the final handoff — grill questions stay short, one unresolved question per turn. ADR/glossary structure and location remain `/domain-modeling`'s own.

**Major solution-path decision point** (approach choice, not a clarifying question) → `/hive-second-opinion`. Surface the result before the human decides — a `blocked` result (no independent model available, per that skill's own return) is still a result to surface, never silently skipped. On `blocked`, the human either proceeds without an independent opinion or pauses the decision — record which; never treat `blocked` as agreement:
- Agree/converge → one unified recommendation.
- Split → both views side by side.
- **One call per independent choice between materially different solution paths**, including ones discovered mid-session. Clarifications, parameter tuning, or a consequence of an already-chosen path reuse the existing opinion — they're not a new decision.
- Never re-ask unless the human asks, or new evidence invalidates the original framing.

## Failure-mode walk (second pass)

Trigger — run whenever ALL of these hold:
- Main grill settled the happy path.
- The design involves shared mutable state, concurrency, long-lived processes, or cross-process communication — or adds/changes an external input or attack surface.
- Runs **before** offering `/hive-spec-writer`.

First list the components and state stores in scope, grouping any that share the same ownership and failure behavior — one walk per group, not one per component. Walk the six axes below as grill questions, one at a time, recommended answer each; reuse and cite an answer already settled in the main grill instead of re-asking it. An axis may be marked N/A only with a concrete reason.

| # | Axis | Ask |
|---|------|-----|
| 1 | Ownership & write authority | Who may write each piece of state? How is that authority proven at write time? |
| 2 | Concurrency & interleavings | Two of these run at once — what happens? Which orders are possible? What serializes them? |
| 3 | Crash points & recovery | Process dies before/during/after this step — what happens? Who detects it? Who is allowed to recover? |
| 4 | Idempotency & retries | Delivered or executed twice — what happens? |
| 5 | Migration & deploy safety | Can this land online, on populated data, mid-rollout with old and new code live? |
| 6 | Tenancy, isolation & attack surface | Can this leak or act across tenant/user/session boundaries? |

*If the design adds an external input surface: who authenticates and authorizes each new entry point? Can injected content from it reach a privileged sink (query, shell, HTML, redirect)?*

- Facts → codebase lookup, cited by file. Can't verify, or implementations conflict → an unresolved factual gap, not a human-preference question — surface it as such.
- Decisions → the human, recorded like any grill outcome.
- An unanswered axis is a design gap to resolve now — never an implementation detail to defer.

## Handoff

Human confirms shared understanding →

**Before handoff:**
- Every major solution-path decision is settled, with one surfaced `/hive-second-opinion` result per independent choice — any `blocked` result and the human's decision to proceed without it are recorded.
- Settled decisions are recorded via `/domain-modeling`; no tentative option is left looking authoritative.
- If the failure-mode walk triggered, every in-scope group has all six axes resolved or marked N/A with a reason.
- No unresolved factual or design gap remains — an unverifiable codebase fact (Failure-mode walk) is stopped and reported, never handed off as a speculative design, same as an unanswered axis is never deferred as "an implementation detail."
- The final `/hive-report` summary names every declared tracker key, the work branch, and points at every ADR and glossary artifact created or updated.

- Offer `/hive-spec-writer` to turn the outcome into a spec.
- Also mention optional `/hive-panel` (document-mode adversarial review on that spec).
