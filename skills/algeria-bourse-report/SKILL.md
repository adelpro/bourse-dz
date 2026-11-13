---
name: algeria-bourse-report
description: Use when producing a Bourse d'Alger daily report.
version: 1.1.0
author: Adel Ben Yahia
license: MIT
platforms: [linux, macos, windows]
metadata:
  hermes:
    tags: [bourse, algeria, sgbv, arabic, reporting, charts]
    related_skills: [algeria-bourse-data, data-visualization, facebook-page-watch]
---

# Bourse d'Alger periodic report

Build once, then let it run: each session the job pulls the day's official numbers, compares them with
history, and writes a narrative Arabic report with four designed charts. Nothing is published to a
document service — the channel is the delivery. Output is descriptive only: no recommendation, no
forecast dressed as advice.

## When to Use

Use when asked for a recurring or one-off report on بورصة الجزائر / Bourse d'Alger / SGBV: a daily
session summary, a weekly summary, or market commentary with charts. Trigger words include the
exchange name, the index `DZAIRINDEX`, and its tickers (`BDL`, `CPA`, `BIO`, `ALL`, `SAIDAL`).

Do not use it for live trading signals, portfolio advice, or any market other than Algiers.

## Procedure

1. Confirm the interpreter carries `pandas`, `matplotlib` and `sqlite3` (often `python3.12`, not
   `python3`), and that `arabic-reshaper` and `python-bidi` are installed.
2. Pull the data per `references/sources.md`: session archive, session quotes, index history, Arabic
   company names, the unfilled order book, and the commentary page. Store it in SQLite using that
   file's schema, backfilling several years before the first report.
3. If no new session id appeared, stop and print nothing.
4. Write the prose following `references/narrative.md`: session summary, liquidity, concentration,
   activity, order book, near-horizon band, rotating market-mechanics note.
5. Interpret the numbers through `references/market-mechanics.md` — volume vs value, trade count, the
   fixing mechanism — never as a naive reading of price alone.
6. Render the four charts to `references/charts.md`, then post the text with the PNGs attached.
7. Schedule the daily run at 15:30 Sunday–Thursday and the weekly run Friday 10:00, both in script
   mode so output goes out verbatim. Details and the fixture are in `references/verification.md`.

## Pitfalls

- **A bare `<tr>` regex fails on this site.** The session table emits `<tr >`; anchor on `<tr[^>]*>`
  or you get `IndexError: list index out of range`.
- **`details_boc` carries no date.** Join the quotes back to the session row rather than reading a
  date off the page.
- **Two index values exist.** The page header and the Highcharts feed differ by a few points while
  agreeing on the percentage change; chart one series consistently and say which.
- **`/ar/?page=ligne_societe` is the bonds table.** Equity tickers and Arabic names come from
  `/ar/?page=societe_cote`.
- **Third-party figures are unreliable.** The commentary page once reported double the official
  turnover; use it for order-book colour only, and prefer `/ar/?page=ordre` for depth.
- **Arabic in matplotlib fails three ways** — unshaped text, reordered blocks when a title starts with
  a Latin run, and empty boxes for Latin glyphs. All three fixes are in `references/charts.md`.
- **Vision calls time out on full-size PNGs.** Downscale before inspecting; never hold delivery on a
  vision check.
- **`curl` is blocked on facebook.com.** That page needs a browser engine; `curl` is fine on sgbv.dz.

## Verification

- Row counts match the exchange for the span loaded, and one known session reconciles exactly against
  the fixture in `references/verification.md`.
- The report prints in full on the first run, and prints nothing on a repeat run of the same session.
- No `findfont` or `Glyph ... missing from font(s)` warning, and every Arabic label reads
  right-to-left.
- Prose numbers equal the numbers plotted in the charts.

## References

- `references/sources.md` — endpoints, payloads, parsing traps, SQLite schema
- `references/market-mechanics.md` — how Algiers prices and settles, and how to read volume
- `references/narrative.md` — paragraph structure and Arabic editing rules
- `references/charts.md` — chart conventions, palette, matplotlib Arabic traps
- `references/verification.md` — schedule, fixture and acceptance checks
