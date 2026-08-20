# Security lens rubric — template

> Instantiated per project by `/hive-profile-builder` as `docs/review-lenses/security.md`; `PROJECT-PROFILE.md`'s **Review lenses** entry points at it. `/hive-inspector` (lens pick) and `/hive-panel` (security/isolation lens) both read the instantiated copy. Fill the per-stack section at profile time; keep the rest verbatim. Distilled 2026-07-17 from ecc's security-bounty-hunter, security-reviewer, security-review (github.com/affaan-m/ecc).

## Evidence standard (gates severity)

**Applies to reachability-class findings only** — injection, SSRF, IDOR, traversal, where reachability *is* the claim. **Presence-class rows** (hardcoded secret, plaintext password compare, unauthenticated route) qualify at their table severity on **file:line evidence alone** — there is no input to trace. For the reachability class: **Major+** = genuinely user-controlled input traced from a real boundary (route, upload, webhook, query param, header, file name) to a meaningful sink (query, shell, HTML, redirect, deserializer, filesystem path), with the smallest safe PoC or a concrete trigger. Theoretical hardening, missing header alone, unreachable path → Minor at most, never merge-blocking.

## In-scope classes (report these)

| Class | CWE | Typical sink |
|---|---|---|
| SSRF | 918 | `fetch(userProvidedUrl)` without allowlist |
| Auth bypass | 287 | middleware ordering / missing check on a route |
| SQL injection | 89 | string-concatenated query |
| Command injection | 78 | shell command built from user input |
| Path traversal | 22 | user input in a filesystem path |
| Deserialization / upload-to-RCE | 502 | remote-reachable deserializer, unchecked upload |
| Stored / auto-triggered XSS | 79 | `innerHTML` / `dangerouslySetInnerHTML` with user content |
| TOCTOU race on money/state | 367 | balance/quota check without a lock → `FOR UPDATE` in a transaction |
| Missing authz on object (IDOR) | 639 | object id from the request, no ownership check |

## Pattern → severity → fix

| Pattern | Severity | Fix |
|---|---|---|
| hardcoded secret / key | **Blocker** | env var / secret store |
| shell command with user input | **Blocker** | `execFile` / args array, no shell |
| string-concat SQL | **Blocker** | parameterized query |
| `innerHTML = userInput` | **Major** | sanitize (DOMPurify) + CSP without `'unsafe-inline'` |
| `fetch(userUrl)` | **Major** | domain allowlist |
| plaintext password compare | **Blocker** | bcrypt / argon2 |
| route without auth check | **Blocker** | auth middleware on every route |
| balance/quota check without lock | **Blocker** | `FOR UPDATE` inside the transaction |
| no rate limit on auth/expensive op | **Major** with a demonstrated brute-force or cost path (credential stuffing, unbounded expensive op); otherwise **Minor**, per the noise catalog | tiered limits, strictest on expensive ops |
| secrets in logs / errors | **Minor** | redaction |

**Every row is `class: bug`** — emit severity + confidence + a merge-blocking *proposal* alongside it, per the finding schema pasted into your dispatch; the conductor sets the authoritative value. Severity is the hive's four-value axis — **Blocker / Major / Minor / Nit** — the only scale the Panel conductor can score (`confirmed Blocker −2.0, confirmed Major −1.0, Minors and Nits never reduce`). The *Evidence standard* above is what gates Major+.

## Noise catalog (do NOT report — the security lens's FP list)

- `.env.example` values, clearly-marked test credentials, intentionally public keys
- SHA256/MD5 used as checksums, not password hashing
- local-only `pickle.loads` / `torch.load`, `eval()` in CLI-only tooling, `shell=True` on fully hardcoded commands
- missing security headers alone; generic rate-limit complaints with no exploit impact; self-XSS
- test / demo / fixture code

Always verify context before reporting; each needs a reachable exploit path to become a finding.

## Deterministic pre-steps

- **Trojan-source scan:** zero-width / bidi unicode anywhere in the diff (document mode: in the reviewed document) = automatic finding.
- Run the stack's audit command where it has one (`npm audit --audit-level=high` or equivalent). **Report advisories as `class: maint`, Minor by default, anchored at the manifest's file:line** — they have no diff file:line of their own. Only an advisory reachable from the diff's own call path goes Major+.

## DOD shapes for security checks (the planner cribs these)

1. Request `<endpoint>` with no token → 401; body contains no record data.
2. Request `<other user's object id>` as user A → 403/404; no field of the object appears.
3. Submit `' OR 1=1--` in `<field>` → validation error on screen; row count unchanged.
4. Replay the same POST twice → exactly one record exists (409 or idempotent success).
5. Exceed the rate limit on `<op>` → 429.

## Per-stack rules (fill at profile time)

<!-- Framework-specific: cookie flags (httpOnly/SameSite), CSRF mechanism, tenancy/RLS mechanism, prod settings checklist, password hasher — point at the stack's own security docs rather than restating them. -->
