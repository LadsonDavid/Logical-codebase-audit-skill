---
name: codebase-audit
description: Deep codebase audit skill powered by 21 books. Use whenever someone wants to audit a codebase, verify that product claims are actually implemented in code, find phantom features (claimed but no code exists), detect stubbed integrations, identify wartime code left unrefactored, run a first-principles pass on architecture, find the chain-link bottleneck, check if internal APIs are clean enough to externalize, or prepare code for investor or customer review. Trigger when the user says "audit our codebase", "is our code production-ready?", "what's actually implemented vs claimed?", "find the technical debt", "is this integration real or mocked?", "stress-test our code", "find phantom features", "what breaks first at scale?", "review our architecture", or any variant of wanting an honest independent code evaluation. The skill has embedded business knowledge — it knows what each type of product claim requires in code without needing a spec. Pure codebase analysis only — no browser or visual tools.
---

# Codebase Audit Skill

**Core question:** Does this codebase tell the truth about what it claims to do?

A codebase makes implicit claims. This skill audits whether the code keeps them.

## Before Starting

Ask the user for:
1. Path to the codebase root (or repo URL if accessible)
2. One-sentence description of what the product claims to do
3. The list of integrations claimed (Slack, GitHub, Jira, etc.) — even a rough list

Use `find`, `grep`, `git log`, `cat`, `wc`, and `ls` throughout. No browser tools. Pure code evidence.

---

## Evidence System — 6 Verdicts

| Verdict | Meaning |
|---------|---------|
| **IMPLEMENTED + ALIGNED** | Code exists AND supports the business claim |
| **IMPLEMENTED + MISALIGNED** | Code exists but doesn't match what the product claims (e.g., "automatic" in copy, manual trigger in code) |
| **STUBBED** | Code exists but returns hardcoded or mock data |
| **PHANTOM** | Claimed in docs, README, or pitch — zero code found |
| **BROKEN** | Code exists, demonstrably non-functional |
| **RISK** | Works today, has identifiable failure mode at scale or edge case |

**MISALIGNED is the most dangerous verdict** — it's the gap most products never find because the code "works" but doesn't do what the business claims it does.

---

## Embedded Business Knowledge Library

The skill knows what each claim type requires in code — no spec needed.

| If product claims... | Code must have... | If missing → verdict |
|----------------------|-------------------|----------------------|
| "Automatic X" | Event listener / webhook handler / background job / cron — NOT only a manual trigger button | MISALIGNED |
| "Real-time Y" | WebSocket server, SSE endpoint, or polling interval <5s | MISALIGNED |
| "Bidirectional sync with Z" | Both read (GET) AND write (POST/PATCH/PUT) API calls to Z | STUBBED |
| "Decision/event capture" | NLP, pattern matching, or ML classification logic — NOT only a manual "mark as" button | MISALIGNED |
| "Reduces time from X to Y" | Timing instrumentation in code — otherwise the claim is unverifiable | RISK |
| "Correlation engine" | Signal weighting or scoring algorithm — NOT only a list aggregator | MISALIGNED |
| "Multi-tenant" | Tenant/org ID scoping on every database query — NOT only at the route level | RISK |
| "Role-based access (RBAC)" | Permission checks inside business logic functions — NOT only at the API route | RISK |
| "Audit log" | Write events firing on every state-changing action | PHANTOM/STUBBED |
| "One-click rollback" | Actual version snapshot + restore logic, not only a UI button | STUBBED |
| "Incident triage / response" | Parallel signal fetching across multiple sources + correlation scoring | MISALIGNED |
| "Unified inbox / feed" | Aggregation logic across all claimed sources, not only one | STUBBED |
| "Deploy safety gate" | Pre-commit or pre-deploy hook that blocks on failed condition | PHANTOM |

Apply the matching library rule to every major product claim in Phase 4.

---

## Ultralearning Protocol (Run at the Start of Every Audit)

Before any verdict, spend ~10% of audit time on metalearning — drawing the map:

1. `find . -type f | grep -v node_modules | grep -v .git | head -100` — get the file landscape
2. `cat README.md` (or equivalent) — what does the codebase say it is?
3. Identify the primary entry point (index.js, main.py, app.ts, etc.)
4. Identify the primary data model (schema file, migration files, types)
5. Count lines of code per directory: `find . -name "*.ts" | xargs wc -l | sort -rn | head -20`
6. Read the first 50 lines of the core entry point cold — predict what the system does. If prediction is wrong, that is a clarity failure, not auditor failure.

Only then begin the 12 phases.

---

## Phase 0 — Territory Map

