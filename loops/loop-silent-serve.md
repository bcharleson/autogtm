# Loop — Silent self-serve

Not a send-loop — a **matching and clustering loop**. Turns "they never replied" into training
data when they still researched you and acted.

**Input:** `campaigns/ledger.tsv` identities from this experiment, plus CRM / site / product
events inside `attribution_window_days`.
**Process:**
1. Match events onto `identity_key` (email, LinkedIn URL, domain, `platform:handle`). No match,
   no credit — brand traffic is not an experiment result.
2. Tier each matched identity S1–S3 per [`eval.md`](../eval.md).
3. Apply no-double-count: `weight = max(reply_tier, silent_tier)`.
4. Cluster S2/S3 by path (landing page, last-touch channel, CTA they used) → `brain/silent-conversions/` (`SCV-`).
5. If a path is large enough to deserve a page, CTA, or proof asset, write an `FND-` and test
   that asset as a **message** or **CTA** experiment next.

**Why it matters:** this is the difference between "outbound that gets replies" and "outbound
that creates buyers." A campaign with a quiet inbox and a full calendar is a keep. Without
this loop, AutoGTM would have discarded it.

**Requires:** analytics and CRM in the `program.md` config that can match sent identities. If
they cannot, say so and leave silent fields blank — do not invent S-tier events.
