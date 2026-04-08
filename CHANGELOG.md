# Changelog

All notable changes to Prompt-Cloud are documented here.

---

## [0.1.0] — 2026-04-08

### Added
- `prompt_cloud_compression.md` — the core PC directive for preloading at session start
- `examples/example-SOUL.md` + `example-SOUL.pc.md` — before/after pair for a personal agent identity file (~39.8% reduction)
- `examples/example-role-spec.md` + `example-role-spec.pc.md` — before/after pair for a behavioral role specification (~43.0% reduction)
- `examples/README.md` — token accounting and explanation for each example
- `RESULTS.md` — production data from a measured 24/7 deployment, including cross-system estimates for OpenClaw and Hermes Agent
- `README.md` — technique overview, production results, taxonomy, workflow, and honest expectations
- Apache 2.0 license

### Production basis
Initial release backed by measured data from a 24/7 agent system (Claude Opus, 1M context) running at 95 interactions/day + 48 scheduled sessions. Measured session-level reduction: 35.8%. Conservative daily savings: ~103,000 tokens.

---

*Format follows [Keep a Changelog](https://keepachangelog.com/). Versions follow [Semantic Versioning](https://semver.org/).*
