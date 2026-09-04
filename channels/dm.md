# Channel — Direct messages

Covers Slack DMs, X/Twitter DMs, WhatsApp, or any one-to-one channel that is not email or
LinkedIn. The loop does not change.

**identity_key:** `platform:handle` (e.g. `x:someuser`, `slack:U123`), else email, else domain.
**Reach (eligible):** the platform accepted the send. Blocks and failures are not eligible.
**Reply:** reply in the same thread, scored T0–T3 with the same rubric.
**Silent:** site / CRM / product events matched to this identity inside the window.

## Reach guardrails

Platform blocks, failed sends, or spam flags on the sending account are a discard. Informal
channels are not an excuse to skip human approval.

## Notes

- Stamp the same `utm_campaign` on any URL you drop in a DM.
- Merge `platform:handle` onto an email/LinkedIn row when you learn they are the same person.
- Group chats and public posts are **not** this channel. AutoGTM is outbound to a chosen
  identity, not broadcast.
