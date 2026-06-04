# The GTM Brain

The compounding asset. The operating agent **reads this before writing any campaign** and
**updates it after scoring every run**. A plain folder of markdown is enough; it is also
[crystallized-intelligence](https://github.com/bcharleson/crystallized-intelligence)-compatible if
you want to compile it into seed / principles / knowledge layers.

| Folder | Holds | Filled by |
|--------|-------|-----------|
| `winning-messages/` | Copy that earned T2/T3 replies, with its eval score attached | keep-step of the loop |
| `objections/` | Each objection seen + the response that re-engaged | `loops/loop-objection.md` |
| `positive-replies/` | Positive replies clustered by intent — the "training data" | Mechanism 2 |
| `icp-patterns/` | Which segments convert; which signals predict a hot (T3) reply | `loops/loop-icp.md` |
| `competitor-mentions/` | Who prospects compare you to, and what they say | reply mining |
| `campaign-performance/` | Running scoreboard, findings, next recommended tests | every loop iteration |

**Rule:** this folder, in a real deployment, is **private** — it contains a company's actual
messages, replies, and ICP intelligence. In this open-source repo the folders ship empty (with
`.gitkeep`) precisely because the framework holds no company data. Keep it that way.
