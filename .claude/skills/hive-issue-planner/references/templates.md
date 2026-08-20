# Hive Issue Planner — publish templates

Used by step 6. `<tickets-file-template>` = **rehearsal** shape (local `tickets.md`, no tracker keys — workflow exercise only, never ships, Law *D*); `<issue-template>` = per-issue shape, shared by both real-tracker and GitHub-issues publication — only the Tracker keys section below differs between them.

**`<gap-list-template>` is the one block step 6 never publishes.** It is a step-5a working artifact that lives beside the tickets file; its rows either become tickets or become findings. It sat *inside* `<issue-template>` once, and that block ships verbatim as the issue body, so every published Jira / Linear / GitHub issue would have carried a gap list.

<gap-list-template>

## Invariant trace gap list — step 5a output

One row per gap. No prose, no narrative, no restated spec. A clean trace returns the header and `no gaps`.

```markdown
| id | invariant / AC | owning ticket | falsifying DOD | gap |
|---|---|---|---|---|
| I-5 | writer fence holds across takeover | — | — | no owning ticket |
| I-7 | terminal event is exactly-once | 1.4 | 1.4 DOD 2 | DOD cannot falsify |
| AC-3 | resume replays from cursor | 1.2, 2.1 | 1.2 DOD 1 | contradiction: 2.1 replays from zero |
```

`gap` ∈ `no owning ticket` · `DOD cannot falsify` · `contradiction: <what>`. Every row is `mechanical` — a fact by lookup, so it enters `/hive-panel` admitted and never reaches a debater.

</gap-list-template>

<tickets-file-template>

# Tickets: <short name of the work>

A one-line summary of what these tickets build. Reference the source spec if there is one.

Work the **frontier**: any ticket whose blockers are all done. For a purely linear chain that means top to bottom.

## phase-1: <phase name>

### 1.1 <Ticket title>

**What to build:** the end-to-end behaviour this ticket makes work, from the user's perspective — not a layer-by-layer implementation list.

**Blocked by:** the ids of the tickets that gate this one, or "None — can start immediately".

**Gate:** autonomous (or HITL)

**Tier:** default (or strict — risk, blast radius, or irreversibility; name which)

- [ ] Acceptance criterion 1
- [ ] Acceptance criterion 2

**Verification DOD:** (or `**Verification DOD (api-level):**` when there's no user-visible surface)

1. Do X. Screen shows Y.
2. ...

### 1.2 <Ticket title>

...

## phase-2: <phase name>

### 2.1 <Ticket title>

...

</tickets-file-template>

<issue-template>

## Labels

`ready-for-agent` (always) · `hitl` (`Gate: HITL` only) — never `ready-for-human`.

## Tracker keys

**Real tracker:** one key per tracker the profile **declares** (Law *D*) — `LP-xxxx` (Jira) · `LAK-xxxx` (Linear), identical across mirrors where two are declared; one declared, one key. `/hive-master` kickoff refuses a ticket missing a declared key.
**GitHub-issues:** `GH-xxxx` (the issue number) — the one key this mode has; never additionally created alongside a Jira or Linear key. `/hive-master` kickoff refuses a ticket missing it.

## Parent

A reference to the parent issue on the tracker (if the source was an existing issue, otherwise omit this section).

## Phase

`phase-n: <phase name>`, and this ticket's numeric id (e.g. `2.3`).

## Tier

`default` or `strict` (risk, blast radius, or irreversibility - name which). **Required.**
Omit it and the published issue loses the tier, so a Lead reading the tracker defaults a
`strict` ticket to the cheap batched loop and the extra ceremony never runs.

## Gate

`autonomous` or `HITL`. On Jira, HITL tickets also get the `hitl` label — **never** `ready-for-human`, which means *requires human implementation* (`docs/agents/triage-labels.md`) and contradicts the `ready-for-agent` label these tickets already carry.

## What to build

The end-to-end behaviour this ticket makes work, from the user's perspective — not layer-by-layer implementation.

## Acceptance criteria

- [ ] Criterion 1
- [ ] Criterion 2

## Verification DOD

(or `## Verification DOD (api-level)` when there's no user-visible surface)

1. Do X. Screen shows Y.
2. ...

## Blocked by

- A reference to each blocking ticket, or "None — can start immediately".

</issue-template>
