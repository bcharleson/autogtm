# Current campaign — the modifiable artifact

The agent edits **this file**. One dimension per experiment. A human approves the send.

Fill the fields. Do not add a second changed dimension in the same experiment.

```yaml
experiment_id:      "{{short git commit after you commit this file}}"
hypothesis:         "{{falsifiable claim — one sentence}}"
dimension:          message          # audience | message | cta | channel
channel:            email            # email | linkedin | dm
audience:           "{{ICP slice this run}}"
utm_campaign:       "{{stable id stamped on every URL this run}}"
status:             draft            # draft | approved | sent | pending | scored
```

## Message

Subject / opener / first line:

```
{{one line}}
```

Body (keep it the simplest version that tests the hypothesis):

```
{{copy}}
```

CTA:

```
{{the ask, placement, and any URL with utm_campaign}}
```

## What changed vs. the last keep

- Dimension: `{{audience | message | cta | channel}}`
- Before: `{{previous value}}`
- After: `{{this value}}`

## Notes (not load-bearing)

Anything the next agent needs that is not a field above. Do not hide a second test here.
