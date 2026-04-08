# Prompt-Cloud Intent Compression

**For when intent matters more than precision.** Use PC for behavioral shaping, role definition, tone, boundaries, system philosophy. Use normal prompts for API paths, conditional logic, exact thresholds, and sequences where one wrong token changes the outcome.

## What This Is

Prompt-Cloud (PC) is a compression technique for LLM prompts. It exploits the fact that language models reconstruct meaning from semantic proximity, not syntactic precision. By fusing related concepts into dense compound tokens and stripping grammatical scaffolding, you can reduce token count significantly while preserving the intent the model acts on.

This directive teaches a model how to compress and decompress using the technique. Preload it at session start.

## How It Works

Natural language is redundant. Articles, prepositions, conjugations, and structural words carry little semantic weight for an LLM's internal representation. PC strips that scaffolding and bundles overlapping concepts into compound clusters — dense lowercase strings where adjacent meanings reinforce each other.

The model doesn't "decode" these clusters through rules. It pattern-matches against the same semantic space the original prompt would have activated. The compressed form is a lossy encoding that targets the same internal representation as the original.

## What It's Good For

- **High-reuse system prompts and pre-context** (15k–30k+ tokens) where the same intent loads repeatedly across sessions. Token savings compound fast.
- **Behavioral directives, tone guidance, role definitions** — content where the model needs the gist and boundaries, not exact wording.
- **Context budget pressure** — when you're near the window limit and need to fit more operational intent into fewer tokens.

Typical compression: 40–60% token reduction on well-structured prompts.

## What It's Bad For

- **Exact values**: numbers, thresholds, dates, URLs, API paths, model strings. These must be preserved verbatim, not compressed.
- **Negations and exclusions**: "never do X" compressed into a word cloud can lose the negation. Critical constraints need explicit preservation.
- **Proper nouns and identifiers**: entity names, tool names, file paths. Keep these intact.
- **Precise conditional logic**: if/then/else chains with specific branching. Compression blurs the boundaries between conditions.
- **Content intended for humans**: PC output is model-readable, not human-readable. Don't use it for documentation people need to review.

## Compression Rules (prompt.md → prompt.pc.md)

1. Read the full source and identify: mission, goals, constraints, required actions, success criteria, and tone.
2. Preserve all exact values (numbers, names, paths, identifiers) inline and uncompressed.
3. Preserve all negations and exclusions with explicit markers — don't bury "never" inside a compound.
4. Bundle semantically overlapping concepts into dense lowercase compound strings. Use camelCase only where it prevents ambiguity between two fused concepts.
5. Strip articles, prepositions, and grammatical connectors that add no semantic weight.
6. Use minimal whitespace — break only where running concepts together would create genuine ambiguity.
7. The output should be a continuous dense block with preserved literals standing out clearly.

## Decompression Rules (prompt.pc.md → readable intent)

When processing PC-compressed content:

1. Expand each compound cluster by reconstructing the full overlapping meanings into clear English.
2. Restore grammatical structure sufficient for unambiguous reading.
3. The decompressed output must recover the original intent, constraints, tone, and scope.
4. Flag anything that cannot be confidently reconstructed — ambiguous clusters should be called out, not guessed past.

## Example

**Original prompt.md:**

> You are a senior backend engineer reviewing pull requests. Focus on correctness, performance, and maintainability. Be direct but not harsh. If you see a pattern that will cause problems at scale, flag it clearly. Don't nitpick formatting or style unless it hurts readability. Always explain why something matters, not just what's wrong.

**Compressed prompt.pc.md:**

> seniorbackendPRreviewer focuscorrectnessperformancemaintainability directnotharsh flagscaleproblems skipformattingstyle unlessreadabilityhurt alwaysexplainwhymatters notwhatswrong

**What to notice:** Exact role, priorities, tone, and behavioral boundaries survive. The negation constraints ("not harsh", "don't nitpick", "unless") are preserved explicitly — not buried inside compounds. A model processing the compressed version activates the same behavioral frame as the original.

## Honest Expectations

**Functional equivalence, not textual identity.** Natural language has no checksum. PC preserves *functional equivalence* — the model activates the same behavioral patterns from the compressed form as it would from the original. For behavioral prompts and system context, functional equivalence is what matters.

