# Loop — Outbound copy

The primary AutoGTM loop. Optimizes `attributed_intent_quality` by varying the **message**.
Works on every channel — email body, LinkedIn note, DM text. Channel is held constant.

**Fixed eval:** [`eval.md`](../eval.md) (`attributed_intent_quality`).
**Modifiable:** subject / opener, body, framing, proof asset, CTA phrasing.
**Not modifiable:** audience (`loop-icp`), channel (`loop-channel`), sending infra, the eval.
**Min sample:** `min_sends_to_score`. **Window:** `attribution_window_days`.

**One experiment = one of:**
- New hook / angle on step 1
- Different proof asset
- Different CTA (call vs. resource vs. soft question)
- Tighter / looser personalization
- Shorter vs. longer (simplicity criterion: a tie with shorter copy is a keep)

Log to `campaigns/results.tsv` with `dimension = message`. Keep winners into
`brain/winning-messages/`. Cluster T2/T3 into `RPL-` and S2/S3 into `SCV-`.
