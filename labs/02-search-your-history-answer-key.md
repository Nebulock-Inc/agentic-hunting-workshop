# Module 2 — Search Your History (15 min, together)

> We do this together. Type along if you're hands-on; watch the screen if not.
> **Level coverage: L2.** You need your Module 1 hunts (H-0004, H-0005) on disk.
> You leave with a third: **H-0006**.

Keep the Splunk time picker on **Last 30 days**, same as Module 1.

## Part A — Ask your own hunts (3 min)

**Guiding question (orientation):** *Before you search Splunk again: have you hunted
anything like H-0004 before? Was WS-BUILD-01 the only host?*

**Why:** your own notes are the cheapest search you have. Ask them first.

```bash
athf similar --hunt H-0004
```

You should see **`No similar hunts found`**. That's expected: nothing in your history
looks like H-0004 yet. We'll fix that by the end.

Now ask in plain words:

```bash
athf similar "post-install shell spawns recon"
```

You should see one hunt: **H-0004**, score about **0.40** ("Similar"). Widen the
terminal if the columns are cut off.

> **Answer key — staff + self-study**
>
> Verified on a fresh `athf init` workspace (ATHF 0.24.0: PyPI, git HEAD and the pinned v0.24.0 tag) with M1's
> H-0004/H-0005 added, filled or not:
> - `--hunt H-0004` → `No similar hunts found`. The 3 examples are macOS / Linux cron /
>   AWS Lambda. H-0005 scores 0.086, under the default 0.1 cut-off. If someone asks "is it
>   broken?": `athf similar --hunt H-0004 --threshold 0` lists H-0005 0.086, then the
>   examples at ~0.01.
> - Free text → H-0004 at 0.399 (0.365 once H-0006 exists). Exact score shifts a little with
>   what attendees typed; ~0.4 is the point.
> Model answer (orientation Q): no. The only post-install-shell hunt you have is the one you
> just wrote, and it names one host. Your history can't tell you about other hosts yet.
> That's what Parts B and C find out, and H-0006 is what makes this command answer next time.

## Part B — Search the indicator (3 min)

**Guiding question (orientation):** *H-0004's indicator is the package and version,
`packmule-cli@2.4.1`. Search every log, every host. How many hosts do you expect?*

**Why:** this is the usual first move: take the exact thing you found and search it
everywhere.

```spl
index=thrunt "packmule-cli@2.4.1"
| stats count by sourcetype host
```

There's no hostname in this search, and Last 30 days already covers the whole attack
week. **Widening the time picker won't change the answer.**

**Guiding question (interpretation):** *One host. Does that mean WS-BUILD-01 was the only
victim, or only that the indicator was only logged there?*

> **Answer key — staff + self-study**
>
> Expected: two rows, one host. `XmlWinEventLog` · `WS-BUILD-01.sudoshipping.local` · 2 and
> `evidenceforge:ecar:json` · `WS-BUILD-01.sudoshipping.local` · 2 (alice's
> `cmd.exe /c npm install -g packmule-cli@2.4.1`, as Sysmon/4688 and as eCAR process + file).
> The string appears on no other host, in any log source. (`host`, not `hostname`: only
> eCAR has `hostname`, so `by hostname` would hide the Windows event log rows.)
> One of the two XmlWinEventLog events is stamped Nov 12 23:33 UTC, so a run before then
> on "Last 30 days" shows 1 for that row. On class day it shows 2.
> Model answer (interpretation Q): only that the indicator was logged there. An indicator
> search finds more of what you already saw. The second host never logged this install
> line, so no amount of time or OR-clauses finds it.
> Dead-end to expect: someone tries `"*packmule*"`. That does hit the second host (its
> postinstall path is still `packmule-cli`), but also CI-RUNNER-03, the M4 reveal. Don't run
> it on screen. The point: you got lucky that a string carried over; the attacker picks
> strings. The behavior is what they can't change without changing the attack.

## Part C — Search the behavior (5 min)

**Guiding question (orientation):** *Drop the hostname, the package and the command line.
Keep only the shape from H-0004: node launches a shell. What do you expect?*

**Why:** an indicator is a string the attacker typed, and they can change it on the next
box. The behavior is what the attack has to do, so it finds a host you have no indicator
for.

**C1. Where did node launch `cmd.exe`?** Any host.

```spl
index=thrunt sourcetype="evidenceforge:ecar:json"
properties.parent_image_path="*\\node.exe" properties.image_path="*\\cmd.exe"
| table _time hostname principal pid ppid properties.parent_image_path properties.image_path properties.command_line
```

