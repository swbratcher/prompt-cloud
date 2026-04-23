# Contributing to Prompt-Cloud

Prompt-Cloud is a technique repo, not a codebase. Contributions are documentation, examples, and validated compression pairs — not pull requests fixing import errors.

---

## What's worth contributing

**New example pairs.** A before/after pair for a file type not yet covered. Good candidates: AGENTS.md, HEARTBEAT.md, a framework integration spec, a multi-agent coordinator prompt. Each pair needs a token accounting header (see existing examples for format) and a note on what content type it demonstrates.

**Compression rate data.** If you've run PC on your own agent files and measured the results, open an issue with the file type, token counts, and what percentage was verbatim. Real-world data from different deployment contexts is the most useful thing this repo can accumulate.

**Taxonomy additions.** Edge cases that don't fit cleanly into compressible or verbatim. If you found a content type that behaved differently than expected, document it in `docs/taxonomy.md` with the reasoning.

**Corrections.** If a claim in the README or RESULTS.md is wrong based on your measurements, open an issue. Include your file composition and what you observed.

---

## What to avoid

Don't open PRs that rewrite the compression technique itself without measured behavioral evidence. The technique is documented as-is because it has been validated against production behavior. Theoretical improvements without data aren't useful here.

Don't add algorithmic or automated compression tools. This repo is specifically about human-authored semantic compression — if you're building a tool that automates PC-style compression, link to it in an issue rather than adding it to this repo.

Don't modify `.pc.md` files directly. Compressed files are derived caches. If the source `.md` needs updating, update the source and regenerate the compressed form. Open a PR with both files changed together and the `pc-hash` updated.

---

## How to submit an example pair

1. Add `yourfile.md` and `yourfile.pc.md` to `examples/`.
2. Include the pc-hash header block in the `.pc.md` file:
   ```
   <!-- pc-source: examples/yourfile.md -->
   <!-- pc-hash: [first 8 chars of sha256 of source] -->
   <!-- pc-tokens-original: N -->
   <!-- pc-tokens-compressed: N -->
   <!-- pc-reduction: N% -->
   <!-- pc-validated: behavioral equivalence confirmed -->
   ```
3. Note in your PR what you validated against — what task inputs you ran against both versions and what you observed.
4. Update `examples/README.md` with a row for your new pair.

Token counts are approximate. Use your model's tokenizer or estimate at ~1.33 words/token for English prose. Exact counts aren't the point — the compression ratio and behavioral validation are.

---

## Opening issues

Issues are welcome for: data from your own deployments, edge cases in the taxonomy, questions about applying PC to a specific file type, and corrections to documented claims.

The repo is maintained by S. W. Bratcher. Response time is not guaranteed but issues are read.
