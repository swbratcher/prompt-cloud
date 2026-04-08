# Prompt-Cloud Intent Compression

**Semantic compression for LLM behavioral context. Dense compound tokens. Same model behavior. Fewer tokens.**

Prompt-Cloud (PC) is a human-authored compression technique for AI agent prompts and behavioral context files. It fuses overlapping concepts into dense compound tokens and strips grammatical scaffolding — reducing token count significantly while preserving the intent the model acts on.

Built for high-volume, 24/7 agent systems where the same system prompts, role definitions, and workflow rules are injected thousands of times per day.

---

## How It Works

Language models reconstruct meaning from semantic proximity, not syntactic precision. A behavioral description written in 200 tokens and one written in 110 tokens produce the same model behavior — if the semantic content is preserved. The model pattern-matches against the same latent space regardless of how many words you used to get there.

PC exploits this. It strips articles, prepositions, and grammatical connectors that carry no semantic weight for an LLM's internal representation, then bundles overlapping concepts into dense compound clusters. The compressed form is a lossy encoding that targets the same internal representation as the original.

**Example — behavioral role prompt:**

```
Original (prompt.md):
You are a senior backend engineer reviewing pull requests. Focus on correctness,
performance, and maintainability. Be direct but not harsh. If you see a pattern
that will cause problems at scale, flag it clearly. Don't nitpick formatting or
style unless it hurts readability. Always explain why something matters, not just
what's wrong.
```

```
Compressed (prompt.pc.md):
seniorbackendPRreviewer focuscorrectnessperformancemaintainability
directnotharsh flagscaleproblems skipformattingstyle
unlessreadabilityhurt alwaysexplainwhymatters notwhatswrong
```

The negations ("not harsh", "don't nitpick", "unless") are preserved explicitly — never buried inside compounds. A model processing the compressed version activates the same behavioral frame as the original.

---

## Production Results

Measured from a 24/7 agent system running Claude Opus on a 1M context window, with ~95 interactions/day and 48 scheduled duty-cycle sessions:

| File type | Original | Compressed | Reduction |
|-----------|----------|------------|-----------|
| Behavioral role specification | 16,652 tokens | 9,912 tokens | **40.5%** |
| Dashboard product spec | 2,497 tokens | 2,069 tokens | **17.2%** |
| Framework integration spec | 10,977 tokens | 9,879 tokens | **10.0%** |
| Infrastructure inventory | 2,970 tokens | 2,736 tokens | **7.9%** |
| Identity preload | 2,876 tokens | 2,623 tokens | **8.8%** |

**Session-level impact:** A portfolio session previously injecting 19,528 tokens of static context now injects 12,535 — a **35.8% reduction**. At 253 tokens saved per turn across 143 daily sessions, that compounds to **~103,000 tokens saved per day**, exceeding 1 million tokens per week during heavy usage.

Compression rate correlates with content type. Behavioral narrative compresses 25–40%. Files with 60–85% verbatim content (exact values, conditional logic, enforcement gates) compress 8–10%. That ceiling is correct behavior, not a limitation — see the taxonomy below.

---

## The Compression Taxonomy

Not all prompt content compresses equally. The taxonomy is pre-classified before compression begins. **The compressing model executes against the classification. It does not determine it.**

**Compressible — behavioral layer:**
- Role definitions and capability descriptions
- Personality, tone, and communication style
- Operational philosophy and vision
- Behavioral context and framework awareness

**Verbatim — directive layer (never compressed):**
- Every NEVER, MUST NOT, ALWAYS, and MUST constraint
- If/then conditional logic and enforcement gates
- Exact values: numbers, file paths, API endpoints, model names, thresholds
- Security rules, proper nouns, and identifiers of any kind

Paraphrase "never send external email without approval" and you've changed the rule. These stay verbatim.

---

## When to Use It

- Repeated system prompts and role definitions injected at high frequency
- Behavioral directives, tone guidance, and agent identity files
- Context-window pressure in long-running or multi-turn sessions
- Cost-sensitive 24/7 production systems where static context is a recurring tax

## When Not to Use It

- Prompts containing critical negations or safety constraints that must be unambiguous on first read
- Exact values, API paths, conditional logic, or precise sequences
- Files under ~2,000 tokens — fidelity risk exceeds savings
- Content intended for humans to review — PC output is model-readable, not human-readable

---

## Recommended Workflow

1. Keep `.md` files as your canonical source of truth — human-readable, version-controlled, editable.
2. Generate a matching `.pc.md` variant for runtime injection.
3. Preload `prompt_cloud_compression.md` once at session start (or bake it into your base system prompt).
4. Validate by behavior, not text diff — run the same task inputs against both variants and compare output. If the agent behaves identically, the compression is valid.
5. Never ship a `.pc.md` without behavioral validation.

---

## Version Tracking

A `.pc.md` file is only valid as long as its source `.md` has not changed. Every compressed file must include a hash header:

```
<!-- pc-source: path/to/prompt.md -->
<!-- pc-hash: a1b2c3d4 -->
```

The hash is the first 8 characters of the SHA-256 of the source file's contents. Before loading a `.pc.md`, compare the stored hash against the current source. Stale hash: fall back to the original and regenerate. Treat compressed files as derived caches — never as replacements for the canonical source.

---

## Honest Expectations

**Lossy, not lossless.** PC preserves functional equivalence — the model activates the same behavioral patterns from the compressed form as it would from the original. For behavioral context and system directives, functional equivalence is what matters. This is not a guarantee of textual identity.

**Fidelity scales inversely with specificity.** The more a file relies on exact phrasing, precise sequences, or conditional logic, the less it benefits from PC. The more it is behavioral shaping, the better it compresses.

**Compression quality varies by compressor.** A model compressing its own system prompt understands the semantic overlaps better than a model compressing someone else's domain-specific content. When possible, compress in the same context where the prompt was authored.

**Test by behavior.** Diffing the compressed form against the original tells you nothing useful. Running equivalent queries against both and comparing output tells you everything.

---

## Files in This Repo

| File | Purpose |
|------|---------|
| `prompt_cloud_compression.md` | The reusable PC directive — preload this at session start |
| `examples/role-spec.md` | Example behavioral role prompt (before) |
| `examples/role-spec.pc.md` | Same prompt, PC-compressed (after) |
| `LICENSE` | Apache 2.0 |

---

## Two Applications in Production Agent Systems

This repo covers static context compression. A companion application — **Memory Intent Compression** — uses the same technique at the memory write path to encode impressionistic observations with full situational context, so the receiving model reconstructs a richer operational picture weeks later. That application produces encodings that are sometimes *larger* than the original, because the goal is semantic density rather than token reduction.

Both applications are documented in the [production findings writeup](https://github.com/swbratcher/prompt-cloud).

---

## License

Apache 2.0. See [LICENSE](LICENSE).

---

*Technique by S. W. Bratcher. Production data from the Hegematon System — [github.com/swbratcher/hegemon](https://github.com/swbratcher/hegemon).*
