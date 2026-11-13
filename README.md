# Agentic Threat Hunting: Building AI That Remembers What You Hunted

Workshop materials for the Nebulock **Agentic Threat Hunting** class — Friday, November 13,
2026, 09:30 ET, Cincinnati.

## The Workshop

150 minutes, four modules, climbing the ATHF **Five Levels of Agentic Hunting** — L0 Ad-hoc,
L1 Structured, L2 Reused, L3 Researched, L4 Agentic — against a seeded supply-chain compromise.

The levels aren't a ladder you abandon rungs on. They're a stack: L4 only works because L1
wrote the hunts down that L2 reused that L4 now reasons over. The memory is the moat.

The scenario pits a fictional victim org, **Sudo Shipping** (fleet-logistics SaaS), against a
fictional e-crime crew, **CRON GOBLIN**. Both are 100% synthetic, as is every host, domain, IP,
and event in the dataset. Nothing here describes a real intrusion or a real customer.

**Framework:** [Agentic Threat Hunting Framework (ATHF)](https://github.com/Nebulock-Inc/agentic-threat-hunting-framework) — open source, MIT.

## Before Class

1. **Read [`PRE_READ.md`](PRE_READ.md)** (~10 minutes): the five levels, LOCK, ABLE, and
   behavior over indicators.
2. **Install ATHF** (Python 3.11+):

   ```bash
   python3 -m venv .venv
   source .venv/bin/activate        # Windows: .venv\Scripts\activate
   pip install --upgrade pip
   pip install 'agentic-threat-hunting-framework[all]'
   athf --version
   ```

   You want a version number back from the last line.
3. **Run the preflight check:** `python3 preflight/validate.py` (see [`preflight/`](preflight/README.md)).
   A warning about a missing LLM key is fine.
4. **Bring a laptop and a web browser.** No LLM API key needed: the AI modules run on the
   presenter's screen.
5. **Splunk login:** pick a row in the
   [credentials sheet](https://docs.google.com/spreadsheets/d/1xDZbHArGZHN1QscB1jALOHR3JeTuHU9S7_z0o9AZMzE/edit?usp=sharing)
   and put your name on it, one person per row.

You should be comfortable reading basic Splunk SPL. No prior AI or agent experience needed.

## How the Day Runs

| Part | Minutes | Format |
|------|---------|--------|
| Kickoff | 15 | ATHF, the scenario, the alert |
| Module 1 — Manual hunt | 30 | Hands-on: install ATHF, hunt by hand in Splunk, write it down (L0 → L1) |
| Module 2 — Search your history | 15 | Hands-on: find the next hunt from the last one (L2) |
| Module 3 — Research agent | 35 | Demo: an AI agent researches the open lead (L3) |
| Module 4 — Assistant runs the hunt | 40 | Demo: the assistant reads your hunts and runs the next one (L4) |
| Wrap | 10 | What to take home |

You leave with three documented hunts in your own ATHF workspace.

## What's Here

| Path | Contents |
|------|----------|
| `PRE_READ.md` | Ten-minute read before class |
| `labs/` | Lab handouts, one per module — [start here](labs/README.md) |
| `preflight/` | Laptop readiness check |

The answer keys (`labs/*-answer-key.*`) are included for anyone who wants to
check their work. The slide decks are posted here after the class.

## Who Built This

**Presenters** — Jarrett Polcari, Brandon Schwartz.

**Original workshop** — Sydney Marrone, Eric Brown, Jarrett Polcari.

**Testers** — Joshua Hines, Alex Hurtado.

**Infrastructure** — Ray Salamon.

## License

See [LICENSE](LICENSE).
