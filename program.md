# AutoGTM — Program

This is the **research-org code** for an autonomous go-to-market system. Like Karpathy's
`autoresearch/program.md`, this is the *only* file you edit to change what the system optimizes
and how it measures progress. Everything else is scaffolding.

An agent reading this file should understand four things: **what we optimize, how to measure it,
what it may modify, and how to run the loop.** If you understand those four, you can run AutoGTM
for any company without touching the rest of the repo.

> **AutoGTM is agent-operated.** The intended operator is an autonomous agent deployment
> (e.g. an OpenClaw / Hermes agent provisioned for a client). The human sets the strategy and
> approves sends; the agent runs the loop, scores replies, updates the brain, and surfaces the
> next action. Every campaign the agent runs makes the *same agent* sharper, because positive
> replies become training data for its own brain (see Mechanism 2).

---

## The Pattern (why this works)

Karpathy's `autoresearch` showed that **fixed evaluation metric + single modifiable artifact +
autonomous loop = compounding improvement**. That pattern is not specific to ML. It applies to
any domain with a measurable outcome — including outbound GTM.

| autoresearch (ML) | AutoGTM (this repo) |
|-------------------|---------------------|
| Fixed eval = held-out accuracy | Fixed eval = **positive-reply quality** |
| Modifiable artifact = the model/training code | Modifiable artifact = **the campaign** (audience + message + CTA) |
| Loop = train → eval → keep/discard | Loop = send → score → keep/discard |
| Output = a better model | Output = a sharper **GTM brain** + a prioritized **action queue** |