Produce a factual map before any verdicts:
- Stack: languages, frameworks, major dependencies (`package.json` / `requirements.txt` / `go.mod`)
- Entry points: where does execution begin?
- Primary data model: what are the core entities?
- External services: what does the code call out to? (`grep -r "https://" --include="*.ts" | grep -v test`)
- Team signals: `git log --oneline -20` — how recent is activity? Who commits most?
- Size signal: total files, total lines — is this a prototype or a mature codebase?

**Output:** TERRITORY MAP (factual, no verdicts yet)

---

## Phase 1 — Architecture Honesty + Abstraction Layer Integrity

*Code (Petzold) · Steve Jobs · Elon Musk*

Map the abstraction layers (e.g., HTTP → Controller → Service → Repository → DB). For each boundary:
- Does each layer hide implementation from the one above it cleanly?
- Does any lower-layer concern leak upward? (DB queries in controllers, HTTP logic in business functions)
- Is the layer necessary? Apply first principles: what fundamental constraint is this layer solving? If no clear answer → candidate for deletion.
- Jobs simplicity test: can a new engineer understand each module in under 10 minutes by reading it cold? If not — is the complexity doing necessary work or is it accidental?
- Check: does the function name and signature tell you exactly what it does? Hidden behavior behind reassuring names = architecture failure.

Run: `grep -rn "require\|import" src/ | head -50` to map actual dependencies vs stated architecture.

