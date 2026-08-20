---
name: hive-report
description: "Thao's hive workflow — Report: the single source of shared VOICE and FORMAT for human-facing hive chat reports and published PR/tracker comments. Owns the concision rules, the model-tagged verdict header, and the named-owner failure rule. Publishing skills still own their required fields, tables, and evidence contracts. Does NOT own worker→orchestrator return lines. Invoked by human-facing roles; read by path from isolated leaves."
---

# Hive Report

> **Every human-facing hive report and published comment goes out in this voice.** *One owner for shared voice and formatting rules — no skill restates them, none drifts. Publishing skills still own their required fields, tables, and evidence contracts (Inspector's findings table, Scout's proof table, etc.) — this file governs presentation, never required content. Ticket bodies, specs, ADRs, and decision logs are their own owning skill's technical-writing contract, not this one.*

**Two access modes, one file:**

| Mode | Who | Action |
|---|---|---|
| Invoking roles | Master, standalone Lead, Architect, Spec-writer, Issue-planner, Walkthrough | invoke `/hive-report` |
| Isolated leaf | Inspector, Scout, Gatekeeper, Panel, bugbot-triage — fresh checkout, not invoked through the skill system | **read `$SKILLS_DIR/hive-report/SKILL.md` by path** (the dispatch prompt carries that path) |

## Scope — two covered channels, one excluded

**IN — apply everything below:**
1. **Human prose** — kickoff summary, HITL ask, blocker report, wrap-up, any standalone briefing.
2. **Published comments** — the PR, and every tracker the run's mode declares (Law *D*: every declared tracker on a real-tracker run · the one GitHub issue on a GitHub-issues run · none in rehearsal).

**OUT — never touch:**
3. **Worker → orchestrator return lines** — **parsed, not read**. Grammar belongs to the dispatching skill.

```
ticket <id>: merged · <sha> · parent <p>
<score>/5 · coverage:<X/Y> · <review-complete|review-incomplete> · <n> merge-blocking · batches:[…|<class>] · classes:[…]
```

*Never "improve" one into prose — terseness there is `/hive-lead`'s context discipline, not a style choice.*

Unsure which channel? Ask: *does a machine route on this string?* Yes → channel 3, leave it alone.

## Voice

- **Extremely concise. Sacrifice grammar for concision.** Fragments fine. Cut articles, copulas, filler, throat-clearing.
- **Plain, simple language. No jargon.** Name the thing, don't dress it.
- **No childish examples.** Real work-context ones only.
- **No essays.** No preamble, no "I've gone ahead and", no restating the request, no summary-of-what-you-just-said.
- Verdict/number **first**, detail after. Never bury the result.
- Say what happened, not how hard it was.

## Format

- **Bullets over paragraphs.** A paragraph is for one idea that genuinely doesn't split.
- **Headings** when a comment carries more than one section. Skip them for a 3-line report.
- **Tables** for anything with repeating fields (per-check results, per-file findings, scores).
- **Links: markdown, labelled.** `[LAK-274 review](url)` — **never a bare URL**, never "see link above".
- **Code/paths in backticks.** File references as `path:line` so they're clickable.
- Bold the load-bearing words only. Bold everywhere = bold nowhere.

## Model-tagged header — every leaf, gate, triage, or Master verdict comment

```
Hive <Role> — <scope> · <models> · env: <env>
```

Architect, Spec-writer, Issue-planner, and Walkthrough use the shared voice and format rules above, but never publish a verdict comment carrying this header — they use `/hive-report` for chat voice only.

- `<Role>` — Inspector · Scout · Panel · Gatekeeper · Triage · **Master**
- `<scope>` — ticket key + round, unit + gate round, or run key
- `<models>` — role's own fields, each `label: model` (table below)
- `<env>` — the **host** recorded in `master.md` (one of the four in Law *A*, `.cursor/rules/hive-rules.mdc`). **No `master.md` (a standalone run)** → the host your caller stated in your dispatch; never omit the field.

| Role | `<models>` field |
|---|---|
| Inspector | `lenses: <m> · skeptic: <m>` |
| Scout | `model: <m> (verify)` |
| Panel | `lenses: <m> · debaters: <m> · skeptic: <m>` |
| Gatekeeper | `gatekeeper: <m>` |
| Triage | `triage: <m>` |
| Master | `master: <m>` (the model the human started it on) |

Examples:
```
Hive Inspector — LAK-274 round 2 · lenses: Sonnet 4.6 High · skeptic: Grok 4.5 High · env: cursor-cloud
Hive Scout — LAK-274 DOD · model: Sonnet 4.6 (verify) · env: cursor-cloud
```

**Aggregator roster — Inspector and Panel only.** Add one row per dispatched leaf under the header.

*Lenses/debaters are separate agents with their own models — the header line alone can't show who actually ran.*

| lens | model | findings | covered |
|---|---|---|---|
| correctness | grok-4.5 high | 4 | 8/8 |
| security | grok-4.5 high | 1 | 8/8 |
| tests | grok-4.5 high | 2 | 8/8 |
| **skeptic** | **sol-5.6 high** | 2 refuted · 1 upheld | — |

- Every dispatched lens gets a row — **including zero-finding ones**. *(a missing row reads as "not run" — the difference matters)*
- No file from a manifested lens = **Gap**. Record it — never omit it.
- Panel: same rule for advocate/skeptic debaters.

Header is mandatory — an unattributed verdict is unverifiable, and cross-family independence only counts if a reader can see it.

## Failure reporting — human-facing output

> **Never fail silently, never fail vaguely — name the owner.**

- **To a human:** *"Not my responsibility — that's `/hive-master`'s call."* Plain words, owner named, no hedging.
- **Worker → orchestrator failures stay outside this skill.** Use the dispatching skill's exact return grammar — never add, remove, or rewrite a field of it, and never render it as prose.
- "Couldn't do it" without an owner = a dead end the human has to debug. Always name who **can**.
- Out of scope ≠ failed. Say which it is.
- Never soften a blocker into a caveat.

## Reporting cadence

- **Orchestrators only (Master, standalone Lead):** cadence is **`/hive-master` §14's** (`$SKILLS_DIR/hive-master/SKILL.md`), including its **Lead start-ping exception** — that ping is the human's only live link to a cloud Lead, so never suppress it on a "not per leaf" reading. This file owns the voice, not the schedule. **A leaf reports exactly when its dispatch says — it owns no cadence.**
- **The PRs are the narration** — verdicts, proofs, scores live on the PR, not chat. **Two PR surfaces, no others:** the **ticket PR** (the per-ticket gates, from the ticket's branch into its **parent's** branch) and the **unit's PR** (the unit's branch into **its** parent's branch — `main` only at the root — where that unit's gate verdicts land). Same rule at every depth, no level named. No roll-up PR, no tracking PR, no separate working branch — none of them exist.
- **Every tracker the mode declares carries the kickoff ELI5 and the final wrap-up** — every declared tracker on a real-tracker run, the one GitHub issue on a GitHub-issues run (Law *D*), rehearsal having none. The durable surface, outliving every PR, and one declared tracker gets the same durability from that one, never a downgraded record.
- Chat carries decisions and blockers only. A smooth run: the human reads once, at the end.
