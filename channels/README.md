# Channels

AutoGTM is one loop on every outbound surface. A channel is not a product. It is a **contract**:
how you send, how you know the attempt reached, how you name the person, how a reply shows up,
and how a silent event still matches them.

The loop in `program.md` never special-cases a vendor. If you add a channel, you add a contract
file here — you do not fork the eval.

| Channel | Contract | Typical tools (examples, not requirements) |
|---------|----------|--------------------------------------------|
| email | [`email.md`](email.md) | any sequencer or mailbox |
| linkedin | [`linkedin.md`](linkedin.md) | any LinkedIn sending tool or manual send |
| dm | [`dm.md`](dm.md) | Slack, X/Twitter, WhatsApp, or any direct channel |

`dimension: channel` is how you *test* which surface wins, holding audience + message + CTA
constant. Identity matching across surfaces is defined in [`eval.md`](../eval.md) and logged in
[`campaigns/ledger.tsv`](../campaigns/ledger.tsv).

## Shared rules

- Stamp `utm_campaign` / experiment id on every URL you control.
- Write one ledger row per attempt at send time, with `delivered` filled when the tool reports it.
- Reach failures (bounce, failed send, restriction) are guardrails, not quality.
- A reply on this channel and a site conversion from the same identity are one weight (`max`).
- Human approval is required on every send, on every channel. There is no "DMs are informal" exception.
