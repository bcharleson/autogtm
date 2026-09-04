# Loop — ICP / audience

Optimizes `attributed_intent_quality` by varying the **audience**, holding message and channel
fixed.

**Fixed eval:** [`eval.md`](../eval.md).
**Modifiable:** segment / firmographic slice, signal filters.
**Not modifiable:** the message (`loop-outbound`), the channel (`loop-channel`), the eval.

**Goal:** which segments produce T3 *or* S3, and which signals predict them. A segment that
does not reply but books from the site is a winning audience — do not discard it for a quiet
inbox.

Log with `dimension = audience`. Promote findings into `brain/icp-patterns/` (`ICP-`),
including signals that predict silent conversion.
