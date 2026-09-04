# Loop — Channel

Optimizes `attributed_intent_quality` by varying the **channel**, holding audience, message,
and CTA fixed. This is how you learn whether email, LinkedIn, or DMs actually create buying
intent — including silent site conversions from people who never reply on that channel.

**Fixed eval:** [`eval.md`](../eval.md).
**Modifiable:** `channel` (`email | linkedin | dm`).
**Not modifiable:** audience, message, CTA, the eval, sending infrastructure.

**Rules:**
- Same offer, same ICP slice, same CTA. Only the surface changes.
- Stamp the same `utm_campaign` family so silent events still match.
- Credit conversions to the **experiment**, not to whichever surface the form was on.
- Reach guardrails are per-channel (see `channels/`). A LinkedIn restriction is a discard
  even if a handful of DMs looked good.

Log with `dimension = channel`. Promote winning surface + copy into `MSG-` with `channel:` set.
