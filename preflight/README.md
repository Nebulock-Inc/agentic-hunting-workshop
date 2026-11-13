# Preflight check

Run this before class to confirm your laptop is ready. It uses only the Python standard
library, so it works before you install anything else.

```bash
python3 preflight/validate.py          # green/red checklist
python3 preflight/validate.py --json   # machine-readable
```

It checks Python (3.11+, which ATHF requires), that `athf` is installed, an LLM key
(optional: a warning is fine, the AI modules are demos), a text editor and a web browser.
Exit code 0 means no hard failures.

Optional: if you've been given a Splunk host to test, `WORKSHOP_SPLUNK_HOST=<host>:<port>
python3 preflight/validate.py` adds a network reach check. Without it the check is skipped;
in class you reach Splunk in your browser.
