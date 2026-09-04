# AutoGTM — Program

This is the **research-org code** for an autonomous go-to-market system. Like Karpathy's
`autoresearch/program.md`, this is the *only* file a human edits to change what the system
optimizes and how it measures progress. Everything else is scaffolding.

An agent reading this file should understand four things: **what we optimize, how to measure it,
what it may modify, and how to run the loop.** If you understand those four, you can run AutoGTM
on any company, on any outbound channel, without touching the rest of the repo.

> **AutoGTM is agent-operated and runtime-agnostic.** Grok Bot, Hermes, OpenClaw, Claude Code,
> Cursor, Codex, or a custom loop can all run it. The operator reads this file, calls
> [`eval.md`](eval.md) as the fixed ruler (do not edit it), edits
> [`campaigns/current.md`](campaigns/current.md) as the single artifact, talks to the configured
> sending tools and CRM, and reads/writes `brain/`. The human sets strategy and **approves every
> send**. The agent runs the **G.E.A.R. loop** (Get → Evaluate → Author → Release), scores
> *replies and silent self-serve*, updates the brain, and surfaces the next action.

Point the agent at [`AGENTS.md`](AGENTS.md). Load a file from [`runtimes/`](runtimes/) only if
your host needs extra wiring.

---

## The Pattern (why this works)

Karpathy's `autoresearch` showed that **fixed evaluation metric + single modifiable artifact +
autonomous loop = compounding improvement**. That pattern is not specific to ML.

| autoresearch (ML) | AutoGTM (this repo) |
|-------------------|---------------------|
| Fixed eval = `val_bpb` in `prepare.py` (agent cannot touch it) | Fixed eval = **`attributed_intent_quality`** in [`eval.md`](eval.md) (agent cannot touch it) |
| Modifiable artifact = `train.py` | Modifiable artifact = **`campaigns/current.md`** (audience · message · CTA · channel) |
| Loop = train 5 min → keep/discard | Loop = send → wait the attribution window → keep/discard |
| First run = unchanged baseline | First run = unchanged baseline |
| Output = a better model | Output = a sharper **GTM brain** + an **action queue** |

The bet most outbound teams get wrong: they optimize **reply rate on one channel**. Real outbound
has three outcomes (reach, reply, silent self-serve). A prospect who never replies but Googles
you, asks a model about you, looks you up on LinkedIn, and books from the site is a *win*. The
eval in `eval.md` scores that win. Reply rate alone would have discarded it.

---

## Configuration (fill this in per deployment)

These are the only company-specific values. They are **inputs to the loop, never structure baked
into it.** Keep this block — and only this block — current. Everything below it is generic.

```yaml
company:            "{{COMPANY_NAME}}"
one_line:           "{{WHAT_THEY_DO_IN_ONE_SENTENCE}}"
icp:                "{{WHO_WE_TARGET}}"
core_offer:         "{{THE_OFFER_OR_WEDGE}}"
proof_assets:       "{{NAMED CLIENTS / DATA / DIFFERENTIATORS}}"

# One or more channels. The loop is the same on every channel; only the tool changes.
# type ∈ email | linkedin | dm
channels:
  - type: email
    tool: "{{your email sequencer or mailbox}}"
  - type: linkedin
    tool: "{{your linkedin sending tool}}"
  - type: dm
    tool: "{{slack | twitter | whatsapp | other}}"

crm:                "{{your crm}}"
analytics:          "{{site / product analytics — required for silent self-serve}}"
language:           "{{en | es | pt | ...}}"
agent_operator:     "{{grok-bot | hermes | openclaw | claude-code | cursor | codex | custom}}"

min_sends_to_score:        100   # do not finalize a campaign below this eligible sample
attribution_window_days:   14    # wait this long after last send before keep/discard
min_delivery_rate:         0.95
max_bounce_rate:           0.03
max_spam_complaint_rate:   0.001
max_unsubscribe_rate:      0.01
max_negative_reply_rate:   0.03
```

