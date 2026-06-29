# Codebase Audit Skill

A deep codebase audit skill for Claude, powered by 21 books. Answers one question:

> **Does this codebase tell the truth about what it claims to do?**

Language-agnostic. No browser tools. Pure code evidence: file reads, grep, git log, wc.

The skill has embedded business knowledge — it knows what code must exist to support each type of product claim without needing a spec or documentation.

---

## 6-Verdict Evidence System

| Verdict | Meaning |
|---------|---------|
| **IMPLEMENTED + ALIGNED** | Code exists AND supports the business claim |
| **IMPLEMENTED + MISALIGNED** | Code exists but does not match what the product claims |
| **STUBBED** | Code exists but returns hardcoded or mock data |
| **PHANTOM** | Claimed in docs, README, or pitch — zero code found |
| **BROKEN** | Code exists, demonstrably non-functional |
| **RISK** | Works today, has identifiable failure mode at scale or edge case |

**MISALIGNED is the most dangerous verdict.** The code "works" but does not do what the business claims it does. Most products never find this gap.

---

## Embedded Business Knowledge Library

The skill knows what each product claim requires in code — no spec needed.

| If the product claims... | Code must have... | If missing |
|--------------------------|-------------------|------------|
| "Automatic X" | Event listener / webhook / background job — NOT only a button | MISALIGNED |
| "Real-time Y" | WebSocket / SSE / polling < 5s | MISALIGNED |
| "Bidirectional sync with Z" | Both READ and WRITE API calls to Z | STUBBED |
| "Correlation engine" | Signal weighting or scoring algorithm — NOT only a list | MISALIGNED |
| "Multi-tenant" | Tenant ID scoping on every database query | RISK |
| "RBAC" | Permission checks inside business logic — not only at route level | RISK |
| "Audit log" | Write events on every state-changing action | PHANTOM/STUBBED |
| "One-click rollback" | Version snapshot + restore logic, not only a UI button | STUBBED |
| "Deploy safety gate" | Pre-commit or pre-deploy hook that blocks on failed condition | PHANTOM |
| "Incident triage" | Parallel signal fetching across multiple sources + correlation scoring | MISALIGNED |

---

## 12 Phases

| Phase | Name | Key Frameworks |
|-------|------|----------------|
| 0 | Territory Map + Metalearning Protocol | Ultralearning (Young) |
| 1 | Architecture Honesty + Abstraction Layer Integrity | Code (Petzold), Jobs, Musk |
| 2 | Dependency Danger Map + Ownership | No Rules (Hastings), Bezos |
| 3 | Integration Completeness | Bezos API Mandate |
| 4 | Business Logic Audit | Bezos Working Backwards, Radical Candor (Scott) |
| 5 | Enterprise Readiness | Multi-tenancy, RBAC, Audit logging, Secrets |
| 6 | Test Quality | Ultralearning Retrieval Principle |
| 7 | Performance Anti-Pattern Scan | Horowitz Lead Bullets, Scale Anticipation Fallacy |
| 8 | Technical Debt Localization | Horowitz Management Debt, Wartime Code |
| 9 | Chesterton's Fence Pass | Radical Candor — Unchallenged Decision Detection |
| 10 | First Principles Pass | Musk, Petzold, Jobs |
| 11 | Chain-Link Report | Rumelt |

---

## Ultralearning Metalearning Protocol (Phase 0)

Before any verdict, spend ~10% of audit time mapping the territory:

```bash
find . -type f | grep -v node_modules | grep -v .git | head -100
find . -name "*.ts" | xargs wc -l | sort -rn | head -20
git log --oneline -20
```

Read the entry point cold. Predict what the system does from the first 50 lines alone. A wrong prediction is a **clarity failure**, not auditor failure.

---

## Final Output: Code Honesty Report

Every audit ends with a structured report:

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

CHAIN-LINK: [component] — [failure mode if unaddressed]

PHANTOM FEATURES: [claimed in README/pitch, no code found]

OVERALL CODE HONESTY:
HONEST / PARTIALLY HONEST / DISHONEST — [specific reason]
```

---

## Book Sources (21 Books)

**8 new frameworks:**
- Charles Petzold — Abstraction Layer Integrity
- Ben Horowitz — Management Debt, Lead Bullets, Wartime Code, Scale Anticipation Fallacy
- Scott Young — Ultralearning: Metalearning Protocol, Retrieval, Directness
- Reed Hastings — Context not Control, Talent Density in git
- Elon Musk / Ashlee Vance — First Principles, Delete Before Optimize
- Steve Jobs / Walter Isaacson — Simplicity Test, Interface Reveals Function
- Kim Scott — Ruinous Empathy in code, Unchallenged Decision Detection
- Jeff Bezos / Brad Stone — API Mandate, Working Backwards, Two-Pizza Ownership

**13 carried from logical-thinking:**
Kahneman · Dalio · Rumelt · Thiel · Dweck · Christensen · Voss · Doerr · Heath · Epstein · Fitzpatrick · Eyal · Patterson

---

## Installation

1. Download `SKILL.md`
2. Save to `.claude/skills/codebase-audit/SKILL.md` in your Claude Cowork workspace
3. Trigger phrases (Claude auto-detects these):

```
"audit our codebase"
"is our code production-ready?"
"what's actually implemented vs claimed?"
"find phantom features"
"is this integration real or mocked?"
"what breaks first at scale?"
"find the technical debt"
"review our architecture"
```

---

## Usage Examples

```
Audit the codebase at /path/to/repo — product claims real-time incident triage in under 10 minutes.

Is our Slack integration real or stubbed? Check /src/integrations/

Find all phantom features — we claim automatic decision capture but I want to verify the code.

Run a first principles pass — what can we delete?
```

---

Built by [Ladson David](https://github.com/LadsonDavid)
