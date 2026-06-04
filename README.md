# AutoGTM

**An autonomous go-to-market system. Karpathy's `autoresearch` loop, applied to outbound.**

AutoGTM treats every campaign like a research run. Fix one evaluation metric — *positive-reply
quality* — let an agent vary one thing at a time (audience, message, or CTA), keep what works,
discard what doesn't, and feed every positive reply back into a brain that makes the next campaign
sharper. Run it long enough and your go-to-market motion stops being a static playbook and becomes
a learning machine.

It is **client-agnostic**: nothing in this repo is specific to any one company. You point it at a
company by filling in the config block in [`program.md`](program.md). All company-specific facts
flow *through* the loop as data; they are never baked into its structure.

---

## The idea in one table

| Karpathy's `autoresearch` | AutoGTM |
|---------------------------|---------|
| Fixed eval = held-out accuracy | Fixed eval = **positive-reply quality** |
| Modifiable artifact = the model | Modifiable artifact = **the campaign** |
| Loop = train → eval → keep/discard | Loop = send → score → keep/discard |
| Output = a better model | Output = a sharper **GTM brain** |

The non-obvious bet: **optimize on positive-reply *quality*, not reply rate.** A "send me a
proposal" is worth more than ten opens. Quality is the signal that compounds.

---

## Agent-operated by design

AutoGTM is built to be run by an autonomous agent deployment (e.g. an
[OpenClaw](https://github.com/) / Hermes agent provisioned for a client), not a human in a
spreadsheet. The human sets strategy and approves sends; the agent runs the loop, scores replies,
updates the brain, and surfaces the next action.

The payoff: because the brain the agent reads *is* the brain the loop updates, **every campaign the
agent runs makes the same agent better.** Campaign → positive replies → brain → sharper agent →
better campaign. That is the entire system.

---

## Repo layout

```
program.md                  ← the one file you edit. The loop, the eval, the rules.
methodology.md              how AutoGTM maps onto the autoresearch pattern (longer form)
loops/                      individual loop specs (outbound, ICP, objection)
brain/                      the GTM brain — the compounding asset
  winning-messages/         copy that earned T2/T3 replies, scored
  objections/               objections seen + responses that re-engaged
  positive-replies/         clustered by intent — the "training data"
  icp-patterns/             which segments convert, which signals predict hot replies
  competitor-mentions/      who you're compared to, and what's said
  campaign-performance/     scoreboard + findings + next recommended tests
campaigns/
  results.tsv               append-only experiment log
deliverables/
  templates/                follow-up packet + action queue templates the loop emits
```

---

## Quickstart

1. Open [`program.md`](program.md) and fill in the **Configuration** block (company, ICP, offer,
   sending tool, language, the agent operating the loop).
2. Read the **Five Mechanisms** and the **Experiment Loop** sections — that's the whole system.
3. Seed `brain/` with whatever you already know (winning messages, known objections).
4. Hand `program.md` to your operating agent and run the loop. Approve sends; let it score, log,
   and learn.

---

## Relationship to crystallized-intelligence

AutoGTM's `brain/` is [crystallized-intelligence](https://github.com/bcharleson/crystallized-intelligence)-
compatible. That project is the framework for *compiling* domain expertise into agent-readable
layers; AutoGTM is the framework for *generating and scoring* the GTM experiments that fill it.
Use them together or use AutoGTM's brain on its own — a plain folder of markdown works fine.

---

## License

MIT — see [LICENSE](LICENSE). A gift to anyone building a self-improving go-to-market motion.