---

## What We Optimize

**`attributed_intent_quality`** — the full spec lives in [`eval.md`](eval.md). Summary:

```
attributed_intent_quality = Σ(intent_weight) / eligible_attempts
```

Each identity contributes **one** weight: the max of its reply tier (T0–T3) and its silent
self-serve tier (S1–S3). Eligible attempts are reached attempts (outcome 1 passed). Unmatched
brand traffic does not count.

Report next to it (these do not vote):

```
attributed_intent_quality | reply_rate | silent_serve_rate | delivery_rate | negative_rate
```

### The metric hierarchy

| Class | Examples | Role |
|-------|----------|------|
| **Primary (the eval)** | `attributed_intent_quality` | The **only** optimization target. Drives keep/discard. Never changes inside a lineage. |
| **Downstream validation** | `meeting_held_rate`, `qualified_opportunity_rate` | Lagging truth. Recalibrates the rubric on a quarterly human review, never inside the inner loop. Stored as `CNV-` entities. |
| **Diagnostic** | `reply_rate`, `silent_serve_rate`, `positive_reply_quality`, time-to-first-intent, step/channel attribution | Explain *why* a campaign moved. Tracked, not optimized. |
| **Guardrail** | `delivery_rate`, bounce / spam / unsubscribe / negative-reply rates, LinkedIn restriction warnings | Hard limits. A run that lifts the primary metric but **breaches a guardrail is a discard.** |

> Optimizing several metrics at once reintroduces the attribution problem the loop exists to
> kill. One target, gated by reach, explained by diagnostics, validated downstream.

---

## What You May Modify (the single artifact)

The modifiable artifact is **[`campaigns/current.md`](campaigns/current.md)**. Within a single
experiment you change *one* dimension so the verdict is attributable:

- **Audience** — the segment / ICP slice
- **Message** — subject, opener, body, creative, proof asset, voice (on any channel)
- **CTA** — the ask, its phrasing, its placement
- **Channel** — email vs LinkedIn vs DM (hold audience + message + CTA constant)

### What you may NOT modify

- [`eval.md`](eval.md) — the ruler
- `min_sends_to_score` or `attribution_window_days` mid-experiment
- Sending infrastructure / domains / warmup / accounts (reach is a guardrail, not a test)
- More than one dimension at once

**Simplicity criterion** (from autoresearch): all else equal, simpler is better. A tiny lift
from a hacky 800-word message is not worth it. A tie with a shorter message is a keep. Deleting
a step and holding quality is a keep.

---

## The Five Mechanisms

Mechanisms 1, 2, and 5 are the **framework**. Mechanisms 3 and 4 are **outputs the loop emits**.

### Mechanism 1 — The GTM experiment loop *(framework)*

Treat every campaign as a research run:

```
hypothesis → (audience | message | CTA | channel) → reach check → replies + silent events → keep / discard
```

A hypothesis is a falsifiable claim: *"A cost-savings hook beats a compliance hook for treasury
leads on email."* Or: *"The same message on LinkedIn produces more silent site-bookings than
email."* Run it, wait the window, score it, keep the winner, discard the loser, log both.

### Mechanism 2 — Intent becomes training data *(framework)*

Every T2/T3 reply **and** every S2/S3 silent event tells you what the market actually cares
about. This is the step that makes the system *learn* instead of merely *send*.

1. Cluster T2/T3 replies by the intent they express → `brain/positive-replies/` (`RPL-`).
2. Cluster S2/S3 silent events by the path they took (which page, which CTA, which channel
   touched them last) → `brain/silent-conversions/` (`SCV-`).
3. Feed both back to the operating agent. The brain the loop updates *is* the brain the agent
   reads — so the next campaign inherits what already converted, including people who never
   replied.

