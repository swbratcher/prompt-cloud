# Examples

Each example is a before/after pair: a canonical `.md` source and its Prompt-Cloud compressed `.pc.md` variant. Every `.pc.md` carries a `pc-hash` header linking it to its source and a token accounting block so you can see exactly what was saved.

---

## example-SOUL.md / example-SOUL.pc.md

**What it is:** An OpenClaw/Hermes-style `SOUL.md` file — the personality, relationship, operating principles, and hard constraints for a 24/7 personal AI agent.

**Why it's a good test case:** SOUL.md is the highest-value compression target in a typical personal agent stack. It is re-injected on every single turn. It is almost entirely behavioral prose — character, tone, relationship dynamics, operating philosophy — with a small but critical verbatim layer (the NEVER constraints).

**Token accounting:**

| | Tokens (approx) |
|---|---|
| Original | ~490 |
| Compressed | ~295 |
| **Reduction** | **~39.8%** |

At 143 sessions/day (95 interactions + 48 heartbeats), that is **~28,000 tokens saved per day** from this one file. Over a week: ~196,000 tokens.

**What to notice in the compression:**

The NEVER constraints are reproduced verbatim — word for word, negation intact. Everything else (character, relationship, operating principles, style) collapses into dense compound clusters. The behavioral frame the model reconstructs is functionally identical to the original.

---

## example-role-spec.md / example-role-spec.pc.md

**What it is:** A behavioral role specification for a senior backend engineer conducting PR reviews. Typical of the role-definition files injected into coding agent systems.

**Why it's a good test case:** Role specs are pure behavioral intent — philosophy, priorities, tone, scope. Almost nothing in a well-written role spec is precision-critical in the verbatim sense. This is the sweet spot for PC compression.

**Token accounting:**

| | Tokens (approx) |
|---|---|
| Original | ~430 |
| Compressed | ~245 |
| **Reduction** | **~43.0%** |

**What to notice in the compression:**

There are no verbatim-preserved sections in this file — it contains no NEVER/MUST constraints, no exact values, no conditional logic. The entire file is compressible behavioral prose, which is why it achieves higher compression than the SOUL.md example. This is the ceiling: a fully behavioral file with no directive content.

---

## Reading the pc-hash header

Every `.pc.md` file begins with:

```
<!-- pc-source: path/to/source.md -->
<!-- pc-hash: 8-char SHA-256 prefix -->
<!-- pc-tokens-original: N -->
<!-- pc-tokens-compressed: N -->
<!-- pc-reduction: N% -->
<!-- pc-validated: behavioral equivalence confirmed -->
```

Before loading a `.pc.md` at runtime, compare `pc-hash` against the first 8 characters of `sha256(source file contents)`. If they don't match, the source has been edited since compression — fall back to the original and regenerate.

Token counts are approximate (English prose averages ~1.33 words/token for GPT-family tokenizers). Use your model's actual tokenizer for precise counts in production.
