# Contributing

This repo is a *mechanism*, not a product. Keep it that way.

## What belongs here

- Clarifications to `program.md`, `eval.md`, or `AGENTS.md` that make the loop easier to run.
- A new **runtime adapter** in `runtimes/` (one markdown file, host wiring only — no change to
  the eval or the loop).
- A new **channel contract** in `channels/` if the loop is genuinely the same and only the
  reach-signal / identity_key differs.
- Ontology fields in `brain/README.md` that an agent needs in order to parse, not prose.

## What does not belong here

- Company names, real copy, real leads, API keys, sending-account details.
- Vendor lock-in. Do not make a sequencer, a CRM, or an agent host the default. List them as
  examples in config comments, never as required infrastructure.
- A second evaluation metric that votes in keep/discard. If you want a different ruler, version
  `eval.md` and start a new `autogtm/<tag>` lineage — do not sneak a second number into the
  verdict.
- App code, SDKs, or installers. AutoGTM is markdown an agent runs. If you need a binary, it
  is a different repo.

## How to add a runtime adapter

1. Copy `runtimes/claude-code.md` as the shape.
2. Document only what this host needs: how it loads `AGENTS.md`, workspace paths, approval UX,
   anything that would otherwise leak into `program.md`.
3. Link it from `runtimes/README.md` and the table in the root `README.md`.
4. Do not fork the eval.

## How to add a channel

1. Same loop. Different reach signal and `identity_key`. See `channels/README.md`.
2. Add `channels/<name>.md` and a row on the channel table.
3. `dimension: channel` already exists — you should not need a new experiment type.

## PRs

- One idea per PR.
- No generated files, no `node_modules`, no live `brain/` records.
- Keep the root README cloneable in five minutes. If a change makes the quickstart longer,
  it is probably in the wrong file.
