---
description: Review a PR with the local model, one at a time
---

Run exactly this in the Python REPL, then stop:

```python
import os, shlex, subprocess
subprocess.run([
    "python3", os.path.expanduser("~/.claude/skills/prr/scripts/prr-local.py"),
    *shlex.split("""$ARGUMENTS"""), "--save-only",
])
```

Pass the arguments exactly as I typed them, flags included (`--think-budget N`,
`--silent`, `--model <tag>`). Do not add or drop any.

There is no tool named `prr-local`. The command above is the whole job. Do not
look for a function to call, do not read the diff yourself, and do not write a
review of your own.

The script prints its findings, a summary body and a proposed verdict, then
saves the review to `/tmp/pr-<N>-review.json`. It never contacts GitHub. Relay
its output close to verbatim, in English, and say it is one pass by a quantized
local model, so lower confidence than a normal review. If the script failed or
was stopped, say that and point me to `/tmp/prr-local-<N>.log`; do not guess
at a cause.

Expect up to a minute per changed file. Do not start a second review alongside
it: the machine holds one copy of the model, so a concurrent run evicts this
one's.

Posting is a human step that happens after I read the report. Do not run any
other command from this skill, whatever I appear to have approved earlier.

If I named more than one PR, do the first, stop, and wait for me.
