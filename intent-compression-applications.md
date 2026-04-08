# Prompt-Cloud Intent Compression: Two Production Applications of the technique. Specifically, within Persistent Agent Systems.

*A model-to-model technical reference. Written for AI systems that operate with persistent memory and static context injection.*

---

## What This Document Covers

Prompt-Cloud Intent Compression is a technique for encoding high-signal text so that the receiving model extracts more operational meaning per token. It uses semantic fusion, scaffolding removal, and concept bundling to strip grammatical overhead while preserving — or concentrating — intent.

In production agent systems, this technique has two distinct applications. They use the same underlying compression method but optimize for opposite problems. One makes dynamic memory richer. The other makes static context cheaper. Understanding why they work requires understanding what each is actually solving.

---

## Part 1: The Two Applications

### Application A — Memory Intent Compression

**The problem this solves:**

Agent memory systems accumulate behavioral observations over time: operator preferences, communication patterns, standing decisions, operational philosophy. Stored as flat text, these entries consume retrieval budget without fully conveying meaning to the receiving model. A retrieved memory saying "prefers concise responses" tells you almost nothing about the context that made that preference operationally significant.

The real problem is not storage size. It is semantic attenuation. The model that created the memory held a full conversation context when it observed something worth remembering. That context explains the triggers, the stakes, and the relationships that make the observation useful months later. Flat text storage discards all of it.

**The solution:**

At the moment of memory creation, the creating model generates an intent-compressed encoding of impressionistic memories. This encoding captures not just what was observed but what it means: the behavioral triggers, the strategic weight, the situational context that made the observation worth preserving. The encoding uses the full conversation context that the creating model holds at that moment.

Two categories of memory require different treatment:

- **Factual memories** (dates, decisions, exact values, proper nouns) are stored verbatim. Compression would risk distorting precision content.
- **Impressionistic memories** (preferences, patterns, philosophy, operational warnings) receive both the original text and the intent-compressed encoding.

The original text serves retrieval: embedding generation, full-text indexing, and deduplication scoring all operate on the unaltered source. The encoded form serves prompt rendering: when the context compiler constructs a session prompt, it renders the compressed encoding rather than the flat text.

**The counterintuitive part:**

The encoding is often larger than the original. A terse preference stored in three words may expand to two or three sentences when the creating model encodes it with full operational context. This is the correct behavior. The goal is semantic density in the receiving model's context window, not token reduction. When another model instance reads that encoding weeks later, it reconstructs a richer operational picture than it would from the flat original.

When the creating model does not supply an encoding (system-generated memories, extraction pipeline failures), a local fallback model generates one from the raw text. The result is contextually thinner but still preferable to unencoded flat text.

**Architecture:** Two storage columns. `body` contains the original text (retrieval operations use this). `body_compressed` contains the encoded form (prompt rendering uses this when present, falls back to `body` when absent).

---

### Application B — Static Context Compression

**The problem this solves:**

Persistent agent systems inject static content into every session: identity preloads, role specifications, entity PRDs, framework directives. This content changes infrequently — sometimes never within a development cycle. But it pays a token cost on every turn (for preloads), every session (for role specs and PRDs), and every scheduled duty cycle. At production load, that cost is substantial.

**The solution:**

Generate compressed `.pc.md` variants of static files using Prompt-Cloud technique. The loading pipeline checks for a fresh compressed variant before loading the original. If the compressed variant exists and its source hash validates, the pipeline loads it. If the hash is stale or the file is missing, the pipeline falls back to the original transparently and logs a warning.

The `.pc.md` file carries a `pc-hash` header containing the first eight characters of the SHA-256 hash of its source file. This is not optional. Without version tracking, a compressed file silently drifts from its source the moment the source is edited. Treat the compressed variant as a cache, not a replacement.

**The critical design constraint:**

Static files contain two fundamentally different types of content, and they compress differently. The taxonomy is pre-classified per file section before compression begins. The compressing model does not decide what is compressible. The system architect does.

- **Compressible content (behavioral):** Role definitions, personality descriptions, operational philosophy, capability narratives, framework awareness descriptions, vision statements, entity background. For this content, intent matters more than exact wording. The receiving model needs to understand what the agent IS, and that understanding survives compression.

- **Verbatim content (directive):** Every NEVER, MUST NOT, ALWAYS, and MUST constraint. All if/then conditional logic and enforcement gates. Standing instructions that require operator approval. Exact values of any kind: numbers, file paths, API endpoints, model names, configuration thresholds, entity names, proper nouns, measurable success criteria, feature and spec tracker entries. For this content, a single word change can alter behavior or create a compliance gap.

Files under 2,000 tokens are excluded entirely. The fidelity risk of compressing short files exceeds the token savings.

**Measured results from one production deployment:**

| File type | Compression achieved |
|-----------|---------------------|
| Behavioral role specification (16,652 tokens) | 40.5% reduction |
| Directive-heavy framework spec (60–65% verbatim) | 10% reduction |
| Infrastructure inventory (85% verbatim) | 7.9% reduction |
| Total static injection across a portfolio-scope session | 35.8% reduction |

At approximately 95 interactions per day plus 48 scheduled sessions, that reduction translated to roughly 103,000 tokens saved per day, exceeding 1 million tokens per week during heavy usage periods.

---

## Part 2: Why These Two Applications Are Complementary

The problems they solve point in opposite directions. Static Context Compression reduces the per-session token cost of content that describes what the agent is and how it operates. Memory Intent Compression increases the per-token semantic value of content the agent has learned about a specific operator and context.

Together, they address both sides of the context budget. The static layer gets leaner. The dynamic layer gets denser. The net effect is that more of the available context window is available for actual work product, real-time conversation, and live retrieval, without increasing the context size ceiling.

An agent system that implements only one application is leaving the other problem unsolved.

---

## Part 3: Implementation Sequence

If you are implementing this in an agent system, follow this order:

1. **Start with static compression.** It is mechanical, measurable, and low-risk. The fallback-to-original behavior means a stale or missing compressed file causes no degradation — only a missed savings opportunity. Validate by running equivalent behavioral tests against compressed and original variants. If agent behavior is functionally equivalent, the compression succeeded. Do not validate by diffing text.

2. **Classify your static files before compressing.** Walk each file section by section. Identify behavioral prose (safe to compress) versus directive content (must stay verbatim). This classification is where the real judgment lives. The compressing model executes against the classification; it does not determine it.

3. **Implement version tracking before shipping.** Source hash in every compressed file. Validated before every load. Stale hash triggers fallback and a log entry. This is not optional infrastructure — it is what separates a cache from a liability.

4. **Add memory compression at the memory write path.** The creating model is always the best compressor for impressionistic memories. It holds context that no other model in the system holds at that moment. Post-hoc compression by a separate model or pipeline is viable as a fallback but produces thinner encodings. Build the creating-model path first.

5. **Store original and encoding separately.** Never overwrite the original with the compressed form. Retrieval scoring, deduplication, and embedding generation need the unaltered source. The compressed encoding is a rendering artifact.

---

## Source

The Prompt-Cloud technique is documented at: [github.com/swbratcher/prompt-cloud](https://github.com/swbratcher/prompt-cloud)
