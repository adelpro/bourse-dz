# Sources, endpoints and parsing traps

All public, no login. `curl`/`urllib` work for sgbv.dz; facebook.com rejects `curl` with HTTP 400.

## Websites

- Official exchange, French: `https://www.sgbv.dz/fr`
- Official exchange, Arabic: `https://www.sgbv.dz/ar`
- Session archive: `https://www.sgbv.dz/fr/histo_boc_ajax.php`
- Session detail: `https://www.sgbv.dz/?page=details_boc&id_sea=<id>&lang=fr`
- Index history: `https://www.sgbv.dz/fr/ajax_indice.php?lang=fr`
- Arabic company list: `https://www.sgbv.dz/ar/?page=societe_cote`
- Unfilled orders: `https://www.sgbv.dz/ar/?page=ordre`

## Facebook page

`https://www.facebook.com/AnalysesBourseDZ213/` — تحليلات بورصة الجزائر Dz. Posts a daily reading of
the session plus unfilled order-book depth, usually early afternoon after the close. Commentary only:
quote it with a link, never take its figures as data.

## Endpoint by endpoint

1. **Sessions of a year** — `POST /fr/histo_boc_ajax.php`, body `date=<year>&lang=fr&type=1`. Returns
   session id, date (`dd/mm/yyyy`), index close, value traded, shares traded, trade count. Anchor the
   row regex on `<tr[^>]*>`: the markup emits `<tr >` and a bare `<tr>` pattern raises `IndexError`.
2. **Session quotes** — `GET /?page=details_boc&id_sea=<id>&lang=fr`. Per ticker: close, volume, value,
   trades. The page carries no date; join back to the session row. Drop header cells (`TITRE`, `CODE`,
   `COURS`) and any row whose close is not numeric.
3. **Index history** — `POST /fr/ajax_indice.php?lang=fr`, body
   `date_debut=dd-mm-yyyy&date_fin=dd-mm-yyyy`. Parse the Highcharts `data : [[epoch_ms, value], ...]`;
   history reaches 2008, so one wide call backfills everything. Chart this series rather than the
   value in the page header — they differ by a few points but agree on the percentage change.
4. **Arabic names** — `GET /ar/?page=societe_cote` → `ticker | name_ar | kind` (`BDL` → بنك التنمية
   المحلية). `ar/?page=ligne_societe` is the **bonds** table, not equities.
5. **Unfilled orders** — `GET /ar/?page=ordre` → `name | ticker | price | size | بيع/شراء`. The
   exchange's own depth: prefer it, snapshot it with a timestamp, and treat the page as corroboration.
   On 10 Nov 2026 both agreed line for line on a 187,351-share BDL offer at 1,400 DZD while the page's
   turnover figure was double the official one.
6. **Commentary** — load the Facebook page once per run, compare the newest permalink against saved
   state, and open the permalink only when it changed (logged out, no history is scrollable). Persist
   the post to `latest_post.json` so a run with nothing new can still quote it.

## Parsing oddities

- Thousands separator is a space, decimal comma (`1 400,00`) — strip spaces before `float()`.
- `NC` means no close (the stock did not trade): carry the previous close forward, do not zero it.
- Arabic tables use `<th>` headers, so parse with `<t[dh][^>]*>` and filter tickers with `[A-Z0-9]{2,8}`.
- Sessions run Sunday–Thursday, roughly 10:00–14:00 Algiers; gate on a new `id_sea`, not the calendar.

## SQLite schema

```sql
sessions   (id_sea PK, date, volume, valeur, nb_trans, fetched_at)
quotes     (id_sea, ticker, cours, volume, valeur, nb_trans, PK(id_sea, ticker))
index_daily(ts PK, date, value)
companies  (ticker PK, name_ar, kind_ar, updated_at)
orders     (snapshot_at, ticker, name_ar, price, size, side,
            PK(snapshot_at, ticker, price, size, side))
```

Backfill several years before building the report (2024-01 → Nov 2026 is roughly 440 sessions, 2,500
quote rows and 1,000+ index closes), and run the long pull as a background process: it exceeds a
foreground timeout. The daily update should refresh the session, the index, the Arabic names and an
order snapshot in one pass, logging failures rather than messaging them.