Typical reply clusters: "send me a deck", "what are the costs?", "how does it work?", "let's
explore." Typical silent clusters: "hit /pricing then booked", "came from LinkedIn, converted
on site", "asked a model then searched the brand." Each cluster is a buying signal and a
content gap at the same time.

### Mechanism 3 — Immediate sales action *(output)*

A live **action queue** of who to contact and why, ranked by intent tier and recency. T3 and
S3 at the top, each with the exact ask *or* the silent path they took. Push it to the human
(chat, CRM task, digest). Shape: `deliverables/templates/action-queue.md`.

### Mechanism 4 — Standard follow-up packet *(output)*

One ready-to-send packet so no T3/S3 waits on a human to assemble materials: one-pager, short
deck, the explainer people keep asking for, qualification questions, calendar CTA. In the
configured `language`. Shape: `deliverables/templates/follow-up-packet.md`.

### Mechanism 5 — The GTM brain *(framework)*

The compounding asset. Typed entity records the agent parses by field, not by prose. Full
ontology: [`brain/README.md`](brain/README.md).

```
brain/
  icp-patterns/            which segments convert; which signals predict T3/S3     (ICP-)
  intent-signals/          external GTM triggers that predict intent               (INT-)
  winning-messages/        copy that earned T2/T3 or S2/S3, with the eval score    (MSG-)
  positive-replies/        reply clusters — talking training data                  (RPL-)
  silent-conversions/      no-reply conversions — quiet training data              (SCV-)
  objections/              objection clusters + the response that re-engaged       (OBJ-)
  competitor-mentions/     who prospects compare you to                            (CMP-)
  conversions/             reply/silent → meeting → opportunity                    (CNV-)
  reach-health/            deliverability / accept / send-success by channel       (RCH-)
  campaign-performance/    scoreboard + findings + next tests                      (FND-)
```

---

## The G.E.A.R. Loop

**G.E.A.R.** = Get, Evaluate, Author, Release. Same autoresearch loop, named in RevOps terms.
The Five Mechanisms are the *what*; G.E.A.R. is the *when*.

Because replies and silent events lag the send, each turn evaluates the **previous** release.
That lag is why the order is G→E→A→R and not G→A→R→E.

| Pillar | Job | Reads | Writes |
|--------|-----|-------|--------|
| **G — Get** | State + signals, one hypothesis | `brain/`, `results.tsv`, `ledger.tsv`, intent | `intent-signals/` |
| **E — Evaluate** | Score the previous release against `eval.md` | replies, analytics, CRM, ledger | `results.tsv`, `ledger.tsv`, `conversions/`, `reach-health/` |
| **A — Author** | Next campaign (one dimension) + brain update | hypothesis, scored identities | `campaigns/current.md`, `brain/*` |
| **R — Release** | Human-approved send; emit queue + packet | `campaigns/current.md` | sending tools, `deliverables/`, ledger rows |

### Setup (once per lineage)

Work with the human:

1. Agree a run tag (e.g. `sep3`). Branch `autogtm/<tag>` must not already exist.
2. `git checkout -b autogtm/<tag>` from `main`.
3. Confirm the config block above is filled. Confirm `eval.md` is untouched.
4. Confirm analytics + CRM can match sent identities (otherwise outcome 3 stays dark and you
   are back to reply-only — say so).
5. Initialize `campaigns/results.tsv` and `campaigns/ledger.tsv` with their header rows if empty.
6. **Baseline first.** Do not change `campaigns/current.md`. Get approval, send, wait, score,
   log as `keep`. Then start changing one dimension at a time.

### The cycle

LOOP (until the human stops you):

**G — Get**
1. Read state: `git log --oneline -5`, `tail -20 campaigns/results.tsv`, skim `brain/`
   (especially `winning-messages/`, `silent-conversions/`, `icp-patterns/`, `reach-health/`).
2. Pull fresh GTM triggers into `brain/intent-signals/` as `INT-` entities.
3. Form **one** falsifiable hypothesis that changes exactly one of {audience, message, CTA, channel}.

