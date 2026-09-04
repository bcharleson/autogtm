# AutoGTM

**Karpathy's [`autoresearch`](https://github.com/karpathy/autoresearch) loop, applied to outbound.**

Clone it. Fill in the company. Hand [`AGENTS.md`](AGENTS.md) to any agent. It treats every campaign
as an experiment: change one thing, send (you approve), wait, score what actually happened, keep
the winner, discard the loser, write the lesson into a brain. Run it long enough and outbound
stops being a static playbook and becomes a learning machine.

It is **channel-agnostic** and **client-agnostic**. Email, LinkedIn, DMs — same loop. Nothing in
this repo is specific to a company or a vendor. You point it at a company in the config block of
[`program.md`](program.md). Company facts and tools flow *through* the loop as data.

---

## The idea in one table

| Karpathy's `autoresearch` | AutoGTM |
|---------------------------|---------|
| Fixed eval = `val_bpb` in `prepare.py` (agent cannot touch it) | Fixed eval = **`attributed_intent_quality`** in [`eval.md`](eval.md) (agent cannot touch it) |
| Modifiable artifact = `train.py` | Modifiable artifact = [`campaigns/current.md`](campaigns/current.md) |
| Loop = train 5 min → keep/discard | Loop = send → wait the attribution window → keep/discard |
| First run = unchanged baseline | First run = unchanged baseline |
| Output = a better model | Output = a sharper **GTM brain** + an **action queue** |

---

## Outbound has three outcomes, not one

Most teams optimize reply rate on a single channel. That discards campaigns that are working.

| Outcome | What happened | Role in AutoGTM |
|---------|---------------|-----------------|
| **1. Reach** | The message was deliverable. You still cannot prove they *read* it. | **Guardrail.** Opens are not a metric. Hygiene is. |
| **2. Reply** | They answered: yes, no, or unsubscribe. | **Eval input** (T0–T3). |
| **3. Silent self-serve** | They never replied. They Googled you, asked a model, looked you up, hit the site, and acted. | **Eval input** (S1–S3). |

The one keep/discard number is **`attributed_intent_quality`**: weighted buying intent per
*reached* attempt, with replies and silent conversions collapsed per person (no double-count).
A low-reply campaign that books meetings from the site is a keep. Full ruler: [`eval.md`](eval.md).

The operator runs this as **G.E.A.R.** — Get → Evaluate → Author → Release — the RevOps name for
the same autoresearch loop. Mapping lives in [`program.md`](program.md).

---

## Quickstart

```bash
git clone https://github.com/bcharleson/autogtm.git
cd autogtm
```

1. Fill the **Configuration** block at the top of [`program.md`](program.md) (company, ICP, offer,
   channels, CRM, analytics, the agent running it).
2. Open the repo in **Grok Bot, Hermes, OpenClaw, Claude Code, Cursor, Codex**, or any agent that
   loads [`AGENTS.md`](AGENTS.md). If your host needs extra wiring, load one file from
   [`runtimes/`](runtimes/).
3. Seed `brain/` with anything you already know (optional).
4. Tell the agent to read `AGENTS.md` and start. **You approve every send.** It scores, logs,
   updates the brain, and asks for the next send until you stop it.

That is the whole product. There is no app to install.

---

## Repo layout

```
AGENTS.md                   ← hand this to the agent. Auto-loaded by most runtimes.
program.md                  ← the one file a human edits (config + loop + rules)
eval.md                     ← the fixed ruler. The agent does not edit this.
methodology.md              ← why it is shaped this way
campaigns/
  current.md                ← the single artifact the agent edits per experiment
  results.tsv               ← one row per experiment (keep / discard / pending)
  ledger.tsv                ← one row per attempt (how channels collapse to a person)
brain/                      ← compounding memory. Ships empty. Typed entities, see brain/README.md
channels/                   ← email · linkedin · dm contracts (same loop, different tool)
runtimes/                   ← optional host adapters (Grok Bot, Hermes, OpenClaw, Claude Code)
loops/                      ← message, audience, objection, channel, silent-serve
deliverables/templates/     ← action queue + follow-up packet the loop emits
```

Lean on purpose, in the spirit of `autoresearch`. Three files matter: `program.md` (human),
`eval.md` (fixed), `campaigns/current.md` (agent). Everything else is scaffolding.

---

## What the agent may change — and what it may not

**May change, one per experiment:** audience, message, CTA, or channel.

**May not change:** `eval.md`, sample-size / attribution-window mid-experiment, sending
infrastructure, more than one dimension at once.

**A human approves every outbound send.** The inner loop is autonomous. The send is not.

---

## Runtime adapters

The loop is runtime-neutral. Adapters are optional glue (paths, chat formatting). Ignore them
if you do not need one.

| Runtime | Adapter |
|---------|---------|
| Grok Bot | [`runtimes/grokbot.md`](runtimes/grokbot.md) |
| Hermes | [`runtimes/hermes.md`](runtimes/hermes.md) |
| OpenClaw | [`runtimes/openclaw.md`](runtimes/openclaw.md) |
| Claude Code / Cursor / Codex | [`runtimes/claude-code.md`](runtimes/claude-code.md) |

---

## Works with crystallized-intelligence

AutoGTM's `brain/` is **[crystallized-intelligence](https://github.com/bcharleson/crystallized-intelligence)-compatible**.
AutoGTM generates and scores the experiments that fill the brain; crystallized-intelligence
compiles that feedstock into tight agent-readable layers. You do not have to use both — a plain
folder of markdown is enough.

A named-persona wrapper that embeds this loop lives at
[`gtm-operator`](https://github.com/bcharleson/gtm-operator) if you want an identity file on top.
You only need **this** repo to run AutoGTM.

---

## License

MIT — see [LICENSE](LICENSE). Clone it, fork it, run it on any channel, sell what you build with
it. A gift to anyone whose outbound should learn.
