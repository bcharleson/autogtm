# Runtime adapter — Hermes

Host wiring for a Hermes agent. The loop does not change.

## How it loads

Point Hermes at this repo. It should pick up [`AGENTS.md`](../AGENTS.md). If your build uses a
skill/instruction file instead, copy `AGENTS.md` into that slot and keep `program.md` as the job.

## Workspace

Use the repo root as the workspace. Brain, campaigns, and deliverables stay in-tree so a
restart does not lose the lineage. Do not write load-bearing files to a volatile `/tmp`.

## How to run a lineage

1. Config block in `program.md` filled.
2. Prompt: `Read AGENTS.md. Start a new AutoGTM lineage. Baseline first.`
3. Hermes drafts `campaigns/current.md` and **stops** for send approval. The human approves
   in chat or via your Hermes approval hook.
4. After send, Hermes writes ledger rows and waits the attribution window (status `pending`)
   before keep/discard.

## Tools

Call the configured sending tool, CRM, and analytics. Heartbeats / cron in Hermes may run
**Get** and **Evaluate**. They may not run **Release** unattended.

If Hermes and OpenClaw share a host in your deployment, still load this file for Hermes-specific
paths and approval UX. Do not assume OpenClaw paths exist.
