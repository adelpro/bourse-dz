# How the Algiers bourse works, and how to read it

The numbers are only meaningful read through these rules. Reporting them naively produces a
misleading picture of a thin, fixing-priced market.

## Volume counts shares, value counts money

1,000 shares at 2,000 DZD is 1,000 shares and 2m DZD; the same 1,000 shares at 200 DZD is a tenth of
the value. Never use one as if it were the other, and never rank stocks by volume alone — a low-priced
stock dominates volume while carrying little money.

## Each executed quantity counts once

Not once for the buyer and once for the seller: 100 shares sold by one investor to another is a volume
of 100, not 200. This is why a trader's sense of "activity" runs ahead of the published figure.

## Volume only means something with the trade count

The same quantity in 200 small trades is diffuse interest; in one trade it is a block transfer or a
change of ownership. Compute `volume / nb_trans` and `valeur / nb_trans`, compare each against its
20-session mean, and state the gap in percent. A session with real turnover but a small average ticket
is a different event from one carried by a single block.

## Pricing is a fixing

Orders are confronted to reach a single price per security, so volume can rise while the close barely
moves: the quantities executed at the fixing price. High volume is therefore **not** evidence of
strong demand. Emit that caveat whenever turnover is real and the index moved less than about 0.5%.

## Activity is not quality

A busy stock is not a good company and a quiet one is not a weak one. Volume measures attention, not
value; the company's worth comes from its financials. Never let the prose imply a buy signal from
volume — the report carries the line `هذا استقراء إحصائي لسلوك سابق، وليس توقعاً ولا توصية استثمارية.`

## Non-traders carry their previous close forward

A stock with no trades keeps its last close. Say so explicitly rather than letting a flat price read
as a market verdict; on a thin session, several stocks not trading is itself the story.

## What to compute from one session

- Turnover against the 5/20/60-session means, in percent.
- Per-trade ticket: `volume / nb_trans` and `valeur / nb_trans`, against their 20-session means.
- Concentration: the top stocks' share of traded value, and which stocks had zero volume.
- Breadth: up / down / flat counts, written as Arabic words, never as `1/1/6`.
- Order-book pressure: total offered vs demanded from the official snapshot, plus the largest single
  bid and offer.

## Attribution

These rules summarise exchange practice and the explainer series published by the page
تحليلات بورصة الجزائر Dz (دقيقة استثمار). Cite that source when reproducing the mechanics in public.
