# Runtime adapter — Grok Bot

Host wiring for Grok Bot / Grok Build. The loop does not change.

## How it loads

Open this repo as the workspace. Grok Bot auto-loads [`AGENTS.md`](../AGENTS.md). If it does
not, paste `AGENTS.md` as the first instruction and tell it to follow `program.md`.

## How to run a lineage

1. Confirm the config block in `program.md` is filled.
2. Prompt: `Read AGENTS.md and start a new AutoGTM lineage. Baseline first.`
3. Approve or reject each proposed send in the chat. Never pre-authorize bulk send.
4. Let it loop: score the previous release, author `campaigns/current.md`, ask again.

## Tools

Use whatever sending tool, CRM, and analytics the config names. Prefer MCP / CLI over
pasting exports. Pull **summaries** (counts, reply bodies, matched identities) — do not dump
raw campaign CSVs into context.

Scheduled G.E.A.R. turns (Evaluate on a closed window, refresh the action queue) are a good
fit for a recurring Grok Bot task. **Release is still a human gate.** A scheduled run may
score and draft. It may not send.

## Files

Work in the repo workspace. Do not write brain entities or ledgers into a scratch home
directory the next session will not see.
