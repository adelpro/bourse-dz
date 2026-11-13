# Changelog

All notable changes to this project are documented here.

## [1.1.0] - 2026-11-13

### Added

- Full plugin layout: root `plugin.json` (Agent Plugins 1.0.0) so Agent Plugins clients and Codex can
  install the skill from the repository.
- `references/` set — `sources.md`, `market-mechanics.md`, `narrative.md`, `charts.md`,
  `verification.md` — with the main SKILL.md trimmed to procedure and pitfalls.
- `platforms` in the skill frontmatter; `## Pitfalls` section.

### Changed

- `## When to use` renamed to `## When to Use`, and the body restructured to the Hermes in-repo
  standard (When to Use / Procedure / Pitfalls / Verification).
- Author normalised to `Adel Ben Yahia` across manifests.

## [1.0.0] - 2026-11-13

### Added

- Skill for producing a periodic Arabic report on the Algiers stock exchange (SGBV).
- Official sources: session archive, per-session quotes, DZAIRINDEX history, Arabic company names
  and the unfilled order book, plus commentary from the AnalysesBourseDZ213 page.
- Narrative structure: session summary, liquidity, concentration, activity, order book,
  near-horizon band, rotating market-mechanics note.
- Chart conventions: insight titles, single accent palette, lollipop ranking, Arabic shaping rules.
- Scheduling and verification guidance, including a fixture session for reconciliation.
