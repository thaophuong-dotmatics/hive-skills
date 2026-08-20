---
name: hive-walkthrough
description: "Thao's hive workflow — Walkthrough: after the breakdown is planned (/hive-issue-planner), before any code (/hive-master), brief the human top-down in plain, short language — the whole thing → phases → tickets — so they understand exactly what the agents will build. A comprehension gate, not grilling."
disable-model-invocation: true
---

# Hive Walkthrough

Last gate before implementation. Explain the settled plan back — top-down, short — so the human knows exactly what agents will code before a line is written.

**Briefing, not grilling.** You explain, they confirm. No interrogation, no new design.

| Read first | Contains | Artifact |
|---|---|---|
| Approved breakdown | Tickets + phases + `Blocked by` edges + each ticket's `Gate`, `Tier`, and `Verification DOD` | `tickets.md` (rehearsal), the matching tickets in every declared tracker (real-tracker), or the matching GitHub issues (GitHub-issues); the mode's own source, Law *D* |
| Approved spec | Problem, solution, decisions, scope, invariants | `spec.md` — use the tracker epic only to verify identity and publication state |
| Applicable ADRs | Settled architectural constraints | ADR paths referenced by the spec or breakdown |

Any approved copies disagree — repo vs. tracker, or between declared trackers — stop and return the mismatch to `/hive-issue-planner` (breakdown) or `/hive-spec-writer` (spec); never pick one silently.

Address the human per **`/hive-report`**.

**What a "phase" is:**
- A **review phase** — tickets reviewed as one diff, at one gate, on one phase branch. The branch folds into its approved parent:
  - **Root:** the human folds it into `main` under Law *H*.
  - **Non-root:** the gate-Lead merges it into its parent branch.
- *Not execution order — order comes only from ticket `Blocked by` edges.*
- Say "phase." Never imply a phase is a sequence.

## Process — top-down, one level at a time

One level at a time. Confirm each before the next. Short everywhere — briefing, not re-derivation.

1. **The whole thing** — a few sentences. What we're building or fixing: current issue + current behavior, before vs. expected-after (fix) — or the feature (new work).
   - *Pause: does this match what they wanted?*
2. **Per phase** — short paragraph each. What it tackles, expected outcome when done.
   - *Pause: do the phase groupings and the `Blocked by` dependency order make sense?*
3. **Per ticket** — one line each, under its phase. What it does, its role in the phase.
   - **Call out any `Gate: HITL` ticket** — it pauses again during the build for the human's go. Say so now, so they know which ones upfront.
   - **Call out any `Tier: strict` ticket** — it carries the expensive per-fix review ceremony instead of the batched default (`/hive-ticket-lead` §10.6). Say so now, so they see the cost before signing off.
4. **Sign-off** — ask the human to explicitly approve the settled breakdown for implementation.
   - **Approval must be unambiguous** — "approved," "proceed," or an equivalent direct instruction to start. Acknowledgement ("okay," "I see"), partial agreement, a question, or approval with an unresolved condition is not sign-off.
   - **This explicit sign-off is `/hive-master`'s go-signal** — no implementation without it.
   - **Hand the Master one line to record:** `walkthrough: signed off <YYYY-MM-DD> · <who>` — the human's local calendar date, and the identity already established by the run (never infer a new name or email; no identity established → `human`). It is recorded in `master.md` at `/hive-master` kickoff (§7 step 2), alongside the run-key and host set. That single line is the artifact — the briefing itself stays chat-only.

## Guardrails

- **Explain, don't re-open.** Surface the settled plan — never redesign it.
  - Real gap or disagreement → **upstream** (`/hive-architect`, `/hive-spec-writer`, `/hive-issue-planner`), then re-walk that part. Never patch the plan here.
- **Short.** The whole thing = a few sentences. Phase = short paragraph. Ticket = one line.
  - More depth on one ticket? Expand only that one — never wall-of-text everything.
- **It's a gate.** Confused or unconvinced human = stop, not proceed.
- **Chat-only.** Lives in the conversation. Publishes nothing — no PR, no tracker comment.
