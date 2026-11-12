# Pre-Read — Before You Arrive

> ~10 minutes. Read this once before the workshop so we can start hunting at minute one instead of explaining vocabulary. You don't need to memorize anything — you need the four ideas below to feel familiar when we say them out loud.

---

## What you're walking into

Two and a half hours (150 minutes), four modules. You'll hunt a supply-chain compromise inside a fake company (Sudo Shipping) against a shared Splunk instance we've pre-seeded. It's all synthetic — no real customer data, no real IOCs. You'll use **ATHF** (the Agentic Threat Hunting Framework, open source, `pip install 'agentic-threat-hunting-framework[all]'`).

Modules 1 and 2 are hands-on: you hunt by hand in Splunk and write the hunts down with ATHF. Modules 3 and 4 are demos on the presenter's screen: an AI assistant researches the open lead, then reads the hunts you wrote and runs the next one. **No LLM API key needed** — the AI parts run on our laptop, not yours.

You'll leave with **three documented hunts in your own workspace** (H-0004, H-0005, H-0006). That's the deal: not slides, actual reusable hunt records.

Come with: a laptop, a web browser, Python 3.11+, ATHF installed, and the preflight script run green. Your Splunk login comes from the shared credentials sheet you'll be given in the room.

---

## Idea 1 — The Five Levels of Agentic Hunting (L0–L4)

This is the spine of the day. Each module climbs one level (Module 1 covers the first two, L0 → L1). It's a **stack, not a ladder** — you don't abandon lower levels, you build on them.

- **L0 — Ad-hoc.** Hunting in a search bar. Queries live in browser tabs; when you close the laptop, the hunt is gone. Where most of the field lives today.
- **L1 — Structured.** You write the hunt down — hypothesis, scope, what you ruled out — so it survives you and someone else can re-run it.
- **L2 — Reused.** You find the *next* hunt by reusing the *behavior* documented in the last one, not the indicators. Behavior transfers across hosts and time; indicators don't.
- **L3 — Researched.** An AI agent reads threat intel and hands you hypotheses you wouldn't have written alone. You still decide what's real.
- **L4 — Agentic.** The agent has tools. It reads your hunt repo, tells you which parts of the kill chain you *haven't* covered, and proposes the next query.

The punchline of the whole workshop: at L4, the agent can only find your blind spots because L1–L3 wrote the hunts down. **Memory is the edge.** (ATHF's docs number these L0–L4 — five levels, zero-indexed. Same thing as "five levels.")

## Idea 2 — LOCK: the shape of a hunt

Every hunt you write follows **LOCK**:

- **L — Learn:** what triggered this hunt? Intel, an anomaly, a coverage gap.
- **O — Observe:** the hypothesis and the query — what behavior would this actor leave behind?
- **C — Check:** run it, triage the hits, rule things out.
- **K — Keep:** the finding, the exact query, the verdict, and the branches you ruled out.

One thing to internalize now: **a ruled-out hypothesis is a finding.** The hunt that found nothing still tells the next hunter where not to look. Dead ends get written down too.

## Idea 3 — ABLE: how you scope a hypothesis

ABLE is the vocabulary that goes *inside* the Observe step. A good hypothesis names four things:

- **A — Actor:** who (real or hypothesized)?
- **B — Behavior:** what do they *do*? (Not what tool — what action.)
- **L — Location:** where in your environment would it show up?
- **E — Evidence:** what data proves or disproves it?

Example of the shape: *"CRON GOBLIN (actor) uses a package-manager post-install script to spawn a shell (behavior) on developer workstations (location); evidence is a `node.exe → cmd.exe` process chain in EDR events (evidence)."* If you can write that sentence, you can hunt it.

## Idea 4 — Behavior over indicators

The core hunting mindset. Think of the Pyramid of Pain: hashes and IPs are trivial for an adversary to change; TTPs are expensive. The compromise you'll hunt uses a **signed** binary with a **clean hash** and **no CVE** — so indicator-based detection is useless by design. The only way in is behavior: the *shape* of what the adversary does, which stays constant even when the hash, domain, and hostname all change.

Everything you find will be behavioral. When you write hunts, describe the behavior — that's the part that transfers to the next host, the next campaign, the next module.

---

## What the ATHF workspace looks like

When you run `athf init`, you get a small, legible workspace:

- **`hunts/`** — your hunt records live here as Markdown (`H-XXXX.md`). This directory *is* your memory. It's what `athf similar` searches and what the MCP coverage tools reason over.
- **`AGENTS.md`** — context that tells an AI agent how your workspace is shaped and how to behave in it.
- **`research/`** — research documents (`R-XXXX.md`) written from threat intel (Module 3).

A handful of commands you'll use or see (don't memorize — you'll have the labs):

- `athf hunt new …` / `athf hunt validate` — write a hunt and check its format (L1).
- `athf hunt list` / `athf hunt coverage` — see what you've done and what you haven't (L2).
- `athf similar --hunt H-XXXX` — find related prior hunts (L2).
- `athf research new …` — research a topic from intel (L3, demo).
- ATHF's MCP server — lets an AI assistant read your hunts and propose the next one (L4, demo).

---

## One mindset note

You'll hit walls on purpose. The modules have designed moments where the manual approach dead-ends and an ATHF capability unsticks it. That's not you failing — that's the lesson. When you're stuck for more than a minute or two, raise a hand; keeping the whole room on pace matters more than solving it alone.

During the labs you'll work from the **lab handouts** (`labs/` in the workshop repo) — each
step leads with a guiding question, then the search to run. Don't worry about capturing the
answers in the moment: after the workshop we post the **answer keys** (expected findings and
model answers) and the slide decks, so you can replay any module on your own.
Hunt first, self-study later.

See you there. Come ready to hunt.
