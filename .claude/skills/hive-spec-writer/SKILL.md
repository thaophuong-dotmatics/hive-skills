---
name: hive-spec-writer
description: "Thao's hive workflow — Spec Writer: fork of Matt's to-spec; turn the current conversation into a spec and publish it to the project issue tracker — no interview, just synthesis of what you've already discussed."
disable-model-invocation: true
---

Turn the current conversation + codebase understanding into a spec (aka PRD). Do NOT re-interview the user — synthesize what you know (step-2 seam check still required).

Pipeline: after `/hive-architect` (sharpened plan) → before `/hive-issue-planner` (spec → review-phased tickets).

Tracker + label conventions ← `PROJECT-PROFILE.md` (Tracker entry); no profile → **stop and ask the human to run `/hive-profile-builder`** (it is `disable-model-invocation`, so you cannot run it yourself).

Address the human per **`/hive-report`**. The spec doc itself stays technical.

## Process

1. Explore the repo for current codebase state (if not already). Use the project's domain glossary vocabulary throughout; respect ADRs in the area touched.

2. Sketch the seams you'll test the feature at (vocabulary: `/codebase-design`).
   - Prefer existing seams over new ones.
   - Prefer the **highest seam** — widest real interface nearest the system boundary. *(So tests exercise behavior, not internals.)*
   - New seams → propose at the highest point possible.
   - Fewer is better — ideally one high seam, reused.
   - Check the seams against expectations already established in the conversation and any accepted decisions — never a new interview question. Mismatch → revise and re-check against those same sources before writing the spec.
   - Those sources contradict each other in a way you can't resolve safely → stop and ask the human, naming the conflicting decisions. This is the one exception to "do not re-interview" — resolving a real contradiction is not fishing for new requirements.

3. Write the spec per the template below, then publish. **How** depends on the `PROJECT-PROFILE.md` tracker:
   - **Real tracker** → **create the epic yourself, in **every tracker the profile declares** (Law *D*: one or both, never a mix across tickets), using the available tracker tools** (Law *D*; do not rely on auto-mirror, never ask the human to create it for you). No key exists until its item is created, so:
     1. Draft the complete spec body, keys omitted.
     2. Create the item in each declared tracker with that title and body.
     3. Capture every generated key (`LP-xxxx` / `LAK-xxxx`).
     4. Update each item's body to the final version carrying every key.
     5. That same final body is what gets written to `spec.md` below.
     Don't continue to `spec.md` until every declared tracker holds the same final reconciled body.
     - **Profile declares a tracker but maps no project key for it** → **stop and ask the human to update it via `/hive-profile-builder`**; never publish to a subset of the declared trackers and never invent a project key. `Linear: none` is a declaration, not a missing key (Law *D*).
     - **A tracker tool is unavailable, unauthorized, or fails mid-create/update** → stop, report which tracker is blocked and what the human needs to restore access. Never publish to a subset of the declared trackers, never ask the human to create the item by hand, never continue to `spec.md` as if publication finished. If one item was already created before another failed, keep its key and mark publication blocked rather than creating a duplicate — reconcile every body once access is restored.
     - **No `ready-for-agent` on the epic** — that label means *ready for an AFK agent to implement* (`docs/agents/triage-labels.md`), and this epic is a container awaiting `/hive-issue-planner`; labelling it grabbable invites one agent to build the whole spec in one context. The planner applies `ready-for-agent` to its tickets. No further triage.
   - **Also write the canonical repository copy** — `spec.md` in repo root (mirrors `/hive-issue-planner`'s `tickets.md`), including every declared tracker key. This is the pushed copy the Panel and `/hive-issue-planner` step 1 read; the tracker epic is the durable record.
   - **GitHub-issues** → **create the epic yourself, as a GitHub issue on the repo**, using the available tracker tools (Law *D*). The same ordering problem, solved the same way, with one system instead of two:
     1. Draft the complete spec body, key omitted.
     2. Create the issue with that title and body.
     3. Capture the generated key (`GH-xxxx`).
     4. Update the issue's body to the final version carrying the key.
     5. That same final body is what gets written to `spec.md` below.
     - Never additionally create a Jira or Linear item — one system is the complete, correct publication here.
     - **The tracker tool is unavailable, unauthorized, or fails mid-create/update** → stop, report the blocker; never continue to `spec.md` as if publication finished.
     - **No `ready-for-agent` on the epic** — same reasoning as the real-tracker case above.
     - **Also write the canonical repository copy** — `spec.md` in repo root, including the `GH-xxxx` key. Same role as the real-tracker case: the pushed copy is what the Panel and `/hive-issue-planner` step 1 read.
   - **Local files (no real tracker, no GitHub-issues)** → `spec.md` in repo root is the only copy. Triage labels don't apply.
   - **Both forms: commit and push `spec.md` before offering the next skill** (never on `main`). A Panel conductor or lens runs in its own checkout — an unpushed `spec.md` is an empty path to it, and `/hive-issue-planner` step 1 reads it the same way.

4. After the spec is published and pushed, offer the human exactly two next actions: run `/hive-panel` in document mode over the spec now, while it's still the cheapest thing to change, or continue directly to `/hive-issue-planner`.

Run the Panel only when the human selects it — this offer is optional, unlike `/hive-issue-planner` step 5's own default-on Panel pass over the approved breakdown. When selected, run it with these rules:

- Follow `/hive-issue-planner` step 5 (`$SKILLS_DIR/hive-issue-planner/SKILL.md`).
- **Scope:** the spec alone.
- **No invariant→DOD trace** — no ticket breakdown exists yet; that's the planner's step-5 job, and a lens told to trace it here would mark every invariant an unmappable Gap and fail round 1.
- **Dispatch only the Panel conductor** — it spawns its own lenses and debaters, on either host family.
- **Use a spawn-capable agent type:** `general-purpose` or `claude`; never `Explore` or `Plan`, which cannot spawn.
- Apply `/hive-panel`'s *Document mode* substitutions.
- Route only on the conductor's one verdict line.
- **Budget:** 2 rounds.

Then offer `/hive-issue-planner` to break the spec into review-phased tickets.

<spec-template>

## Problem Statement

The problem, from the user's perspective.

## Solution

The solution, from the user's perspective.

## User Stories

A **long** numbered list. Format:

```
1. As an <actor>, I want a <feature>, so that <benefit>
```

**Actor** = whoever the work serves. *(Backend/infra: often a **calling service, an operator, or an API consumer** — not an end user.)*

<user-story-example>
1. As a mobile bank customer, I want to see balance on my accounts, so that I can make better informed decisions about my spending
2. As the retry worker, I want the charge endpoint to be idempotent, so that a redelivery doesn't double-charge
</user-story-example>

Extremely extensive — cover all aspects of the feature.

## Implementation Decisions

Decisions made — may include:

- Modules built/modified
- Interfaces of those modules modified
- Technical clarifications from the developer
- Architectural decisions
- Schema changes
- API contracts
- Specific interactions

No file paths, no code snippets — they go stale fast.

**Exception:** a prototype snippet that encodes a decision more precisely than prose can (state machine, reducer, schema, type shape) → inline it in that decision, noting it came from a prototype. Trim to the decision-rich parts — not a working demo.

## Ownership and invariants

Before writing this section, read `$SKILLS_DIR/hive-architect/SKILL.md` for the authoritative failure-mode triggers and the six-axis walk. Apply that file's current trigger list directly — don't reconstruct or rely on a duplicated copy here, which would drift.

Required whenever the feature hits any trigger defined there. Else, write "Not applicable" explicitly — name each authoritative trigger checked and found absent. *(A wrong Not-applicable bypasses the whole downstream invariant chain — never silently omit the section.)*

Per data aggregate (table, queue, log, shared structure):

| Field | Specify |
|---|---|
| **Owner** | The one module that owns the aggregate and mediates every write to it |
| **Allowed writers** | Which callers may request a write through the owner, and under what conditions |
| **Serialization mechanism** | What prevents two writers racing |
| **Crash-recovery story** | Who detects a dead writer, who may recover |

**Invariants** — falsifiable; phrase each so a test can violate it.
- *Examples: "exactly one terminal event per turn," "a stale writer's append is rejected after takeover."*
- Each needs **how enforced**: type/schema/DB constraint, or discipline.
- Discipline-only → must map to a test in some ticket's DOD. *(The issue-planner's coverage rule picks it up.)*

