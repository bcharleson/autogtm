# The GTM Brain — Ontology of Things

The compounding asset. The operating agent **reads this before writing any campaign (Get)** and
**updates it after scoring every run (Author)**. A folder of markdown, governed by a strict
ontology so an agent parses it by typed field, not by prose. A human browses; an agent **parses**.

This public repo ships **empty folders**. Real winning messages, replies, identities, and
objections are private. Keep them that way in any public fork.

## The Things (entity types)

Every file in `brain/` is exactly **one entity record**. Filename **is** the ID.

| Folder | Entity type | ID prefix | Filled by | G.E.A.R. |
|--------|-------------|-----------|-----------|----------|
| `icp-patterns/` | ICP signal | `ICP-` | `loops/loop-icp.md` | Get |
| `intent-signals/` | Intent signal | `INT-` | external triggers at Get | Get |
| `winning-messages/` | Message | `MSG-` | keep-step | Author |
| `positive-replies/` | Reply cluster | `RPL-` | Mechanism 2 (talking) | Author |
| `silent-conversions/` | Silent path | `SCV-` | Mechanism 2 (quiet) | Author |
| `objections/` | Objection cluster | `OBJ-` | `loops/loop-objection.md` | Author |
| `competitor-mentions/` | Competitor | `CMP-` | reply mining | Author |
| `conversions/` | Pipeline record | `CNV-` | Evaluate (downstream) | Evaluate |
| `reach-health/` | Channel hygiene | `RCH-` | Evaluate (outcome 1) | Evaluate |
| `campaign-performance/` | Finding | `FND-` | every iteration | Evaluate / Author |

## Agent-literalness rules (non-negotiable)

1. **One entity per file. The filename *is* the ID** — `OBJ-price-too-high.md`, `SCV-pricing-page-book.md`.
2. **Every file opens with valid YAML frontmatter.** Body below is optional human notes; nothing
   load-bearing lives only in prose.
3. **Use controlled-vocabulary values verbatim.** Never invent a synonym for an enum.
4. **Cross-reference by ID only** (`related: [MSG-roi-reframe]`), never by description.
5. **Numbers are numbers.** No `~12`; write `evidence_n: 12` and lower `confidence`.
6. **Append, never rewrite history.** Retire with `status: retired`; do not delete.
7. **Every quantitative claim carries `evidence_n` and `source_campaigns`.**
8. **Dates are ISO-8601.** IDs are lowercase-kebab and never reused.

## Common frontmatter (all entities)

```yaml
id: OBJ-price-too-high          # == filename, <PREFIX>-<kebab-slug>
type: objection                 # the entity type
status: active                  # active | hypothesis | retired
created: 2026-09-03
updated: 2026-09-03
confidence: high                # high | medium | low
evidence_n: 14
source_campaigns: [a1b2c3d]     # commit hashes from results.tsv
related: [MSG-roi-reframe, FND-pricing-content-gap]
tags: [pricing, finance]
```

## Per-type fields

```yaml
# ICP-  (icp-patterns/)
segment: treasury-fintech
signal: "uses incumbent X for payments"
predicts: T3                    # T0|T1|T2|T3|S1|S2|S3
lift: 2.4                       # quality lift vs. baseline (x)
channel: email                  # email | linkedin | dm | any
firmographic: {employee_count: "100-250", industry: fintech, geo: US}

# INT-  (intent-signals/)
trigger: funding_series_b       # funding_series_b|hiring_revops|tech_install|leadership_change|review_site|other
recency_window_days: 30
predicts: T3
source: "news / data provider"

# MSG-  (winning-messages/)
dimension: message              # audience | message | cta | channel
channel: email                  # email | linkedin | dm
step: 1
hook: cost-savings
proof_asset: named-client       # named-client | data-point | proof-point
cta: book-call                  # book-call | resource | soft-question
score_attributed_intent_quality: 0.058
won_against: MSG-compliance-hook

# RPL-  (positive-replies/)     talking training data
intent: pricing                 # pricing|demo-request|mechanism-question|synergy|tell-me-more|other
tier: T3                        # T2 | T3
channel: email
count: 9
representative_quote: "what are the costs?"
content_gap: true

# SCV-  (silent-conversions/)   quiet training data — no reply, they still acted
path: pricing-page-then-book    # kebab path label
tier: S3                        # S1 | S2 | S3
last_touch_channel: linkedin    # email | linkedin | dm
count: 6
landing: /pricing
content_gap: true

# OBJ-  (objections/)
objection_class: price          # price|incumbent|timing|authority|trust|fit|other
channel: email
count: 14
winning_response: MSG-roi-reframe
reengage_rate: 0.31
content_asset_needed: roi-calculator

# CMP-  (competitor-mentions/)
competitor: example-inc
mention_count: 7
context: comparison             # comparison | switching | incumbent
their_pitch: "cheaper, less setup"
our_counter: MSG-proof-edge
win_rate_vs: 0.4

# CNV-  (conversions/)          downstream pipeline, reply OR silent
origin: silent                  # reply | silent
reply_cluster: RPL-pricing      # optional
silent_cluster: SCV-pricing-page-then-book
booked: true
meeting_held: true
qualified_opportunity: true
stage: discovery                # discovery|proposal|closed_won|closed_lost
days_touch_to_meeting: 3

# RCH-  (reach-health/)         outcome 1 — hygiene, not the eval
channel: email
delivery_rate: 0.97
bounce_rate: 0.018
spam_complaint_rate: 0.0004
unsubscribe_rate: 0.006
negative_reply_rate: 0.011
guardrail_breach: false

# FND-  (campaign-performance/)
claim: "ROI hook lifts attributed_intent_quality 32% for treasury on email"
metric_delta: 0.018
action: "default ROI hook for treasury; build ROI calculator"
```

## Controlled vocabularies

Emit these values **exactly**:

- `reply_tier`: `T0 | T1 | T2 | T3`
- `silent_tier`: `S1 | S2 | S3`
- `dimension`: `audience | message | cta | channel`
- `channel`: `email | linkedin | dm | any`
- `status`: `active | hypothesis | retired`
- `confidence`: `high | medium | low`
- `intent`: `pricing | demo-request | mechanism-question | synergy | tell-me-more | other`
- `objection_class`: `price | incumbent | timing | authority | trust | fit | other`
- `proof_asset`: `named-client | data-point | proof-point`
- `cta`: `book-call | resource | soft-question`
- `trigger`: `funding_series_b | hiring_revops | tech_install | leadership_change | review_site | other`
- `stage`: `discovery | proposal | closed_won | closed_lost`
- `origin`: `reply | silent`

The ontology is [crystallized-intelligence](https://github.com/bcharleson/crystallized-intelligence)-compatible:
typed entities compile into its seed / principles / knowledge layers.
