# Agentic Threat Hunting: Building AI That Remembers What You Hunted

Workshop materials from the DEF CON 4-hour workshop on agentic threat hunting.

## What's Here

| Path | Contents |
|------|----------|
| `slides/agentic-hunting-workshop-full.pdf` | Full workshop deck (63 slides) |
| `slides/agentic-hunting-workshop-full.pptx` | Same deck, editable |
| `labs/lab-workbook.pdf` | All five labs, student edition (100 pages) |
| `labs/complete-answer-key.pdf` | Full solutions for every lab and bonus |

Markdown sources for both workbooks are alongside the PDFs.

## The Workshop

A lab-heavy walk through the ATHF **Five Levels of Agentic Hunting** — L0 Ad-hoc, L1 Structured, L2 Reused, L3 Researched, L4 Agentic — against a seeded supply-chain compromise.

The levels aren't a ladder you abandon rungs on. They're a stack: L4 only works because L1 wrote the hunts down that L2 reused that L4 now reasons over. The memory is the moat.

The scenario pits a fictional victim org, **Sudo Shipping** (fleet-logistics SaaS), against a fictional e-crime crew, **CRON GOBLIN**. Both are 100% synthetic, as is every host, domain, IP, and event in the dataset. Nothing here describes a real intrusion or a real customer.

**Framework:** [Agentic Threat Hunting Framework (ATHF)](https://github.com/Nebulock-Inc/agentic-threat-hunting-framework) — open source, MIT.

**Scope:** hunt methodology and agentic workflow. Not detection engineering, not red-teaming AI, not Splunk vendor training.

## Working Through the Labs on Your Own

The labs were written against a shared Splunk instance seeded for the live session, which isn't running anymore. The workbook still stands on its own as a reading path — every lab states its hypothesis, the SPL that answers it, and what the answer means. The answer key includes expected results, so you can follow the reasoning end to end without the data.

Queries assume the workshop dataset in `index=thrunt`, with process telemetry in the **ECAR** schema (`sourcetype="evidenceforge:ecar:json"`).

## Who Built This

**Workshop leads** — Sydney Marrone, Eric Brown, Jarrett Polcari.

**Testers** — ran every module and lab cold, no answer key, and told us what broke: Joshua Hines, Alex Hurtado, Brandon Schwartz.

**Infrastructure** — Ray Salamon.

## License

See [LICENSE](LICENSE).
