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

This repository follows the standard skill layout: `skills/<name>/SKILL.md`.

```bash
git clone https://github.com/adelpro/bourse-dz.git
cp -r bourse-dz/skills/algeria-bourse-report ~/.hermes/skills/research/
# or point any agent that loads SKILL.md files at that directory
```

As a Claude Code plugin, load the directory containing `.claude-plugin/`.

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