**E — Evaluate** *(the previous release)*
4. Pull reach stats from the sending tool. If a guardrail is breached, verdict is **discard**
   regardless of quality. Write/update an `RCH-` entity.
5. If `eligible_attempts` < `min_sends_to_score`, refuse to finalize. Log `skip` or leave
   `pending`.
6. Score every reply T0–T3. Match silent events S1–S3 onto `ledger.tsv` identities. Apply
   no-double-count.
7. If the attribution window is still open, log/update the row as `pending` and **do not**
   keep/discard. Move on to Author/Release of the *next* experiment only if the human wants
   overlap; default is one in-flight experiment per branch.
8. When the window is closed: compute `attributed_intent_quality`. Improved (and guardrails
   held) → **keep** the experiment commit. Equal/worse **or** a guardrail breached → **discard**
   (`git revert <experiment-commit>`).
9. Open/append `CNV-` records for meetings and opportunities so pipeline traces back to the
   experiment.

**A — Author**
10. If kept, promote the winning artifact into `brain/winning-messages/` as an `MSG-` with its
    score and the channel it ran on.
11. Cluster new T2/T3 → `RPL-`; new S2/S3 → `SCV-`; mine T0/T1 → `OBJ-`/`CMP-`; write the next
    recommended test as `FND-`.
12. Edit `campaigns/current.md` for the next experiment. One dimension. Simpler if tied.

**R — Release**
13. `git commit -m "experiment: <hypothesis>"`.
14. **Stop and get explicit human approval to send.** Then send through the configured tool
    for that channel, ≥ `min_sends_to_score` attempts, with a unique `utm_campaign` /
    experiment id on every URL.
15. Append one ledger row per attempt (`campaigns/ledger.tsv`).
16. Refresh the action queue (M3) and follow-up packet (M4).
17. Go to **G**. Do not ask "should I keep going?" — continue until the human interrupts.
    The inner loop is autonomous; the send is not.

**Operating rules**
- Never finalize below `min_sends_to_score`. Never finalize before `attribution_window_days`.
- One dimension per experiment. Attribution is the whole value.
- A lift that breaches a guardrail is a discard.
- Cross-reference by entity ID and `identity_key`, never by prose.
- On a blocker (API error, no data), log `skip` (or `crash` if the tool failed), move on.
  Don't retry the same approach twice.
- Redirect noisy tool output away from context (`> run.log 2>&1` analog: pull *summaries*
  from the sequencer/CRM/analytics, not raw dumps).

---

## Log format

### `campaigns/results.tsv` — one row per experiment

Tab-separated, append-only:

```
commit	hypothesis	audience	channel	dimension	eligible_attempts	attributed_intent_quality	reply_rate	silent_serve_rate	delivery_rate	negative_rate	status	notes
```

`status` ∈ `keep | discard | skip | pending | crash`.
`dimension` ∈ `audience | message | cta | channel`.
`channel` ∈ `email | linkedin | dm`.
Leave lagging fields blank until known. `pending` becomes `keep` or `discard` when the window
closes. Do not rewrite history — append a corrected row and note the original commit in `notes`
if you must amend.

### `campaigns/ledger.tsv` — one row per attempt

```
identity_key	experiment	channel	sent_at	delivered	reply_tier	silent_tier	weight	notes
```

This is how email, LinkedIn, and DMs collapse into one person. Merge rows when you learn a
second identifier for the same human.

---

## Findings

A finding is a result that changes how you sell. Write it in plain language as an `FND-` in
`brain/campaign-performance/`:

- A message or channel that moved `attributed_intent_quality` >20%
- A silent path (no reply → site book) large enough to deserve its own CTA or page
- An objection cluster that deserves a content asset
- An ICP signal that predicts T3 or S3
- A competitor mention pattern worth a positioning change

Findings are the source material for case studies, sales enablement, and the next quarter's tests.
