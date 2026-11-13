---
name: algeria-bourse-report
description: Use when producing a Bourse d'Alger daily report.
version: 1.0.0
author: Adel
license: MIT
metadata:
  hermes:
    tags: [bourse, algeria, sgbv, arabic, reporting, charts]
    related_skills: [algeria-bourse-data, data-visualization, facebook-page-watch]
---

# Bourse d'Alger periodic report

Build once, then let it run: each session the job pulls the day's official numbers, compares them with
history, writes an Arabic narrative report with four designed charts, and posts it. Nothing is
published to a document service — the channel is the delivery.

## When to use

Use when asked for a recurring or one-off report on بورصة الجزائر / Bourse d'Alger / SGBV: daily
session summary, weekly summary, market commentary with charts. Not for trading signals — the output
is descriptive and carries no recommendation.

## Tools

- `python3.12` with `pandas`, `matplotlib`, `sqlite3` — call this interpreter explicitly, the default
  `python3` may lack the stack.
- `python3.12 -m pip install --user --break-system-packages arabic-reshaper python-bidi` — Arabic
  shaping and bidi reordering for matplotlib.
- Playwright driving the system Chrome (`executable_path="/usr/bin/google-chrome"`) — only for the
  Facebook page. `curl` works on sgbv.dz, returns HTTP 400 on facebook.com.
- `ffmpeg` to downscale a PNG before inspecting it (vision calls time out on full-size charts).

## Sources of information

**Websites**

- Official exchange, French: `https://www.sgbv.dz/fr`
- Official exchange, Arabic: `https://www.sgbv.dz/ar`
- Session archive: `https://www.sgbv.dz/fr/histo_boc_ajax.php`
- Session detail: `https://www.sgbv.dz/?page=details_boc&id_sea=<id>&lang=fr`
- Index history: `https://www.sgbv.dz/fr/ajax_indice.php?lang=fr`
- Arabic company list: `https://www.sgbv.dz/ar/?page=societe_cote`
- Unfilled orders: `https://www.sgbv.dz/ar/?page=ordre`

**Facebook page**

- `https://www.facebook.com/AnalysesBourseDZ213/` — تحليلات بورصة الجزائر Dz: a daily reading of
  the session plus unfilled order-book depth, posted early afternoon after the close. Commentary
  only — quote it with a link, never treat its figures as data.

## Source details and traps

1. **Sessions of a year** — `POST /fr/histo_boc_ajax.php`, body `date=<year>&lang=fr&type=1`. Returns
   session id, date, index close, value, shares, trade count. Anchor the row regex on `<tr[^>]*>`: the
   markup emits `<tr >` and a bare `<tr>` pattern raises `IndexError`.
2. **Session quotes** — `GET /?page=details_boc&id_sea=<id>&lang=fr`. Per ticker: close, volume, value,
   trades. The page carries no date; join back to the session row. Drop header cells (`TITRE`, `CODE`,
   `COURS`) and any row with a non-numeric close.
3. **Index history** — `POST /fr/ajax_indice.php?lang=fr`, body
   `date_debut=dd-mm-yyyy&date_fin=dd-mm-yyyy`. Parse the Highcharts `data : [[epoch_ms, value], ...]`;
   history reaches 2008. Chart this series consistently rather than the value in the page header
   (they differ by a few points but agree on the % change).
4. **Arabic names** — `GET /ar/?page=societe_cote` → `ticker | name_ar | kind` (`BDL` → بنك التنمية
   المحلية). `ar/?page=ligne_societe` is the **bonds** table, not equities — do not look for tickers
   there.
5. **Unfilled orders** — `GET /ar/?page=ordre` → `name | ticker | price | size | بيع/شراء`. The
   exchange's own depth: prefer it, snapshot it with a timestamp, and treat the Facebook page as
   corroboration. On 10 Nov 2026 the two agreed line for line on a 187,351-share BDL offer at 1,400
   DZD, while the page's turnover figure was double the official one.
6. **Facebook** — load the page once per run, compare the newest permalink against saved state, open
   the permalink only when it changed (logged out, no history is scrollable). Always persist the post
   to `latest_post.json` so a run with nothing new can still quote it.

## Database

```sql
sessions   (id_sea PK, date, volume, valeur, nb_trans, fetched_at)
quotes     (id_sea, ticker, cours, volume, valeur, nb_trans, PK(id_sea, ticker))
index_daily(ts PK, date, value)
companies  (ticker PK, name_ar, kind_ar, updated_at)
orders     (snapshot_at, ticker, name_ar, price, size, side,
            PK(snapshot_at, ticker, price, size, side))
```

Backfill several years before building the report (2024-01 → Nov 2026 ≈ 440 sessions, 2,500 quotes,
1,000+ closes) and run the long pull in the background: it exceeds a foreground timeout.

## Market mechanics (read the numbers through these)

- **Volume counts shares, value counts money.** 1,000 shares at 2,000 DZD is 2m DZD; the same 1,000 at
  200 DZD is a tenth. Never substitute one for the other, never rank stocks by volume alone.
