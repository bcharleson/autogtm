# Loop — Objection handling

Not a send-loop — a **mining loop**. Turns negative, stalled, and "no thanks" replies into
reusable assets.

**Input:** every T0/T1 reply that contains a reason ("too expensive", "already use X",
"not now", "need approval").
**Process:**
1. Cluster objections by type (`objection_class`).
2. For each cluster, draft the response that re-engages — then test it as a **message**
   experiment on the next occurrence (one dimension: message).
3. Record the objection + the response that worked in `brain/objections/` (`OBJ-`).

**Why it matters:** objection clusters are content gaps. A large enough cluster justifies an
asset (explainer, comparison, calculator) that pre-empts the objection — which raises
`attributed_intent_quality` upstream, including for people who will never reply and will
self-serve that asset on the site.

Large clusters graduate into an `FND-` in `brain/campaign-performance/`.
