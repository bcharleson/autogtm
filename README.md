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

## Works with crystallized-intelligence

AutoGTM's `brain/` is **[crystallized-intelligence](https://github.com/bcharleson/crystallized-intelligence)-compatible**
out of the box. The two frameworks are designed to pair:

| | crystallized-intelligence | AutoGTM |
|---|---|---|
| Job | **Compiles** expertise into agent-readable layers (seed → principles → knowledge → sources → raw) | **Generates and scores** the GTM experiments that fill those layers |
| Direction | Structures what you already know | Discovers what actually converts |
| Role in the pair | The brain's *format* | The brain's *feedstock* |

**How they connect:** every positive reply AutoGTM scores becomes first-party signal. Run it through
crystallized-intelligence and it compiles into the `brain/`'s seed/principles layers — so the operating
agent reads a *crystallized* brain (tight, load-bearing) instead of a raw pile of replies. AutoGTM keeps
the loop honest; crystallized-intelligence keeps the brain compact. Each makes the other sharper.

You don't have to use both — AutoGTM's `brain/` works as a plain folder of markdown. But if you want the
brain to stay lean as it grows, point crystallized-intelligence at it. See its
[repo](https://github.com/bcharleson/crystallized-intelligence) for the layer spec and the `crystallize` tooling.

---

## License

MIT — see [LICENSE](LICENSE). A gift to anyone building a self-improving go-to-market motion.
