# Channel — Email

**identity_key:** lowercase email, else company domain.
**Reach (eligible):** accepted / delivered. Bounces are not eligible. Opens are **not** reach.
**Reply:** inbound on the sending mailbox, scored T0–T3.
**Silent:** site / CRM / product events matched to that email or domain inside the window.

## Reach guardrails

Use the floors in the `program.md` config block (`min_delivery_rate`, `max_bounce_rate`,
`max_spam_complaint_rate`, `max_unsubscribe_rate`). Domain warmup, DNS, and sending-account
health are **infrastructure** — out of scope for `campaigns/current.md`. Do not "test" a new
domain in the same experiment as a new message.

## What the agent pulls from the sending tool

Summaries, not raw dumps: sends, delivered, bounced, unsubscribes, spam complaints, replies
with bodies. Write ledger rows. Score replies. Leave silent fields blank until analytics/CRM
match lands.

## What "eyeballs" means here

You do not know they read it. You know it was not bounced and did not spam-complaint at a
rate that burns the domain. That is all outcome 1 can honestly claim. Optimize outcome 2 and 3
among eligible attempts.
