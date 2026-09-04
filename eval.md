# eval.md — the fixed evaluation harness

This file is AutoGTM's `prepare.py`. **The operating agent does not modify it.**

It is the ground-truth ruler: the three outbound outcomes, the one number used for keep/discard,
how identities are matched across channels, and the guardrails that can veto a "win." Changing
this file mid-lineage invalidates every prior row in `campaigns/results.tsv`. A human may version
it (start a new `autogtm/<tag>` branch) — an agent may not.

The loop in [`program.md`](program.md) *calls* this harness. It does not reinvent it.

---

## The three outcomes of outbound

Every attempt on every channel ends in exactly one of three observable states. A campaign that
only measures (2) is guessing.

| # | Outcome | What happened | How you know | Role |
|---|---------|---------------|--------------|------|
| **1. Reach** | The message was deliverable. Eyeballs are *possible*. | Delivered / accepted / not bounced. You still cannot prove they *read* it. | **Guardrail.** Prerequisite. Never the eval. |
| **2. Reply** | They answered on the same channel: yes, no, or unsubscribe. | Inbox, LinkedIn reply, DM thread. | **Eval input** (T0–T3). |
| **3. Silent self-serve** | They did not reply. They researched you elsewhere (Google, an LLM, LinkedIn, your site) and acted. | Identity match onto site / CRM / product within the attribution window. | **Eval input** (S1–S3). |

> **The uncomfortable fact about (1):** there is no honest lead-level "they saw it" signal.
> Opens are unreliable; pixels are blocked; LinkedIn does not tell you a profile was stared at.
> Treat reach as *deliverability hygiene* — bounce, spam, accept, send-success — and refuse to
> score a campaign whose reach is broken. Then optimize (2)+(3) among people you actually reached.

A campaign with a low reply rate and a high silent-serve rate is not a failed campaign. It is a
campaign the old ruler would have discarded. That is the bug this harness exists to kill.

---

## The one number (keep / discard)

**`attributed_intent_quality`** — weighted buying intent per *eligible* attempt.

```
attributed_intent_quality = Σ(intent_weight) / eligible_attempts
```

- **`eligible_attempts`** = attempts that cleared the reach guardrail on their channel
  (delivered email, accepted/sent LinkedIn message, successful DM). Bounces and failed sends
  do not enter the denominator — they are a guardrail failure, not a quality failure.
- **`intent_weight`** = the single highest weight that identity produced in the attribution
  window. **Do not add reply + silent events for the same person.** A T2 reply who then books
  on the site is a 3, not a 5.

This is the only number the keep/discard verdict reads. Everything else is supporting.

### Reply weights (outcome 2)

Score every reply into exactly one tier. Tiers are coarse on purpose so an agent classifies
consistently across channels (an email "let's book time," a LinkedIn "send me times," and a
DM "what's pricing" are the same tier).

| Tier | Weight | What it looks like |
|------|--------|--------------------|
| **T3 — Hot** | 3 | Asks for a call/meeting, pricing, a proposal/deck, "let's book time" |
| **T2 — Engaged** | 2 | "Tell me more", "how does X work", forwards to a colleague, asks a real question |
| **T1 — Soft** | 1 | "Interesting", "maybe later", "keep in touch" |
| **T0 — Dead** | 0 | Neutral, negative, unsubscribe, out-of-office, wrong person |

Unsubscribe and explicit "never contact me" are T0 **and** a guardrail event (`negative_rate`).

### Silent self-serve weights (outcome 3)

Only count events that **match a sent identity from this experiment** (see Identity). No match,
no credit — brand-level traffic is not an experiment result.

| Tier | Weight | What it looks like |
|------|--------|--------------------|
| **S3 — Converted** | 3 | Demo request, meeting booked from site, pricing form, product signup, inbound email from a sent account |
| **S2 — High-intent research** | 2 | Session on pricing / demo / docs / comparison pages; repeated return visits |
| **S1 — Touched** | 1 | Matched site visit, LinkedIn profile view of the sender (if the channel gives it), self-identified return |

If the same identity also replied, keep `max(reply_weight, silent_weight)` once.

### What you report next to the eval (never vote)

```
attributed_intent_quality | reply_rate | silent_serve_rate | delivery_rate | negative_rate
```

`positive_reply_quality` (Σ reply weights / eligible_attempts, ignoring silent events) is still
logged so you can see whether a win came from talking or from self-serve. It does **not** decide
keep/discard.

---

## Identity (how channels connect)

Outbound is not one mailbox. The same person may get an email, ignore it, Google you, open
LinkedIn, read the site, and book — never hitting reply. Connecting those dots is the harness.

Every attempt writes one row to `campaigns/ledger.tsv` with a stable **identity_key**:

| Channel | identity_key (first that exists) |
|---------|----------------------------------|
| email | lowercase email, else company domain |
| linkedin | LinkedIn profile URL (canonical, no tracking params), else email, else domain |
| dm | handle (`platform:user`), else email, else domain |

Rules:

1. One identity_key per person per experiment. If you later learn the LinkedIn URL for an
   emailed lead, **merge** — do not create a second identity.
2. Credit silent events to the **experiment** that last touched them inside the window, not
   to whichever channel the conversion arrived on. Channel is a dimension you test; it is not
   a separate funnel.
3. Cross-reference by identity_key and entity ID, never by "the Acme guy."
4. No PII in `brain/` beyond what a private deployment already stores. This public repo ships
   empty ledgers.

---

## Reach guardrails (outcome 1)

A run that lifts `attributed_intent_quality` but breaches a guardrail is a **discard**.

| Channel | Pass when | Fail (discard) when |
|---------|-----------|---------------------|
| email | `delivery_rate` ≥ configured floor | bounce, spam-complaint, or unsubscribe rates breach the floor |
| linkedin | send/accept success ≥ configured floor | restriction warnings, failed sends, or block signals |
| dm | send-success ≥ configured floor | platform blocks, failed sends |

Defaults (override in the `program.md` config block, not here):

```
min_delivery_rate:          0.95
max_bounce_rate:            0.03
max_spam_complaint_rate:    0.001
max_unsubscribe_rate:       0.01
max_negative_reply_rate:    0.03
```

You may **not** "fix" a guardrail breach by changing copy *and* infrastructure in the same
experiment. Infrastructure (domains, warmup, accounts, proxies) is out of scope for the
campaign artifact — it is a separate concern, like `prepare.py`'s constants.

---

## When you are allowed to score

Do not score early. Replies lag. Silent self-serve lags more.

1. **Refuse** if `eligible_attempts` < `min_sends_to_score`. Small samples lie.
2. **Provisional** (status `pending`) once the send is done and some replies are in, but the
   attribution window is still open. Log the row; do **not** keep/discard yet.
3. **Final** when `attribution_window_days` have passed since the last send of the experiment
   (default 14). Compute `attributed_intent_quality`, check guardrails, then keep or discard.

The first run of a new branch is always the **baseline**: ship the current campaign unchanged,
score it, log it as `keep`. Every later experiment is compared to the current best on that branch.

---

## What the agent may not do

- Edit this file.
- Change the rubric, the weights, the no-double-count rule, or `min_sends_to_score` mid-experiment.
- Count unmatched brand traffic, newsletter signups from ads, or "we think they saw it" as S1–S3.
- Optimize reply rate, open rate, or connection-accept rate. Those are diagnostics and guardrails.
- Change more than one campaign dimension per experiment (see `program.md`).
