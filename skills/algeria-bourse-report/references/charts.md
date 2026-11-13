# Charts: conventions, palette and Arabic traps

Follow the `data-visualization` skill rather than matplotlib defaults. Four figures per daily report,
one stacked figure for the weekly.

## Conventions

- **The title states the finding**, not the topic: `سيولة الجلسة 12.7 مليون دينار، دون المتوسط بنسبة
  71%`, never `قيمة التداولات مقابل متوسطها`. The subtitle carries range, unit and source.
- **One accent for the story, grey for everything else**: teal `#0E7490` for the series, burnt orange
  `#C2410C` for the highlighted session, `#E5E7EB` and `#9CA3AF` for context, ink `#111827` for type.
  Never a red/green pair.
- **No chartjunk**: hide top/right/left spines, no vertical gridlines, faint horizontal grid only, no
  bar borders, no boxed legend, no background fill. Annotate only the last point and the reference
  line — never every point.
- **Ranking is a lollipop**: thin guide line, one dot, `value · share` set beside the dot. Where bars
  remain they start at zero. Keep the `+`/`-` sign in every label so colour never carries meaning
  alone.
- Aspect about 2.4:1 for time series, squarer for comparisons; titles 13.5 pt bold, labels 9–10 pt.

## The four figures

1. Index line with a 20-session mean dashed, the last close annotated, and the change in the title.
2. Daily turnover for the last 90 sessions, the 20-session mean as a dashed line, the latest session
   in the accent colour with its value labelled.
3. Performance heatmap over 1 session / 1 week / 1 month / 3 months, the value printed in every cell
   with its sign, empty cells as `—`.
4. The session's turnover by stock as a ranked lollipop, zero-volume stocks grey and labelled
   `دون تداول`.

Weekly: index closes and weekly turnover stacked, weeks ending Thursday.

## Arabic in matplotlib — three traps

1. **Shape and reorder**: `get_display(arabic_reshaper.reshape(text), base_dir="R")`. Without
   `base_dir="R"`, any string that starts with a Latin run (a ticker, `DZAIRINDEX`) has its Arabic
   blocks reordered and the title reads as nonsense.
2. **Register the font by file**: scan `font_manager.findSystemFonts(fontext="ttf")` for
   `NaskhArabic`, `addfont()` it, and use `FontProperties(fname=...).get_name()`. Asking for the family
   by name silently yields boxes.
3. **Noto Naskh Arabic has no Latin glyphs** — every string mixing Arabic with tickers, digits or `%`
   must pass a fallback list: `fontfamily=[AR_FONT, "DejaVu Sans"]`, and the same list in
   `legend(prop={"family": ...})`.

## Inspecting the output

Downscale before reading a PNG back: `ffmpeg -y -i chart.png -vf scale=680:-1 out.jpg`. Vision calls
time out on full-size charts, and delivery must never wait on one.
