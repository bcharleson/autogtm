# Channel — LinkedIn

**identity_key:** canonical LinkedIn profile URL (no tracking params), else email, else domain.
**Reach (eligible):** connection request or InMail/message that the platform reports as sent /
accepted per your tool. Failed sends and restriction warnings are not eligible.
**Reply:** reply on the LinkedIn thread, scored T0–T3 with the **same** rubric as email
("let's book time" is T3 here too).
**Silent:** profile views of the sender (only if the tool actually gives them), plus site / CRM
events matched to this identity. A profile view with no other action is S1 at most.

## Reach guardrails

LinkedIn restriction warnings, failed sends, or a collapsing accept rate are a **discard**,
same as a spam-complaint spike on email. Do not raise volume to chase the eval.

## Connecting the dots

People ignore cold email and still look you up on LinkedIn, then convert on the site. That is
outcome 3, not a LinkedIn "reply." Merge the LinkedIn URL onto the existing email ledger row
when you learn it. Credit the **experiment**, not the surface the conversion arrived on.

## What the agent pulls

Send/accept/fail counts, restriction flags, reply threads, any view signal the tool actually
exposes. Do not invent "they saw the message" from time-on-profile or similar.
