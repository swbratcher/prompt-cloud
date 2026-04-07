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

## Usage

```
# Load the tactic (once per session)
[paste this directive or include as pre-context]

# Compress
"Compress the following prompt using the Prompt-Cloud tactic."

# Use directly
[paste .pc.md content — model decompresses internally and executes]

# Validate (optional)
"Decompress this .pc.md back to full English, then I'll compare behavior."
```

## When to Skip This Entirely

If your prompt is under 2k tokens, compression isn't worth the fidelity risk. If your prompt is mostly structured data, API specs, or exact instructions, compress the narrative parts and leave the structured parts intact. If you're debugging unexpected model behavior, decompress first — don't debug against compressed context.
