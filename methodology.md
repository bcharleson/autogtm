# Methodology

How AutoGTM maps Karpathy's `autoresearch` pattern onto outbound go-to-market. Companion to
[`program.md`](program.md) (what you run) and [`eval.md`](eval.md) (the ruler). This file is why
it is shaped this way.

## The original pattern

`autoresearch` works because three things are held in a specific relationship:

1. **A fixed evaluation metric.** It never changes between runs. The keep/discard call means
   something only if the ruler is constant. In Karpathy's repo that ruler lives in `prepare.py`,
   which the agent is forbidden to edit.
2. **A single modifiable artifact.** Exactly one thing changes per run, so improvement is
   *attributable*. There it is `train.py`.
3. **An autonomous loop.** Run → measure → keep-or-discard → log → repeat. First run is the
   unchanged baseline. The human edits `program.md`; the agent does not stop to ask permission
   inside the loop.

Compounding is the interaction: constant ruler + attributable changes + many iterations.

## What ports unchanged

| autoresearch | AutoGTM |
|--------------|---------|
| `prepare.py` is read-only | [`eval.md`](eval.md) is read-only |
| Agent edits only `train.py` | Agent edits only [`campaigns/current.md`](campaigns/current.md) |
| Human edits only `program.md` | Human edits only [`program.md`](program.md) (the config block) |
| Dedicated branch `autoresearch/<tag>` | Dedicated branch `autogtm/<tag>` |
| First run = baseline | First run = baseline |
| `results.tsv`, keep / discard / crash | `results.tsv`, keep / discard / skip / pending / crash |
| Simplicity criterion | Simplicity criterion |
| NEVER STOP the inner loop | NEVER STOP the inner loop — **except** a human must approve each send |
| Fixed 5-minute budget so runs are comparable | Fixed `min_sends_to_score` + `attribution_window_days` so runs are comparable |

The GTM analog of "wait for the GPU run to finish" is **wait for the attribution window**. Scoring
a campaign the morning after send is the same mistake as killing `train.py` at 30 seconds.

## What had to change (and why)

### The metric is not reply rate

Reply rate counts unsubscribes as signal and treats silence as failure. Silence is not failure.
Three things can happen after a reached attempt:

1. **Reach** — the message was deliverable. You still cannot prove eyeballs. Opens are a lie
   (or at least not a ruler). Reach is a **guardrail**: bounce, spam, accept, send-success.
   A broken channel does not get to "win" on quality.
2. **Reply** — yes, no, unsubscribe. Scored T0–T3 by buying intent, not by volume.
3. **Silent self-serve** — no reply, but they researched you (search, an LLM, LinkedIn, your
   site) and acted. Matched onto a sent identity inside the window. Scored S1–S3.

The one number, `attributed_intent_quality`, is Σ of per-identity max(reply, silent) over
*eligible* (reached) attempts. Same person who replies T2 and then books from the site is a 3,
not a 5. Unmatched brand traffic is not an experiment result.

This is the whole point of the port. A campaign that "loses" on reply rate and books pipeline
from the site is a successful outbound motion. The old ruler would have discarded it.

### The artifact has a fourth dimension

Audience, message, CTA — and **channel**. Email vs LinkedIn vs DM is a testable dimension,
held to the same one-at-a-time rule. The loop does not care which tool sent it. Identity
resolution in `eval.md` is how the dots connect: the person who was emailed, ignored it, and
booked from the site after a LinkedIn look-up is one row, one weight, one experiment.

### A human stays on the send

Karpathy can leave the agent running overnight because `train.py` cannot email strangers.
AutoGTM cannot. The inner loop (score, log, author the next artifact, update the brain) is
autonomous. **Release always stops for explicit human approval.** That is the only extra gate.
It is not optional, and no runtime adapter may remove it.

### G.E.A.R. is not a second framework

Get → Evaluate → Author → Release is the autoresearch loop named so a RevOps operator (human
or agent) can execute it without translation. Evaluate looks *backward* at the previous
release because GTM signal lags. That is why the letters are not G→A→R→E.

## Why one target and many watchers

RevOps dashboards invite you to optimize twelve numbers. The moment two metrics vote, you
lose attribution — the thing the loop exists to produce. So AutoGTM:

- **optimizes** exactly one number (`attributed_intent_quality`)
- **gates** it with reach / brand guardrails (a lift that burns the domain is a discard)
- **explains** it with diagnostics (reply vs silent, channel, time-to-intent)
- **validates** it downstream (meetings held, qualified opportunities) on a human cadence,
  never inside the inner loop

## Why agent-operated

Scoring replies, matching silent events, clustering intent, updating a typed brain, and
drafting the next one-dimension experiment is work an agent does well. More importantly:
when the operator *is* the agent, the brain the loop writes is the brain the agent reads.
A human operator breaks that loop; an agent operator closes it.

Runtime-agnostic on purpose. Grok Bot, Hermes, OpenClaw, Claude Code, Cursor, Codex, or a
script — if it can read markdown, call a sending tool and a CRM, and write files, it can run
this. Host-specific glue lives in `runtimes/` so the core never takes a vendor dependency.

## What stays out of this repo

Everything company-specific. Named clients, real copy, real reply data, sending accounts,
API keys. Configured per deployment in the `program.md` config block and stored in a private
brain. This repo only ever contains the *mechanism*. That boundary is what makes it a
framework instead of one company's playbook — and what makes a public fork safe to clone.
