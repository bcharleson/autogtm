# Loop — Objection Handling

Not a send-loop — a **mining loop**. Turns negative and stalled replies into reusable assets.

**Input:** every T0/T1 reply that contains a reason ("too expensive", "already use X", "not now",
"need approval").
**Process:**
1. Cluster objections by type.
2. For each cluster, draft the response that re-engages — then test it on the next occurrence.
3. Record the objection + the response that worked in `../brain/objections/`.

**Why it matters:** objection clusters are content gaps. A large enough cluster justifies a
follow-up asset (explainer, comparison, ROI calculator) that pre-empts the objection in the next
campaign — which raises `positive_reply_quality` upstream.

Large clusters graduate into a finding in `../brain/campaign-performance/findings.md`.
