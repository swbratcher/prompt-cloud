Prompt-Cloud Intent Compression
================================

For when intent matters more than precision.

Prompt-Cloud (PC) is a human-authored semantic compression technique for LLM prompts. It fuses overlapping concepts into dense compound tokens that activate the same behavioral patterns as the original verbose prompt — at 40-60% fewer tokens.

This is not an algorithmic tool. There is no library to install. PC is a tactic: a set of rules that teaches a model how to compress, decompress, and maintain prompt files in a paired .md / .pc.md format.


What's in this repo
-------------------

prompt-cloud-compression.md   
The directive. Preload this into any session where you want the model to compress or consume PC content. Covers compression rules, decompression rules, version tracking, workflow, and memory applications.

This README.txt

Other Examples and Stats that are my findings and estimates. 


How it works
------------

Natural language carries a lot of structural weight — articles, prepositions,
conjugations — that adds tokens but not meaning for an LLM. PC strips that
scaffolding and bundles semantically overlapping concepts into dense lowercase
compound strings. The model pattern-matches these clusters against the same
semantic space the original would have activated.

The result is functionally equivalent behavior at a fraction of the token cost.


When to use it
--------------

- System prompts and pre-context that load repeatedly across sessions
- Role definitions, behavioral directives, tone and style guidance
- Agent memory (impressionistic tier — see the directive for details)
- Any context where you're budget-constrained and the content is intent-shaped


When not to use it
------------------

- Exact values: numbers, dates, URLs, API paths, model strings
- Negations and exclusions that can't afford ambiguity
- Conditional logic with specific branching
- Content under 2k tokens (not worth the tradeoff)
- Anything humans need to read directly


Workflow
--------

1. Human writes and edits the .md source file.
2. Model generates the .pc.md from the source, including a SHA-256 hash header.
3. Model checks the hash before using the .pc.md. Stale hash = regenerate.
4. Model consumes the .pc.md, decompressing internally at inference.

The human never edits .pc.md files. The model never edits .md source files
without explicit instruction.


What this is not
----------------

This is not related to LLMLingua, CompactPrompt, or other algorithmic prompt
compression tools that use smaller models to prune tokens automatically. Those
operate at the token-information level. PC operates at the semantic-intent
level through human-authored concept fusion. Different technique, different
use case, complementary when needed.


Origin
------

Developed by KD (Kriger Danes LLC). No dependencies, no license restrictions
on the tactic itself. Use it however you want.
