# bourse-dz — an agent skill for reporting on the Algiers stock exchange

A portable skill that turns an agent into a reporter for بورصة الجزائر (Bourse d'Alger / SGBV):
it pulls the official session numbers, compares them with history, and writes a narrative **Arabic**
report with four designed charts on a schedule.

No API keys. All sources are public exchange pages.

## What it produces

- A daily session report in Modern Standard Arabic, written as connected prose rather than bullets:
  index close and breadth, liquidity against the 5/20/60-session means, turnover concentration,
  a per-trade reading of activity, the official unfilled order book, and a near-horizon statistical
  band with an explicit "not investment advice" line.
- A weekly summary every Friday for the week ending Thursday.
- Four charts: index trend with a 20-session mean, daily turnover against its mean, a performance
  heatmap across four horizons, and the session's turnover by stock as a ranked lollipop.
- One market-mechanics note per session, rotated by date, so a reader learns to interpret volume,
  value, trade count and the fixing mechanism instead of just reading numbers.

## Structure

```
plugin.json                          Agent Plugins 1.0.0 manifest
.claude-plugin/plugin.json           Claude Code plugin manifest
.claude-plugin/marketplace.json      marketplace entry
skills/algeria-bourse-report/
  SKILL.md                           procedure, pitfalls, verification
  references/sources.md              endpoints, payloads, parsing traps, SQLite schema
  references/market-mechanics.md     how Algiers prices and settles
  references/narrative.md            paragraph structure and Arabic editing rules
  references/charts.md               chart conventions and matplotlib Arabic traps
  references/verification.md         schedule, fixture, acceptance checks
README.md
CHANGELOG.md
LICENSE
```

## Sources

Official exchange, French and Arabic: `https://www.sgbv.dz/fr` · `https://www.sgbv.dz/ar`

- Session archive — `POST /fr/histo_boc_ajax.php`
- Per-session quotes — `GET /?page=details_boc&id_sea=<id>&lang=fr`
- DZAIRINDEX history — `POST /fr/ajax_indice.php?lang=fr`
- Arabic company names — `GET /ar/?page=societe_cote`
- Unfilled order book — `GET /ar/?page=ordre`

Commentary: [تحليلات بورصة الجزائر Dz](https://www.facebook.com/AnalysesBourseDZ213/) — read for
qualitative order-book colour only; its figures are not used as data.

## Install

```bash
git clone https://github.com/adelpro/bourse-dz.git

# Hermes, or any agent that loads SKILL.md directories
cp -r bourse-dz/skills/algeria-bourse-report ~/.hermes/skills/research/

# skills.sh CLI
npx skills add adelpro/bourse-dz -a claude-code -a hermes-agent --copy -y
```

Claude Code and Agent Plugins clients load the repository directly through
`.claude-plugin/marketplace.json` and the root `plugin.json`.

## Validation

Checked with [skill-validator-omni](https://github.com/adelpro/skill-validator-omni): **50/51** —
agentskills.io, Anthropic, Hermes, skills.sh, Claude Code, Agent Plugins and multi-tool compatibility
all pass in full. The single failure is a Codex frontmatter rule that cannot hold at the same time as
the Hermes standard: Hermes requires `version`, `author` and `platforms`, while Codex's
`quick_validate.py` allows only `name`, `description`, `license`, `allowed-tools` and `metadata`.
Hermes is prioritised here; drop those three fields if you need Codex 12/12 instead.

## Requirements

- Python 3.12 with `pandas`, `matplotlib`, `sqlite3`
- `arabic-reshaper` and `python-bidi` for Arabic shaping in matplotlib
- Playwright with a system Chrome build — only for the Facebook page
- `ffmpeg` to downscale charts before inspecting them

## Notes

- The report is descriptive. It contains no recommendation and says so.
- Exchange endpoints are public but undocumented; markup can change without notice — the skill lists
  the parsing traps known at the time of writing.
- The market-mechanics section summarises rules published by the exchange and the explainer series
  على صفحة تحليلات بورصة الجزائر.

## Licence

MIT — see [LICENSE](LICENSE).
