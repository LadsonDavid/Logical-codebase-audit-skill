# codebase-audit

> A Claude skill that answers one question: **does this codebase tell the truth about what it claims to do?**

![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)
![Works with Claude Code](https://img.shields.io/badge/Claude-Code%20%7C%20Claude.ai%20%7C%20API-orange)
![Phases](https://img.shields.io/badge/Phases-12-green)
![Books](https://img.shields.io/badge/Book%20Frameworks-21-purple)

Runs a 12-phase evidence-based audit across any codebase. Language-agnostic. No browser tools, no visual checks. Pure code evidence via `grep`, `git log`, `find`, and file reads.

The skill carries an embedded Business Knowledge Library. It knows what code must exist to support claims like "automatic," "real-time," or "bidirectional sync" without needing any documentation or spec from you.

---

## Why This Exists

Most code reviews check for bugs or style. This skill checks for **truthfulness**.

The most dangerous gap in a codebase is not a bug. It is when the code "works" but does not do what the product claims it does. A feature described as "automatic" that requires a manual button press. A "bidirectional sync" that only reads. A "correlation engine" that is a sorted list. This skill finds those gaps and names them with evidence.

---

## Quick Start

### 1. Install

Download `SKILL.md` and save it to your Claude workspace:

```
.claude/skills/codebase-audit/SKILL.md
```

That is it. Claude will detect and load it automatically.

### 2. Use

Tell Claude what to audit:

```
Audit the codebase at /path/to/repo. The product claims real-time incident triage.
```

```
Is our Slack integration bidirectional or read-only? Check /src/integrations/
```

```
Find every phantom feature — things claimed in the README with no code behind them.
```

### 3. Auto-Trigger Phrases

Claude activates this skill when you say any of these:

| Phrase | What It Runs |
|--------|-------------|
| `"audit our codebase"` | Full 12-phase audit |
| `"is our code production-ready?"` | Enterprise readiness + integration scorecard |
| `"what is actually implemented vs claimed?"` | Business logic audit against all claims |
| `"find phantom features"` | Claim-to-code matching across README, docs, pitch |
| `"is this integration real or mocked?"` | Phase 3: Integration completeness |
| `"what breaks first at scale?"` | Phase 7: Performance anti-pattern scan |
| `"find the chain-link bottleneck"` | Phase 11: Chain-link report |
| `"stress-test our architecture"` | Phase 1 + 10: Architecture + first principles |

---

## The 6-Verdict Evidence System

Every finding gets exactly one verdict:

| Verdict | What It Means |
|---------|---------------|
| `IMPLEMENTED + ALIGNED` | Code exists and supports the business claim |
| `IMPLEMENTED + MISALIGNED` | Code exists but contradicts what the product claims |
| `STUBBED` | Code exists but returns hardcoded or mock data |
| `PHANTOM` | Claimed in docs, README, or pitch — zero code found |
| `BROKEN` | Code exists, demonstrably non-functional |
| `RISK` | Works today, identifiable failure mode at scale or edge case |

**MISALIGNED is the most dangerous verdict.** The code passes tests. The feature ships. But it does not do what the business claims. This is the gap that costs you enterprise deals and breaks trust with early customers.

---

## Embedded Business Knowledge Library

No spec needed. The skill already knows what each claim requires in code.

| If the product claims... | Code must have... | Default verdict if missing |
|--------------------------|-------------------|---------------------------|
| "Automatic X" | Event listener, webhook handler, or background job (not a button) | `MISALIGNED` |
| "Real-time Y" | WebSocket, SSE, or polling under 5 seconds | `MISALIGNED` |
| "Bidirectional sync with Z" | Both GET and POST/PATCH calls to Z's API | `STUBBED` |
| "Correlation engine" | Signal weighting or scoring algorithm, not a sorted list | `MISALIGNED` |
| "Multi-tenant" | Tenant ID scoped on every single database query | `RISK` |
| "Role-based access (RBAC)" | Permission checks inside business logic, not only route guards | `RISK` |
| "Audit log" | Write events firing on every state-changing action | `PHANTOM / STUBBED` |
| "One-click rollback" | Version snapshot and restore logic, not only a UI button | `STUBBED` |
| "Deploy safety gate" | Pre-commit or pre-deploy hook that blocks on a failed condition | `PHANTOM` |
| "Incident triage" | Parallel signal fetch across multiple sources with correlation scoring | `MISALIGNED` |
| "Reduces time from X to Y" | Timing instrumentation in code (otherwise the claim is unverifiable) | `RISK` |

---

## 12 Phases

| Phase | Name | Key Question |
|-------|------|-------------|
| 0 | Territory Map + Metalearning Protocol | What is the actual shape of this codebase before any verdict? |
| 1 | Architecture Honesty + Abstraction Layer Integrity | Do layers hide implementation or leak it upward? |
| 2 | Dependency Danger Map + Ownership | Who owns what? What has no owner? What breaks if this fails? |
| 3 | Integration Completeness | Are claimed integrations real, partial, stubbed, or phantom? |
| 4 | Business Logic Audit | Does the code support every major product claim? |
| 5 | Enterprise Readiness | Multi-tenancy, RBAC depth, audit log, secret management, error handling |
| 6 | Test Quality | Do tests break when features break, or only when code changes? |
| 7 | Performance Anti-Pattern Scan | N+1 queries, unbounded reads, scale mismatch (over or under) |
| 8 | Technical Debt Localization | Where is debt concentrated? Is it paying down or compounding? |
| 9 | Chesterton's Fence Pass | Before flagging weird code, understand why it exists via git blame |
| 10 | First Principles Pass | What can be deleted? What exists with no discoverable justification? |
| 11 | Chain-Link Report | Which single component sets the performance ceiling for everything else? |

Phase 0 always runs first. The metalearning protocol maps the full codebase before any verdict is issued. The audit covers the actual codebase, not assumptions about it.

---

## Example Output

```
CODE HONESTY REPORT
===================
Product: Incident Response Platform
Stack: TypeScript / Next.js / PostgreSQL
Size: 847 files / 142,000 lines

CLAIM VERDICTS:
- "automatic decision capture from conversations"
  -> MISALIGNED
  Evidence: src/decisions/capture.ts:L88 — only a manual mark-as-decision button.
            No NLP, pattern matching, or event listener found.

- "bidirectional Slack sync"
  -> STUBBED
  Evidence: src/integrations/slack.ts:L44 — GET /channels only.
            No send-message, create-channel, or POST implementation.

- "real-time incident correlation"
  -> IMPLEMENTED + ALIGNED
  Evidence: src/engines/correlation.ts — WebSocket feed + scoring algorithm
            with 6 weighted signal sources.

INTEGRATION SCORECARD: 3 of 7 claimed integrations fully real.
STUBBED: Slack (write), PagerDuty (webhook), Datadog (write)
PHANTOM: LaunchDarkly

CHAIN-LINK: src/engines/healthEngine.ts
  If this is slow, all project health signals are delayed for every user.
  No caching layer, no background computation. Called synchronously on every dashboard load.

PHANTOM FEATURES:
  "deploy safety gate" — no pre-commit hook, no blocking logic found in repo.

OVERALL: PARTIALLY HONEST
  Core incident triage is real. 4 of 7 integrations are STUBBED or PHANTOM.
  Chain-link unaddressed at current load.
```

---

## Requirements

- Claude Code, Claude.ai (paid plan), or Claude API with skills support
- Codebase path accessible from Claude's file system
- Git history available for Phases 8 and 9 (recommended, not required)

---

## Book Sources (21 Books)

### 8 frameworks added in this skill

| Author | Book | Concepts Used |
|--------|------|---------------|
| Charles Petzold | Code | Abstraction layer integrity, layer leakage detection |
| Ben Horowitz | The Hard Thing About Hard Things | Management debt, lead bullets, wartime code |
| Scott Young | Ultralearning | Metalearning protocol, retrieval-based prediction, directness |
| Reed Hastings | No Rules Rules | Context not control, talent density via git blame |
| Ashlee Vance | Elon Musk | First principles engineering, delete before optimize |
| Walter Isaacson | Steve Jobs | Simplicity test, interface reveals function |
| Kim Scott | Radical Candor | Ruinous empathy in code, unchallenged decision detection |
| Brad Stone | The Everything Store | API mandate, two-pizza team ownership, working backwards |

### 13 frameworks carried from logical-thinking skill

Daniel Kahneman, Ray Dalio, Richard Rumelt, Peter Thiel, Carol Dweck, Clayton Christensen, Chris Voss, John Doerr, Chip Heath, David Epstein, Rob Fitzpatrick, Nir Eyal, Kerry Patterson

---

## Related

This skill is the codebase-analysis companion to [logical-thinking](https://github.com/LadsonDavid/Logical-codebase-audit-skill) — which handles product and strategy evaluation through live browser verification alongside logical analysis.

---

## License

MIT. See [LICENSE](./LICENSE) for full text.

---

Built by [Ladson David](https://github.com/LadsonDavid)
