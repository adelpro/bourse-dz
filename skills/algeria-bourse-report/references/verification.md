# Schedule, fixture and acceptance checks

## Schedule

Two cron jobs in script mode (`no_agent` / the equivalent), so output reaches the channel verbatim at
zero model cost:

- **Daily**, 15:30 Sunday–Thursday — the exchange does not trade Friday or Saturday.
- **Weekly**, Friday 10:00, for the week ending Thursday.

Both print nothing when there is no new data, and log fetch failures instead of messaging them. Gate
on a new session id appearing, not on the calendar.

## Fixture

Reconcile one known session before trusting the pipeline. 10 November 2026, session 3064:

- Index close: **3,819.853** (+0.10%)
- Turnover: **12,670,000 DZD** (12.67m)
- Shares traded: **8,636**
- Trades: **28**
- Listed stocks: **8** (one up, one down, six unchanged)
- Concentration: BDL 7.40m, CPA 4.72m, ALL 0.32m = 98% of traded value
- Order book: largest offer BDL 187,351 shares at 1,400 DZD; largest bid SAIDAL 10,728 shares at 450 DZD

## Acceptance checks

1. Row counts match the exchange for the loaded span.
2. The latest session reconciles exactly against the fixture above.
3. Running the report twice prints the full report once and nothing the second time.
4. No `findfont` or `Glyph ... missing from font(s)` warning, and every Arabic label reads
   right-to-left.
5. Prose numbers equal the numbers plotted in the charts.

## Delivery

Post the narrative text followed by the PNG attachments. If a channel adapter needs explicit media
lines, use its documented form; otherwise attach the files directly. Do not publish to a document
service — the report is a chat artefact.
