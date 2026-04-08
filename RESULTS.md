# Production Results

Measured data from a 24/7 agent system running Claude Opus on a 1M token context window. 95 interactions/day via messaging bridge, 48 scheduled duty-cycle sessions. Static context re-injected every session from a portfolio of PRD and identity files.

These are not projections from a benchmark. They are measured from a running system at production load.

---

## Per-File Compression Results

| File type | Original | Compressed | Reduction |
|-----------|----------|------------|-----------|
| Behavioral role specification | 16,652 tokens | 9,912 tokens | **40.5%** |
| Dashboard product spec | 2,497 tokens | 2,069 tokens | **17.2%** |
| Framework integration spec | 10,977 tokens | 9,879 tokens | **10.0%** |
| Infrastructure inventory | 2,970 tokens | 2,736 tokens | **7.9%** |
| Product specification | 4,827 tokens | 4,628 tokens | **4.1%** |
| Identity preload | 2,876 tokens | 2,623 tokens | **8.8%** |

**Why the range is wide:** Compression rate correlates directly with behavioral content density. The behavioral role specification (40.5%) is predominantly narrative prose with minimal hard constraints. The framework integration spec (10.0%) is 60–65% verbatim — conditional logic, enforcement gates, and exact configuration values that cannot be compressed. The infrastructure inventory (7.9%) is 85% verbatim. The taxonomy correctly identifies what's compressible; the technique only operates there.

Files under ~2,000 tokens are excluded entirely. The fidelity risk exceeds the savings.

---

## Session-Level Impact

| Metric | Value |
|--------|-------|
| Static context before PC | 19,528 tokens/session |
| Static context after PC | 12,535 tokens/session |
| **Session reduction** | **35.8%** |
| Tokens saved per turn | 253 |
| Savings over a 100-turn session | 25,300 tokens |
| Equivalent additional turns freed | ~12 full-complexity turns |

A portfolio session that previously injected 19,528 tokens of static context now injects 12,535. The preload savings compound: 253 tokens saved per turn, every turn.

---

## Daily Operational Savings

| Source | Daily savings |
|--------|---------------|
| Bridge preload (95 turns × 253 tokens) | ~24,000 tokens |
| PRD injections on topic-shift events (~10–15/day × ~6,739 avg saved) | ~67,000–101,000 tokens |
| Duty-cycle preload (48 sessions × 253 tokens) | ~12,000 tokens |
| **Conservative daily total** | **~103,000 tokens** |
| Peak day estimate | ~175,000+ tokens |
| Heavy week projection | **1,050,000+ tokens** |

---

## Cross-System Estimates

The following estimates are derived from documented architecture specifics and community-reported overhead data for each system. They are not measured — they are modeled using the same compressibility taxonomy applied to production.

| System | Per-session bootstrap | With PC | Est. reduction | Source basis |
|--------|-----------------------|---------|----------------|--------------|
| Hegematon System | 19,528 tokens | 12,535 tokens | **35.8%** (measured) | Production deployment |
| OpenClaw active config | ~15,400 tokens | ~12,900 tokens | ~16.2% | docs.openclaw.ai + issue #21999 |
| Hermes Agent (Telegram) | ~13,935 tokens | ~11,300 tokens | ~18.9% | NousResearch/hermes-agent issue #4379 |

**OpenClaw context:** The standard 7-file bootstrap (SOUL.md, AGENTS.md, USER.md, IDENTITY.md, TOOLS.md, MEMORY.md, HEARTBEAT.md) injects 12–20K tokens per session in an active deployment. The compressible layer — SOUL.md, USER.md, behavioral sections of AGENTS.md — is where PC operates.

**Hermes context:** A community monitoring dashboard found 73% of every Hermes API call is fixed overhead (~13,935 tokens). Of that, ~7K tokens is compressible behavioral system prompt content; ~6.9K is verbatim tool definitions. PC targets the behavioral layer only.

---

## Methodology Notes

- Token counts are approximate using GPT-family tokenizer averages (~1.33 words/token for English prose).
- Fidelity validation is behavioral, not textual: equivalent queries run against both original and compressed variants, output compared.
- Cross-system estimates assume the same content-type distribution observed in the measured system — actual results will vary by bootstrap composition.
- "Conservative daily total" excludes peak topic-shift events and uses the lower bound of the PRD injection estimate.

---

*Production deployment: Hegematon System (Hegemon Dev Framework + Automaton Agent Plugin) — [github.com/swbratcher/hegemon](https://github.com/swbratcher/hegemon)*
