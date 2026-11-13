# Narrative: structure and Arabic editing rules

Modern Standard Arabic, written as connected prose. Never bullet lists — a reader should be able to
follow the session from the text alone, and every clause must be derivable from the data.

## Paragraph order

1. **خلاصة الجلسة** — close and percentage change, who moved it, breadth in words
   ("ارتفع سهم واحد وتراجع سهم واحد واستقرّت ستة أسهم"). Calibrate the qualifier to the real breadth:
   with one riser and one faller say `تباين محدود`, not `واضح`.
2. **السيولة** — turnover, shares, trades, and the gap against the 5/20/60-session means in percent.
3. **التركّز** — the top stocks' share of traded value, naming companies in Arabic with the ticker in
   brackets on first mention (`بنك التنمية المحلية (BDL)`), then the stocks that did not trade.
4. **النشاط** — the per-trade reading: average shares and dinars per trade against their 20-session
   means, plus the fixing caveat when turnover is real and the index barely moved.
5. **أوامر السوق** — the official unfilled book first (largest bid, largest offer, total offered vs
   demanded and what it implies), then the commentary page's reading as corroboration with its link.
6. **الأفق القريب** — 60-session daily log returns: central path, 80% band over five sessions,
   annualised volatility, and the share of up sessions in comparable windows.
7. **مفاهيم السوق** — one short note per session, rotated deterministically by date, from: volume vs
   value, trade count, the fixing mechanism, activity vs quality, quantity counted once.

## Editing rules

- No invented causes. "Because of oil prices" is not in the data; omit it.
- Numbers one to ten as Arabic words (`_WORDS`), digits beyond that.
- A neutral estimate is `عند مستوى الإغلاق الحالي تقريباً`, never `-0.0%`.
- Strip trailing periods from fragments before joining them with `؛`, or the join emits `… جزائري.؛`.
- Every estimate is followed by `هذا استقراء إحصائي لسلوك سابق، وليس توقعاً ولا توصية استثمارية.`
- Breadth, concentration and per-trade figures are stated once each; do not repeat the same number in
  two paragraphs.
- The weekly edition covers the week ending Thursday: index change, week turnover against the prior
  week, week best and worst, then one chart.

## Tone

Descriptive and measured. Describe what happened and what it implies for reading the next session;
never advise, never predict a direction. A quiet session is reported as quiet, not as an opportunity.