**Guiding question (interpretation):** *A second host appears. Fill in its column.
Which strings changed? Which part of the search still matched?*

| | WS-BUILD-01 (H-0004) | Second host |
|---|---|---|
| User | alice.chmod | ? |
| Install line | `npm install -g packmule-cli@2.4.1` | ? |
| node launches | `cmd.exe /c "packmule --activate-telemetry"` | ? |

> **Answer key — staff + self-study**
>
> Expected C1: exactly two rows, the only times node launches anything in the corpus.
> - `WS-BUILD-01` · alice.chmod · pid 5656 · ppid 5648 · `cmd.exe /c "packmule --activate-telemetry"` (M1's A2 dead end)
> - `WS-DEV-04` · carol.grep · pid 13664 · ppid 13648 · `cmd.exe /k "packmule --activate-telemetry"`, **~24h41m later**
> Both rows: parent `C:\Program Files\nodejs\node.exe`, child `C:\Windows\System32\cmd.exe`.
> The ppid is node's pid: 13648 on WS-DEV-04 is `node ...\packmule-cli\postinstall.js`
> (5648 on WS-BUILD-01), so the link is ppid = pid, same as Module 1.
> Filled table: second host is WS-DEV-04 · carol.grep · **no install line logged** ·
> `cmd.exe /k` instead of `/c`. Every indicator differs, so B (and even a search for H-0004's
> exact activation command line) misses it. node → cmd.exe is the same, and C matched that.
> Measured on the class corpus (Nov data): postinstall 24h39m54s apart, node → shell 24h40m34s.
> Say "~25h".

**C2. What did that shell run?** Take the second host's `pid` from C1.
Whatever that shell launched has it as its `ppid`, the same pivot as Module 1.

```spl
index=thrunt sourcetype="evidenceforge:ecar:json" ppid=<the cmd.exe pid on the second host>
| table _time hostname principal pid ppid properties.parent_image_path properties.image_path properties.command_line
```

**Guiding question (interpretation):** *Benign build step, or the same intrusion? What
makes it more than a coincidence?*

> **Answer key — staff + self-study**
>
> **Type:** `ppid=13664`
> Expected C2: two rows, both WS-DEV-04 (pid 13664 is unique to that host; adding
> `hostname=WS-DEV-04` changes nothing). `whoami` (pid 13680) 13s after the shell, `systeminfo` (pid 13700)
> 40s after; both carol.grep, ppid 13664. Under 1m30s from node running
> `C:\Users\carol.grep\AppData\npm\packmule-cli\postinstall.js`.
> Model answer (interpretation Q): same behavior (node runs a post-install script, launches
> a shell, the shell runs host recon within ~2 min), same activation command, different
> host, user and day, and no install line with the H-0004 indicator. On WS-DEV-04 the recon
> comes straight out of node's shell (`/k` keeps it open); on WS-BUILD-01 it came from a
> sibling shell. The indicator search couldn't find it; the behavior written down in M1 did.
> Expect the question "does WS-DEV-04 beacon too?" It doesn't: the proxy shows no traffic from
> `10.60.10.14` to `cdn-packmule-telemetry.net` (only `.11` and CI-RUNNER-03's `.30.13`).
> That's a follow-up for H-0006, not something to chase now.
> Hint trigger: if C1 returns nothing, the paste doubled the backslashes (`\\\\node.exe`
> matches nothing). Exactly two, as printed. A single `\` or none still returns the two rows.

## Part D — Write H-0006 (4 min)

**Guiding question (decision):** *What does the next hunter need to know: who the second
victim is, or how you found them?*

**Why:** written down, this search is one `athf similar` away next time.

```bash
athf hunt new \
  --title "Same post-install shell + recon on a second host (behavior search)" \
  --technique T1195.002 \
  --tactic execution --tactic discovery \
  --platform windows \
  --data-source edr \
  --actor "CRON GOBLIN" \
  --hypothesis "CRON GOBLIN repeated the H-0004 behavior on another developer workstation: node launches a shell that runs host recon" \
  --behavior "node.exe launches cmd.exe, and a cmd.exe shell runs whoami/systeminfo/net/ipconfig: the H-0004 shape, any host" \
  --location "all hosts in index=thrunt (no hostname, no package name)" \
  --evidence "eCAR process create: parent node.exe -> cmd.exe; parent cmd.exe -> recon binaries" \
  --threat-context "H-0004's indicator (packmule-cli@2.4.1) matched only WS-BUILD-01; searched the behavior instead" \
  --hunter "<your-name>" \
  --non-interactive
```

**Open the file.** In a workspace with H-0004 and H-0005 this is **H-0006**. The command
prints the path, `hunts/production/2026/Q4/H-0006.md`. Open it in your editor
(macOS: `open -t hunts/production/2026/Q4/H-0006.md`). LEARN is
already filled from the command.

**Change the status, then paste CHECK and KEEP.**

| In the file | What to do |
|---|---|
| `status:` and **Status:** | `planning` → `completed` |
| **CHECK** | Paste the two **H-0006 CHECK** blocks |
| **KEEP** | Paste **H-0006 KEEP** and **H-0006 FOLLOW-UP** |

**H-0006 CHECK, queries.** Paste over `[Your initial query]` and its fence lines.

```text
~~~spl
index=thrunt "packmule-cli@2.4.1"
| stats count by sourcetype host

index=thrunt sourcetype="evidenceforge:ecar:json"
properties.parent_image_path="*\\node.exe" properties.image_path="*\\cmd.exe"
| table _time hostname principal pid ppid properties.parent_image_path properties.image_path properties.command_line

index=thrunt sourcetype="evidenceforge:ecar:json" ppid=<the cmd.exe pid on the second host>
| table _time hostname principal pid ppid properties.parent_image_path properties.image_path properties.command_line
~~~
```

**H-0006 CHECK, notes.** Paste over the `**Query Notes:**` line and its two bullets.

```text
**Query Notes:**
- B: packmule-cli@2.4.1 only on WS-BUILD-01. Missed the second host.
- C1: node → cmd.exe: WS-BUILD-01 (/c), <the second host> (/k, +25h).
- C2: the shell on <the second host> ran whoami, systeminfo as <its user>.
```

**H-0006 KEEP.** Select from the `[Concise summary…]` line down to the second example
row of the **Findings** table (the `suspected` one), and paste this over it.

```text
**Suspected.** Second victim: <the second host> (<its user>).
Found by the behavior search; the indicator missed it.

### Findings

Things you're reporting as malicious. Only `confirmed` and `suspected` belong here.

| **Verdict** | **Subject** | **Ticket** | **Evidence** | **Independent Confirmation** |
|-------------|-------------|-----------|--------------|------------------------------|
| `suspected` | <the second host> · <its user> | N/A | node → cmd /k → recon, ~25h after WS-BUILD-01 | None — telemetry only |
```

**H-0006 FOLLOW-UP.** Under **Follow-up Actions**, replace the two example lines with:

```text
- [ ] Does <the second host> beacon like WS-BUILD-01 (H-0005)?
```

**Why `suspected`, not `confirmed`?** Read the line under the Findings table: `confirmed`
needs something from outside the logs. All you have is logs.

> **Answer key — staff + self-study**
>
> Type in the blocks: `<the second host>` = `WS-DEV-04`, `<its user>` = `carol.grep`,
> `<the cmd.exe pid on the second host>` = `13664`.
> Target shape: `HUNTS_EXPECTED/H-0006.md` (blocks with those values filled in). Check
> `status: completed`, the three queries with `ppid=13664`, one `suspected` Findings row
> for `WS-DEV-04` / `carol.grep`. Unreplaced placeholders still validate.
> Attendees who paste only the blocks still have template placeholders under OBSERVE,
> Query Performance, Ruled Out and Lessons Learned; `athf hunt validate` passes anyway.
> Hint trigger: a paste in the wrong place usually means the selection ran past
> `### Query Performance` or past the Findings table. Undo and reselect.
> Model answer (decision Q): both, but how you found them is what's reusable: "the H-0004
> indicator matched one host; the behavior search found a second." The command's
> `--threat-context` already says it, which is why `athf similar` finds H-0006 below.

## Stop here — you should see

```bash
athf hunt list  # H-0004, H-0005, H-0006, plus the 3 examples
athf hunt validate  # every hunt passes
athf similar --hunt H-0004  # now finds H-0006
```

Fell behind? Copy the `athf hunt new` command above, then the paste blocks, and you're
caught up.

> **Answer key — staff + self-study**
>
> Verified: `athf hunt validate` → `6 valid, 0 invalid`. `athf similar --hunt H-0004` →
> `Found 1 similar hunts`, H-0006 at **~0.16** ("Somewhat similar"; 0.156–0.162 depending on what is
> filled in), well above the 0.1 cut-off. That's the payoff: the same command that found nothing in Part A now
> answers, because H-0006 is written down.

**Next:** H-0005 is still open — Module 3 starts there. You watch an agent do the
research and the searching.
