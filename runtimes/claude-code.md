# Runtime adapter — Claude Code / Cursor / Codex

Host wiring for coding-agent runtimes that load [`AGENTS.md`](../AGENTS.md) (Claude Code,
Cursor, Codex, and anything following the agents.md convention). The loop does not change.

## How it loads

Open this repo in the tool. `AGENTS.md` is picked up automatically. If not, `@AGENTS.md` and
tell it to follow `program.md`.

## How to run a lineage

```
Read AGENTS.md and start a new AutoGTM lineage. Baseline first. Do not send anything until I approve.
```

1. The agent reads config, eval, brain, last results.
2. It proposes the baseline (or the next one-dimension change) in `campaigns/current.md`.
3. You approve. It commits `experiment: <hypothesis>` and calls the sending tool you configured.
4. It logs ledger rows. It will not finalize keep/discard until the attribution window closes —
   leave the session, come back, or schedule a later turn to Evaluate.

## Git

Use a dedicated branch `autogtm/<tag>` as `program.md` specifies. Do not commit live `brain/*.md`
records to a public fork (see `.gitignore`). `results.tsv` and `ledger.tsv` headers are tracked;
a private deployment may gitignore the live rows if they contain customer data.

## Tools

Use the CLIs / MCP servers you already have for mail, LinkedIn, CRM, and analytics. Pull
summaries. Redirect noisy output away from context.

This adapter also covers "I just opened the folder in Cursor and started chatting." No extra
install. If you later deploy the same repo onto Grok Bot, Hermes, or OpenClaw, add that
adapter — do not rewrite `program.md`.
