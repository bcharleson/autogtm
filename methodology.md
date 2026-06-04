# Methodology

How AutoGTM maps Karpathy's `autoresearch` pattern onto outbound go-to-market. This is the longer-
form companion to [`program.md`](program.md); `program.md` is what you run, this is why it's shaped
the way it is.

## The original pattern

`autoresearch` works because three things are held in a specific relationship:

1. **A fixed evaluation metric.** It never changes between runs. This is what makes the loop
   trustworthy — the keep/discard decision means something only if the ruler is constant.
2. **A single modifiable artifact.** Exactly one thing changes per run, so improvement is
   *attributable*.
3. **An autonomous loop.** Run → measure → keep-or-discard → log → repeat, with no human in the
   inner loop.

Compounding comes from the interaction: a constant ruler + attributable changes + many iterations
= a monotonic climb, even when any single step is a guess.

## Porting it to GTM

The only real design work is choosing the metric and the artifact. Get those right and the rest
of the pattern transfers unchanged.

- **Metric → positive-reply quality.** Not opens (vanity, increasingly unmeasurable), not raw
  reply rate (counts unsubscribes and "no thanks" as signal). Quality weights replies by buying
  intent, so the optimization target is *buyers*, not *responders*. Full rubric in `program.md`.
- **Artifact → the campaign.** Audience, message, CTA. One dimension per experiment.

## Why agent-operated

A human can run this loop, but it's wasteful: scoring replies, clustering them, updating a brain,
and re-drafting copy is exactly the work an agent does well and cheaply. More importantly, when the
operator *is* the agent, the brain the loop updates is the same brain the agent reads — so the
system improves the operator, not just the output. A human operator breaks that feedback loop;
an agent operator closes it.

## What stays out of this repo

Everything company-specific. Regulatory facts, named clients, real copy, real reply data, the
sending account — all of it is *data that flows through the loop*, configured per deployment in the
`program.md` config block and stored in a private brain. This repo only ever contains the
*mechanism*. That boundary is what makes it a framework instead of one company's playbook.