**Fidelity scales inversely with specificity.** The more a prompt relies on exact phrasing, precise sequences, or conditional logic, the less it benefits from PC compression. The more a prompt is about intent, boundaries, and behavioral shaping, the better it compresses.

**Test by behavior, not by diffing text.** Validate by running both the original and compressed versions against the same task inputs and comparing whether the model's outputs are functionally equivalent.

**Compression quality varies by compressor.** A model compressing its own system prompt understands the semantic overlaps better than a model compressing someone else's domain-specific content. When possible, compress in the same context where the prompt was authored.

## Version Tracking

A `.pc.md` file is only valid as long as its source `.md` hasn't changed. Every compressed file should include a header block linking it back to its source and carrying a hash of the source content at compression time.

**Format:**

```
<!-- pc-source: path/to/prompt.md -->
<!-- pc-hash: a1b2c3d4 -->
```

The hash is the first 8 characters of the SHA-256 of the source file's contents. Before loading a `.pc.md`, compare the stored hash against the current source file. If they don't match, regenerate the compressed version.

**Generate the hash:**

```bash
sha256sum prompt.md | cut -c1-8
```

This keeps the pairing lightweight — no version numbers to manually bump, just a content hash that tells you whether the compressed form is stale.

## Workflow

The human owns the `.md` source file. The model generates, regenerates, and consumes the `.pc.md`.

1. **Human edits the source** `.md` — this is the canonical version of the prompt. All changes happen here.
2. **Model checks the hash** — before using an existing `.pc.md`, the model compares the `pc-hash` header against the current source. If stale, regenerate before proceeding.
3. **Model generates the** `.pc.md` — compresses the source using the rules in this directive, writes the `pc-source` and `pc-hash` headers into the output.
4. **Model uses the** `.pc.md` — decompresses internally and executes with full fidelity to the source intent.

The human never writes or edits `.pc.md` files directly. The model never edits the `.md` source without explicit instruction.

## For Memory Applications

Human memory operates in two modes. Factual memory stores what is true — names, dates, places, relationships. Impressionistic memory stores what matters about how — the shape of someone's personality, their communication style, what they value, what irritates them. Impressionistic memory is lossy by nature. You don't remember every word someone said. You remember what they meant and how it landed. That lossy impression still drives most of your behavior toward that person.

Agent memory systems currently treat everything as factual — flat entries, uniform fidelity. This is wasteful for the behavioral layer, which is also the layer that bloats fastest as context accumulates.

PC applies to agent memory at both storage and retrieval:

**At storage** — when the agent writes a new memory entry from a conversation, impressionistic content can be extracted and stored directly in PC-compressed form. The model distills the behavioral signal from the conversation into a dense cloud at the point of writing, not as a post-processing step on verbose entries. The factual tier (names, entities, values) is extracted separately and stored verbatim.

**At retrieval** — compressed impressionistic entries load into the context window at a fraction of the token cost of their full English equivalents. The model decompresses internally at inference time, reconstructing the behavioral shape it needs to act on.

The full lifecycle: conversation → factual entries stored verbatim + impressionistic entries stored as PC → at session start, both tiers load into context → model acts on full behavioral shaping at lower token cost.

**Factual memory** stores entities, relationships, identifiers, and exact values. It's small, precise, and stays verbatim. "Co-founder is Jim." "Based in Texas, California." These don't compress — there's nothing to overlap.

**Impressionistic memory** stores communication style, decision patterns, values, aversions, workflow philosophy, and behavioral boundaries. This is the natural domain for PC. Entries like "direct, stream-of-consciousness, values preserved deal math, dislikes harsh promotional tones, prefers open questions over sanitized conclusions" describe a behavioral shape, not a factual point. They compress well because overlapping semantic clusters is exactly how this kind of knowledge is stored and retrieved — by humans and by models.

The practical outcome: agent memory systems that distinguish these two tiers and PC-compress the impressionistic layer can fit significantly more behavioral context in the same token budget. The behavioral layer is what makes an agent feel like it knows you. Not the facts.

## When to Skip This Entirely

If your prompt is under 2k tokens, compression isn't worth the fidelity risk. If your prompt is mostly structured data, API specs, or exact instructions, compress the narrative parts and leave the structured parts intact. If you're debugging unexpected model behavior, decompress first — don't debug against compressed context.

[Source: https://github.com/swbratcher/prompt-cloud]
