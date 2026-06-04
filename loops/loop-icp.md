# Loop — ICP / Audience

Optimizes `positive_reply_quality` by varying the **audience**, holding the message fixed.

**Fixed eval:** positive-reply quality (same rubric — never changes).
**Modifiable:** which segment / firmographic slice the campaign targets, signal filters.
**Not modifiable:** the message (that's `loop-outbound`), the eval rubric.

**Goal:** discover which segments actually produce T3 replies, and which signals predict them.
Hold a known-good message constant, run it against segment A vs. segment B, compare quality.

Log to `../campaigns/results.tsv` with `dimension = audience`. Promote findings into
`../brain/icp-patterns/` — especially any signal that strongly predicts a hot (T3) reply.