- **Each executed quantity counts once**, not once per side.
- **Volume needs the trade count.** Same quantity in 200 trades is diffuse interest; in one trade it is
  a block transfer. Compute `volume/nb_trans` and `valeur/nb_trans`, compare with their 20-session
  means, state the gap in percent.
- **Pricing is a fixing** — orders confronted to one price per security — so volume can rise while the
  close holds. High volume is not evidence of demand: emit the caveat when turnover is real and the
  index moved under ~0.5%.
- **Activity is not quality**; never imply a buy signal from volume.
- **Non-traders carry their previous close forward** — say so rather than reading a flat price as a
  verdict.

## Procedure

1. Fetch the new session, the index closes, the Arabic names and an order snapshot. Log failures; do
   not message them.
2. If no new `id_sea`, exit silently (an `--if-new` marker file).
3. Write Arabic MSA prose — connected paragraphs, never bullet lists — in this order:
   - **خلاصة الجلسة** — close and change, who moved it, breadth in words ("ارتفع سهم واحد وتراجع سهم
     واحد واستقرّت ستة أسهم"), qualifier calibrated to the real breadth.
   - **السيولة** — turnover, shares, trades, gap against the 5/20/60-session means in percent.
   - **التركّز** — top stocks' share of value, Arabic name with the ticker in brackets on first
     mention; list the stocks that did not trade.
   - **النشاط** — the per-trade reading and the fixing caveat when relevant.
   - **أوامر السوق** — official book first (largest bid, largest offer, total offered vs demanded and
     what it implies), then the page's reading as corroboration with its link.
   - **الأفق القريب** — 60-session daily log returns: central path, 80% band over five sessions,
     annualised volatility, share of up sessions, then `هذا استقراء إحصائي لسلوك سابق، وليس توقعاً ولا
     توصية استثمارية.`
   - **مفاهيم السوق** — one note per session, rotated by date, from: volume vs value, trade count,
     fixing, activity vs quality, quantity counted once.
4. Render four charts and post the text with them attached.
5. Weekly edition on Friday: the week ending Thursday — index change, week turnover against the prior
   week, week best and worst, one stacked chart.

Editing rules: every clause derivable from the data, no invented causes; one to ten as Arabic words,
 digits beyond; a neutral estimate as `عند مستوى الإغلاق الحالي تقريباً`, never `-0.0%`; strip trailing
periods before joining fragments with `؛` or you emit `… جزائري.؛`.

## Charts

Follow the `data-visualization` skill, not matplotlib defaults:

- Title states the finding (`سيولة الجلسة 12.7 مليون دينار، دون المتوسط بنسبة 71%`); subtitle carries
  range, unit and source.
- One accent for the story, grey for context: teal `#0E7490`, burnt orange `#C2410C`, context
  `#E5E7EB`/`#9CA3AF`, ink `#111827`. Never a red/green pair.
- No top/right/left spines, no vertical gridlines, faint horizontal grid, no bar borders, no boxed
  legend. Annotate only the last point and the reference line.
- Ranking as a lollipop (guide line, one dot, `value · share` beside it); bars start at zero; keep the
  `+`/`-` sign in every label so colour never carries meaning alone.
- About 2.4:1 for time series; titles 13.5 pt bold, labels 9–10 pt.
- The four figures: index line with a 20-session mean and last close annotated; 90 sessions of daily
  turnover with the mean dashed and the latest session in accent; a heatmap over 1 session / 1 week /
  1 month / 3 months with the value in every cell (`—` when empty); a lollipop of the session's
  turnover by stock, zero-volume stocks grey and labelled `دون تداول`.

**Arabic in matplotlib — three traps:**

1. `get_display(arabic_reshaper.reshape(text), base_dir="R")`. Without `base_dir="R"`, any title
   starting with a Latin run (a ticker, `DZAIRINDEX`) has its Arabic blocks reordered.
2. Register the font by file: scan `font_manager.findSystemFonts(fontext="ttf")` for `NaskhArabic`,
   `addfont()` it, use `FontProperties(fname=...).get_name()`. By-name lookup fails silently to boxes.
3. **Noto Naskh Arabic has no Latin glyphs** — always pass `fontfamily=[AR_FONT, "DejaVu Sans"]`, and
   the same list in `legend(prop={"family": ...})`.

## Schedule

Two script-mode cron jobs (output to the channel verbatim, no model cost):

- **Daily**, 15:30 Sunday–Thursday (the exchange does not trade Friday/Saturday).
- **Weekly**, Friday 10:00, for the week ending Thursday.

Both print nothing when there is no new data and log fetch failures instead of messaging them.

## Verification

- Row counts match the exchange for the loaded span.
- Reconcile the fixture — 10 Nov 2026 (session 3064): index 3,819.853, turnover 12.67m DZD, 8,636
  shares, 28 trades, 8 listed stocks.
- Run twice: full output once, silence the second time.
- No `findfont` or `Glyph ... missing from font(s)` warning; Arabic reads right-to-left on every chart.
- Prose numbers equal chart numbers.
