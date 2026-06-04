# Loop — Outbound Copy

The primary AutoGTM loop. Optimizes `positive_reply_quality` by varying the **message**.

**Fixed eval:** positive-reply quality (rubric in [`../program.md`](../program.md)).
**Modifiable:** subject lines, body copy of each step, framing, proof asset, CTA phrasing.
**Not modifiable:** audience (that's `loop-icp`), sending infra, the eval rubric.
**Min sample:** `min_sends_to_score` from the config block.

**One experiment = one of:**
- New hook / angle on step 1
- Different proof asset (named client vs. data point vs. regulatory)
- Different CTA (call vs. resource vs. soft question)
- Tighter / looser personalization

Log to `../campaigns/results.tsv` with `dimension = message`. Keep winners into
`../brain/winning-messages/`.