Must cover all six axes exactly as defined in `/hive-architect`'s failure-mode walk — name each axis explicitly, including when it's not applicable.

**Ownership or an invariant can't be stated precisely** → design isn't settled. Stop spec publication and return the unresolved point to `/hive-architect`'s failure-mode walk. Never invent an invariant or bury the gap in Further Notes.

## Testing Decisions

Testing strategy — feeds the issue-planner's coverage rule, the builder's TDD, the Scout's DOD verification. Be concrete:

| Item | Specify |
|---|---|
| **What makes a good test** | External behaviour at a seam — never implementation details (`/codebase-design`) |
| **Seams** | Which seam (step 2) each behaviour is tested through; prefer the highest existing |
| **Tiers** | Unit vs integration vs end-to-end coverage, and why |
| **Edge cases & data** | Boundary conditions, failure inputs, fixtures/test data needed |
| **Prior art** | Similar tests already in the codebase to mirror |

**Verify vs validate**
- Happy-path checks *verify* it works.
- **Plus one check per invariant** (from *Ownership and invariants*) that *validates* the failure mode is prevented — phrased so a test can violate it.
- Every invariant → ≥1 test.

**Security**
- Every new input or attack surface (axis 6) gets a **negative test**.
- Unauthorized → denied. Malicious/injection input → rejected or sanitized. Boundary → validated.
- *(Issue-planner turns these into per-ticket DOD.)*

**Verification DOD**
- Observable behaviours defining "done" — concrete enough to check in the real app.
- Seeds the tickets' Verification DOD that the **Scout** drives.
- UI-facing behaviour → name the exact **screens, actions, and expected visual result** to screenshot and record as proof.

## Out of Scope

What's out of scope for this spec.

## Further Notes

Any further notes about the feature.

</spec-template>