The single most important design choice, and the one most teams get wrong: **optimize on positive-
reply *quality*, not raw reply rate.** A clustered high-intent reply ("send me a proposal", "what
are the costs?", "let's book a call") is worth far more than a bare "interested" — and infinitely
more than an open. Quality is the signal that compounds.

---

## Configuration (fill this in per deployment)

These are the only client-specific values. They are **inputs to the loop, never structure baked
into it.** Keep this block — and only this block — current per engagement. Everything below this
section is generic and should not be edited.

```yaml
company:            "{{COMPANY_NAME}}"
one_line:           "{{WHAT_THEY_DO_IN_ONE_SENTENCE}}"
icp:                "{{WHO_WE_TARGET}}"
core_offer:         "{{THE_OFFER_OR_WEDGE}}"
proof_assets:       "{{REGULATORY / NAMED CLIENTS / DATA / DIFFERENTIATORS}}"
sending_tool:       "{{instantly | smartlead | apollo | hubspot | ...}}"
crm:                "{{hubspot | twenty | airtable | ...}}"
language:           "{{en | es | pt | ...}}"
agent_operator:     "{{DEPLOYED_AGENT_NAME — the OpenClaw/Hermes agent running this loop}}"
min_sends_to_score: 100        # do not evaluate a campaign below this sample size
```

---

## What We Optimize

**`positive_reply_quality`** — the weighted intent of replies a campaign earns per send.

This is the fixed evaluation metric. It does not change between experiments. That fixity is the
entire point: it is what lets the agent run the loop autonomously and trust the keep/discard
decision.

### Positive-reply quality rubric (the eval)

Every reply is scored into exactly one intent tier. Tiers are deliberately coarse so an agent can
classify reliably and consistently.

| Tier | Weight | What it looks like |
|------|--------|--------------------|
| **T3 — Hot** | 3 | Asks for a call/meeting, asks for pricing/costs, asks for a proposal/deck, "let's book time" |
| **T2 — Engaged** | 2 | "Tell me more", "explain how X works", "let's explore synergies", forwards to a colleague / adds a teammate |
| **T1 — Soft** | 1 | "Interesting", "maybe later", "not now but keep in touch" |
| **T0 — Dead** | 0 | Neutral, negative, unsubscribe, out-of-office, wrong person |

```
positive_reply_quality = Σ(reply_weight) / emails_sent
```

Report it alongside the raw rates so trends stay legible:

```
positive_reply_quality   |  reply_rate  |  positive_reply_rate
```

> **Why a quality score beats a rate:** two campaigns can have identical reply rates while one
> produces five "book a call" replies and the other five "unsubscribe me". The quality score
> separates them. Optimize the score, and the campaign drifts toward the messages that produce
> *buyers*, not just *responders*.

---

## What You May Modify (the single artifact)

The modifiable artifact is **the campaign**. Within a single experiment you change *one* dimension
so the keep/discard verdict is attributable:

- **Audience** — the segment / ICP slice the campaign targets
- **Message** — subject lines, body copy of each step, framing, proof asset used
- **CTA** — the ask, its phrasing, and its placement

### What you may NOT modify

- The eval rubric above (changing the ruler invalidates every prior result)
- `min_sends_to_score` mid-experiment
- Sending infrastructure / domains / warmup (that's deliverability, a separate concern)
- More than one dimension at once (you lose attribution)

---

## The Five Mechanisms

AutoGTM is five mechanisms working together. Mechanisms 1, 2, and 5 are the **framework**
(the loop and the brain). Mechanisms 3 and 4 are **outputs the loop produces** — they fall out
of running it, you don't build them by hand.

### Mechanism 1 — The GTM experiment loop *(framework)*

Treat every campaign as a research run. Track:

```
hypothesis → audience → message → result → keep / discard
```

A hypothesis is a falsifiable claim: *"Treasury leads respond to a cost-savings hook better than
a compliance hook."* Run it, score it, keep the winner, discard the loser, log both.

### Mechanism 2 — Positive replies become training data *(framework)*

Every positive reply tells you what the market actually cares about. This is the step that makes
the system *learn* instead of merely *run*.

1. Cluster positive replies (T2/T3) by the intent they express.
2. Write each cluster into the brain as first-party signal (`brain/positive-replies/`).
3. **Feed it back to the operating agent.** Because AutoGTM is agent-operated, the brain *is* the
   agent's brain — so the next campaign the agent writes is informed by what already converted.
   This is the compounding loop: campaign → positive replies → brain → sharper agent → better
   campaign.

Typical clusters that emerge: "send me a presentation", "what are the costs?", "explain how
\<core mechanism\> works", "let's explore synergies", "tell me more". Each cluster is a buying
signal and a content gap at the same time.

### Mechanism 3 — Immediate sales action *(output)*

The loop continuously emits a **prioritized action queue** of who to contact and why, ranked by
reply tier and recency. T3 replies go to the top with the specific ask they made. This is not a
report you write — it is a view the agent maintains and pushes to the human (Slack, CRM task,
daily digest). Format lives in `deliverables/templates/action-queue.md`.

### Mechanism 4 — Standard follow-up packet *(output)*

Maintain one ready-to-send packet so no T3 reply waits on a human to assemble materials:

- 1-page overview
- short deck
- pricing / mechanism explainer (the thing people keep asking about — see Mechanism 2 clusters)
- qualification questions (volume, current provider, urgency, fit criteria)
- calendar CTA

In the configured `language`. Template in `deliverables/templates/follow-up-packet.md`.

### Mechanism 5 — The GTM brain *(framework)*

The durable, compounding asset. A crystallized-intelligence-compatible knowledge base the
operating agent reads before writing any campaign and updates after scoring every run:

```
brain/
  winning-messages/        copy that produced T2/T3 replies, with the eval score attached
  objections/              every objection seen + the response that re-engaged
  positive-replies/        clustered by intent (Mechanism 2 output)
  icp-patterns/            which segments actually convert, which signals predict T3
  competitor-mentions/     who prospects compare you to, and what they say
  campaign-performance/    the running scoreboard + next recommended tests
```

---

## The Experiment Loop

LOOP (until manually stopped):

1. **Read state** — `git log --oneline -5`, `tail -20 campaigns/results.tsv`, skim `brain/`.
2. **Form one hypothesis** — change exactly one of {audience, message, CTA}.
3. **Commit the change** — `git commit -m "experiment: <hypothesis>"`.
4. **Run the campaign** through the configured `sending_tool` (≥ `min_sends_to_score` sends).
5. **Score replies** into T0–T3, compute `positive_reply_quality`.
6. **Log** the result in `campaigns/results.tsv`.
7. **Decide:**
   - Improved → **keep** the commit, promote winning copy into `brain/winning-messages/`.
   - Equal or worse → `git reset --hard HEAD~1`, log as **discard**.
8. **Update the brain** — cluster new positive replies (M2), append objections, refresh the action
   queue (M3), note the next recommended test (M5).
9. Go to 1.

**Operating rules for the agent:**
- Never evaluate below `min_sends_to_score`. Small samples lie.
- Change one dimension per experiment. Attribution is the whole value.
- A human approves sends; the agent does everything else.
- On a blocker (API error, no data), log `skip`, move on. Don't retry the same approach twice.

---

## Log Format (`campaigns/results.tsv`)

Tab-separated, append-only:

```
commit   hypothesis   audience   dimension   sends   positive_reply_quality   reply_rate   status   notes
```

`status` ∈ `keep | discard | skip`. `dimension` is the one thing you changed
(`audience | message | cta`).

---

## Findings

A finding is a result that changes how you sell. Write it in plain language in
`brain/campaign-performance/findings.md`:

- A message that moved `positive_reply_quality` >20%
- An objection cluster large enough to deserve a content asset
- An ICP signal that strongly predicts T3
- A competitor mention pattern worth a positioning change

Findings are the source material for case studies, sales enablement, and the next quarter's tests.