**Verdict:** COHERENT (layers clean) / LEAKING (specify which layer, which direction) / PHANTOM ARCHITECTURE (diagram exists, code doesn't match)

---

## Phase 2 — Dependency Danger Map + Ownership

*No Rules (Hastings) · Bezos (two-pizza rule)*

- List all internal modules/services. For each: what breaks if this fails?
- Identify blast radius: `grep -rn "import.*ModuleName" --include="*.ts"` — how many files depend on this?
- Ownership check via git: `git log --follow -p <file> | grep "^Author" | sort | uniq -c | sort -rn` — who owns this module? More than 3 authors with no clear lead = no real owner.
- Orphan code: files with no git activity in >6 months that are still imported = dangerous unknowns.
- Two-pizza test: can each service/module be understood and owned by a team of 4-6? Modules touching 10+ other modules = coordination debt, not architecture.
- Context vs control: does the code communicate its intent to future developers, or just enforce patterns?

**Verdict:** HEALTHY (clear ownership, bounded blast radius) / ORPHANED (list files) / OVERLOADED (list modules exceeding two-pizza scope)

---

## Phase 3 — Integration Completeness

*Bezos (API Mandate)*

For every claimed integration, grep the codebase and classify:

Steps per integration:
1. `grep -r "slack\|slack.com\|WebClient\|SlackAPI" --include="*.ts" -l` — does code exist at all?
2. If yes: does it READ (GET) only, or also WRITE (POST/send message/create channel)?
3. Does OAuth flow exist? (`grep -r "oauth\|access_token\|refresh_token"`)
4. Does a webhook listener exist? (`grep -r "webhook\|/events\|/webhook"`)
5. Are credentials handled via env vars or hardcoded? (`grep -r "xoxb-\|sk-\|Bearer " --include="*.ts"`)
6. Bezos API Mandate test: are internal service interfaces clean enough to be externalized? Could this integration module be published as a standalone library without rewrites?

Classify each claimed integration: REAL (bidirectional + auth + webhooks) / PARTIAL (read-only or missing auth) / STUBBED (mock/hardcoded) / PHANTOM (no code found).

**Verdict:** Integration scorecard — X of Y claimed integrations fully real. List each.

---

## Phase 4 — Business Logic Audit

*Bezos (working backwards) · Radical Candor (Scott)*

Apply the Business Knowledge Library to every major product claim:

For each claim:
1. State the claim explicitly ("automatic decision capture from conversations")
2. Identify what the code MUST have to support it (from the library above)
3. Search: `grep -rn "decision\|capture\|detect\|classify\|nlp\|parse" --include="*.ts" -l`
4. Read the matching files. Is the logic real, or a placeholder function that always returns true/mock data?
5. Working Backwards test: was this module designed from the caller's needs backwards, or from implementation convenience forwards? Convenience-first = API debt (hard to use, easy to build).
6. Radical Candor scan: look for unchallenged decisions — over-commented defensive code, functions named "temp" or "fix" that are months old, TODO comments never actioned. These are decisions that were never honestly reviewed.

**Verdict:** Per claim → IMPLEMENTED+ALIGNED / IMPLEMENTED+MISALIGNED / STUBBED / PHANTOM. List the specific file and line for each.

---

## Phase 5 — Enterprise Readiness

For B2B products, check these non-negotiables:

- Multi-tenancy: `grep -rn "org_id\|tenant_id\|workspace_id" --include="*.ts"` — is it present in every query? Missing from even one = data isolation risk.
- RBAC depth: `grep -rn "role\|permission\|canDo\|hasAccess" --include="*.ts"` — are checks inside business functions or only at route level?
- Audit logging: `grep -rn "audit\|log.*action\|activity" --include="*.ts"` — fires on every state change, or only on login?
- Secret management: `grep -rn "process.env\|os.environ\|dotenv" --include="*.ts"` — are all credentials from env vars? Any hardcoded keys?
- Error handling: `grep -rn "catch\|try\|error" --include="*.ts" | wc -l` vs total functions — are errors surfaced or swallowed silently?
- PII handling: where is user data stored? Is it encrypted at rest?

**Verdict:** ENTERPRISE-READY / GAPS: [list specific missing item + file]

---

## Phase 6 — Test Quality

*Ultralearning (retrieval principle)*

Not coverage % — quality:

- Read 5 test files cold. Before reading the test body, predict from the test name what it tests. Wrong prediction = opaque test (implementation test, not behavior test).
- `grep -rn "mock\|stub\|spy\|jest.fn" --include="*.test.*" | wc -l` vs total assertions — high mock ratio = tests testing implementation, not behavior. Breaks without catching real bugs.
- Are business-critical flows tested? (auth, billing hooks, core feature path, error states)
- Do tests include edge cases? (`grep -rn "null\|undefined\|empty\|zero\|max\|overflow" --include="*.test.*"`)
- Self-documenting check: can you learn what the system does by reading only the tests? Yes = good tests. No = tests add no documentation value.

**Verdict:** BEHAVIORAL (tests break when features break) / IMPLEMENTATION-COUPLED (tests break on refactor, not on failure) / UNDERTESTED (critical paths with no tests — list them)

---

## Phase 7 — Performance Anti-Pattern Scan

*Horowitz (Lead Bullets, Scale Anticipation Fallacy)*

Scan for known patterns:

- N+1 queries: database calls inside loops (`grep -rn "for\|forEach\|map" --include="*.ts" -A 3 | grep -i "find\|query\|select"`)
- Sync where async needed: blocking operations in request handlers
- Missing pagination: `grep -rn "findAll\|getAll\|SELECT \*" --include="*.ts"` — any unbounded queries?
- Over-engineered for non-existent scale: microservice architecture with 1 user/day, distributed caching for 100 records, Kubernetes for a single-server app
- Under-engineered for real scale: no connection pooling, no rate limiting, no queue for heavy jobs
- Lead Bullets test: if performance is broken at the core (wrong data structure, wrong algorithm), identify it directly — no clever caching layer fixes a fundamentally slow query

**Verdict:** EFFICIENT / ANTI-PATTERNS: [list pattern + file + estimated impact at 10x load] / SCALE MISMATCH: [over or under-engineered — specify]

---

## Phase 8 — Technical Debt Localization

*Horowitz (Management Debt, Wartime Code)*

Locate WHERE debt is concentrated, not just that it exists:

- Wartime code detection: `git log --format="%ad %s" --date=short | grep -i "fix\|hotfix\|urgent\|quick\|temp\|hack\|bandaid"` — commits during crisis that were never revisited
- `grep -rn "TODO\|FIXME\|HACK\|XXX\|temp\|workaround" --include="*.ts" | wc -l` — count and list oldest TODOs (`git log -S "TODO:" --format="%ad" -- <file>`)
- Compounding debt: identify files changed in >50% of recent commits — these files are the debt epicenter
- Management Debt in code: process shortcuts encoded as code (e.g., hardcoded user IDs, feature flags that were "temporary" 12 months ago, config that should be dynamic but is static)
- Classify each debt item: PAYING DOWN (getting smaller with each feature) / COMPOUNDING (getting worse with each feature)

**Verdict:** DEBT MAP — top 3-5 debt locations with file path + type (wartime/management/architecture) + classification (paying down / compounding)

---

## Phase 9 — Chesterton's Fence Pass

*Radical Candor (Scott)*

Before flagging any weird code as a problem — understand why it exists:

For each unusual pattern found in previous phases:
1. `git log --follow -p <file> | head -100` — what was the original commit message?
2. `git blame <file>` — who wrote it and when?
3. Were there PR comments? (check linked PR if accessible)
4. Radical Candor check: was this code ever honestly challenged, or did it pass review because no one wanted to question the author? Signs of unchallengeable code: no PR comments, written by senior author, surrounded by silence.

Distinguish: INTENTIONAL FENCE (weird but there for a documented reason) / ACCIDENTAL COMPLEXITY (weird with no discoverable reason — safe to refactor) / UNCHALLENGED DECISION (never reviewed honestly — highest risk category)

**Verdict:** Per unusual pattern → INTENTIONAL / ACCIDENTAL / UNCHALLENGED

---

## Phase 10 — First Principles Pass

*Elon Musk · Code (Petzold) · Steve Jobs*

For every major architectural component, apply the three-question test:

1. **Why does this exist?** What fundamental constraint is it solving?
2. **Could this be deleted?** (Musk's rule: delete before optimize. The best code is no code.)
3. **Is there a simpler approach** that achieves the same result with less complexity?

Run: `find . -type f -name "*.ts" | xargs wc -l | sort -rn | head -10` — the largest files are candidates for this pass.

Dependency audit: `cat package.json | grep dependencies` — for each dependency, ask: what does this do that 20 lines of code wouldn't? Each dependency is a liability (security surface, version conflicts, bundle size, breaking changes). Unjustified dependencies = complexity debt.

Abstraction overkill: if a function is only called once, the abstraction is not reducing complexity — it's hiding it.

**Verdict:** Per component → JUSTIFIED (exists for a real constraint) / CANDIDATE FOR DELETION (could be removed without loss) / OVER-ABSTRACTED (complexity added, not removed)

---

## Phase 11 — Chain-Link Report

*Rumelt*

List all major components of the codebase and business model. For each:
- "If this component degrades, does the whole system fail?"
- Identify the single weakest component that sets the performance ceiling for everything else
- Excellence in other components does NOT compensate for the chain-link

Check resource allocation: are engineering hours spent proportional to component risk? Most time goes to visible features, most risk lives in invisible infrastructure.

**Verdict:** CHAIN-LINK: [component + file path] / WHY IT SETS THE CEILING: [specific failure mode] / RESOURCE MISMATCH: [what's over-invested vs under-invested]

---

## Code Honesty Report (Always Produce at the End)

```
CODE HONESTY REPORT
===================
Product: [name]
Stack: [languages + frameworks]
Codebase size: [files / lines]

CLAIM VERDICTS:
- [claim] → IMPLEMENTED+ALIGNED / MISALIGNED / STUBBED / PHANTOM / BROKEN / RISK
  Evidence: [file:line]

INTEGRATION SCORECARD:
- [X] of [Y] claimed integrations fully real
- [integration] → REAL / PARTIAL / STUBBED / PHANTOM

CHAIN-LINK: [component] — [failure mode if unaddressed]

WARTIME CODE: [files + age of unresolved TODO/HACK comments]

UNCHALLENGED DECISIONS: [list — highest risk category]

PHANTOM FEATURES: [claimed in README/pitch, no code found]

FIRST PRINCIPLES CANDIDATES: [components that should be deleted]

ENTERPRISE GAPS: [specific missing B2B requirement + file]

DEBT MAP: [top 3-5 locations, type, paying down or compounding]

OVERALL CODE HONESTY:
HONEST — major claims IMPLEMENTED+ALIGNED, no phantom features, chain-link addressed
PARTIALLY HONEST — some MISALIGNED or STUBBED claims, debt compounding in specific areas
DISHONEST — PHANTOM features claimed, MISALIGNED claims across core product, or chain-link unaddressed

[State which and the specific reason]
```

---

## Source Frameworks (21 Books)

**New (8):** Petzold — Abstraction Layer Integrity · Horowitz — Management Debt, Lead Bullets, Wartime Code, Scale Anticipation Fallacy · Young — Ultralearning Metalearning Protocol, Retrieval, Drill · Hastings/Meyer — Context not Control, Talent Density in git, Process Debt · Vance/Musk — First Principles, Delete Before Optimize · Isaacson/Jobs — Simplicity Test, Interface Reveals Function · Scott — Ruinous Empathy in code, Unchallenged Decision Detection · Stone/Bezos — API Mandate, Working Backwards, Two-Pizza Ownership

**Carried (13):** Kahneman — WYSIATI, Confirmation Bias in code review · Dalio — Believability Weighting via git blame · Rumelt — Chain-Link, Bad Strategy signs · Thiel — Monopoly, is the code creating structural advantage? · Dweck — Fixed mindset code (defensive, untouchable) · Christensen — Jobs-to-be-Done, does code solve the actual job? · Voss — Black Swan unknowns in hidden code states · Doerr — OKR: no measurable outcome = no result · Heath — Curse of Knowledge in code naming · Epstein — Kind vs Wicked environment classification · Fitzpatrick — Behavioral evidence standard applied to code claims · Eyal — Hook model completeness in code · Patterson — Evaluation psychology, separate code from coder
