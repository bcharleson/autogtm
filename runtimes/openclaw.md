# Runtime adapter — OpenClaw

Host wiring for an OpenClaw agent. The loop does not change.

## How it loads

OpenClaw auto-loads [`AGENTS.md`](../AGENTS.md) from the workspace. If your gateway uses a
different instruction path, symlink or copy `AGENTS.md` there. Do not duplicate the loop into
an OpenClaw-only skill — one `program.md` is the job.

## Workspace paths

Use the repo (or the OpenClaw workspace that *is* this repo) as the base:

- Brain: `./brain/`
- Campaigns: `./campaigns/`
- Deliverables: `./deliverables/`

Prefer paths relative to the workspace. If your OpenClaw sandbox cannot see `~`, do not use
home-directory paths.

## How to run a lineage

1. Config block in `program.md` filled.
2. Prompt: `Read AGENTS.md. Start a new AutoGTM lineage. Baseline first.`
3. OpenClaw drafts the campaign and waits for explicit human approval before calling any
   sending tool.
4. After send, write ledger rows. Evaluate only when `eval.md` says you may.

## Chat formatting

If the human is on Slack via OpenClaw, use Slack mrkdwn (`*bold*`, `_italic_`, `<url|text>`).
If the human is in a standard chat, use ordinary markdown. Match the channel you are in.

## Unattended work

OpenClaw cron / heartbeat: **Get** and **Evaluate** are safe unattended. **Release is not.**
Queue a proposed send for the human; do not fire it from a heartbeat.
