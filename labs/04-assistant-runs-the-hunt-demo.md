# Module 4 — The Assistant Runs the Hunt (40 min, instructor demo)

> **Watch the screen.** The instructor drives; you don't need to type anything.
> **Level coverage: L3–L4.** Claude (Claude Code) reads the hunts you wrote in Modules 1–3
> through ATHF's MCP server, finds what you haven't covered, runs the Splunk searches
> itself, and writes **H-0008**.
> Every prompt below is plain English, and each one ends with a limit ("then stop"), so
> Claude answers that question and nothing more. To replay this at home, see `MCP_SETUP.md`.

**What Claude can use:**

| Tool | What it gives Claude |
|---|---|
| ATHF MCP server (`athf-mcp`, comes with `athf[all]`) | Your hunts: list, search, read, coverage, create, validate |
| Splunk search helper (instructor's laptop only) | Runs one SPL search, returns the rows |
| A short rules file (`CLAUDE.md`) | Where the data lives, field names, "say why before each search" |

## Beat 1 — Memory (5 min)

**Guiding question (orientation):** *You wrote four hunts today. What should an assistant
be able to tell you about them without you opening a file?*

**Prompt:**

```text
What have I hunted before? One line each. Just list them, don't investigate
further.
```

## Beat 2 — Blind spots (10 min)

**Guiding question (decision):** *M1–M3 each started from a thread you could see. What
happened in this intrusion that nothing has pointed you at yet?*

**Prompts, in order:**

```text
Where are my coverage gaps? Just list them, don't search Splunk yet.
```

```text
Propose one Splunk query for the credential access gap: what would this
attacker steal from a developer's build machine? Use the field names from my
hunts' CHECK sections, and include pid and ppid. Show me the query, don't
run it yet.
```

## Beat 3 — Run and record (15 min)

**Guiding question (interpretation):** *Claude says it found something. What would you
check before you believe it?*

**Prompts, in order:**

```text
Run it. Just this one search, then tell me what it found, don't investigate
further.
```

```text
Show me everything on that host in the minute around that read, with pid and
ppid. Only that host and that minute, don't investigate further.
```

```text
Now check DNS for exfiltration, across all hosts: tunnelling shows up as
many unique names under one domain. Run it, then tell me which host sent
them and when. DNS only, then stop.
```

```text
Write this up as a hunt, including the findings: list in the front matter,
and validate it. Only what we found so far, no new searches.
```

**Fallback searches** (if the instructor needs to paste):

```spl
index=thrunt sourcetype="evidenceforge:ecar:json" object=PROCESS action=CREATE
(properties.command_line="*type*\\.env*" OR properties.command_line="*\\.aws\\credentials*"
OR properties.command_line="*findstr*password*.netrc*")
| table _time hostname principal pid ppid properties.parent_image_path properties.command_line
```

```spl
index=thrunt sourcetype="bro:dns:json" qtype_name=TXT
| rex field=query "(?<domain>[^.]+\.[^.]+)$"
| stats count dc(query) as uniq_names by domain
| sort - uniq_names
| head 10
```

## Beat 4 — Same actor, new place (8 min)

**Guiding question (decision):** *You wrote the C2 domain down in Module 1. What can one
question do with it now?*

**Prompts, in order:**

```text
Has any other host talked to the C2 domains in my hunts? Find the host name,
then stop, don't look at what it ran yet.
```

```text
What did that host run around its first C2 connection? Show the process
chain in time order, with pid and ppid, then stop.
```

```text
Follow that gitlab-runner session to its last command, searching the whole
day, then tell me when it started compared with the npm install in H-0004.
Then stop.
```

```text
Add this to H-0008, including the findings: list in the front matter, and
link it to H-0005 and H-0007. Validate it. No new searches.
```

**Fallback searches:**

```spl
index=thrunt sourcetype="apache:access:json" server="cdn-packmule-telemetry.net"
| stats count by client
```

```spl
index=thrunt sourcetype="evidenceforge:ecar:json" object=FLOW properties.direction=OUTBOUND
properties.src_ip="<the new client IP>"
| stats count by hostname
```

## Recap (2 min)

**Guiding question (reflection):** *Module 1 took 30 minutes for two hunts. What did
Claude need from you to do this in 40 — and what did it still need you for?*

```bash
athf hunt list
```

```bash
athf hunt coverage
```

**Take home:** `MCP_SETUP.md` connects Claude Code or Claude Desktop to your own hunts.
