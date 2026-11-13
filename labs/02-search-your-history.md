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

**C2. What did that shell run?** Take the second host's `pid` from C1.
Whatever that shell launched has it as its `ppid`, the same pivot as Module 1.

```spl
index=thrunt sourcetype="evidenceforge:ecar:json" ppid=<the cmd.exe pid on the second host>
| table _time hostname principal pid ppid properties.parent_image_path properties.image_path properties.command_line
```

**Guiding question (interpretation):** *Benign build step, or the same intrusion? What
makes it more than a coincidence?*

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

## Stop here — you should see

```bash
athf hunt list  # H-0004, H-0005, H-0006, plus the 3 examples
athf hunt validate  # every hunt passes
athf similar --hunt H-0004  # now finds H-0006
```

Fell behind? Copy the `athf hunt new` command above, then the paste blocks, and you're
caught up.

**Next:** H-0005 is still open — Module 3 starts there. You watch an agent do the
research and the searching.
