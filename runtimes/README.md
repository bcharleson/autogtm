# Runtime adapters

The loop (`program.md`) and the ruler (`eval.md`) are runtime-neutral. Some hosts need extra
wiring: how they load `AGENTS.md`, workspace paths, how a human approves a send in that UI.
That glue lives here, one file per runtime, so the core never takes a vendor dependency.

Load the adapter that matches your host. Ignore the rest. If there is no adapter, you do not
need one — the operator runs on `AGENTS.md` + `program.md` alone.

| Adapter | For |
|---------|-----|
| [`grokbot.md`](grokbot.md) | Grok Bot / Grok Build |
| [`hermes.md`](hermes.md) | Hermes |
| [`openclaw.md`](openclaw.md) | OpenClaw |
| [`claude-code.md`](claude-code.md) | Claude Code, Cursor, Codex, and any agents.md host |

Adapters **may not** weaken the human-approval rule, edit `eval.md`, or introduce a second
keep/discard metric. If a host cannot wait for a human on send, it cannot run Release — it
can still Get / Evaluate / Author and queue the send for a human.
