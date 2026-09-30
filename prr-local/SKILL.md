---
name: prr-local
description: Review a GitHub pull request with the local Ollama model, as a single-source pass. Use this, never the full prr skill, whenever you are running on a local model and asked to review a PR.
---

# prr-local

> **Status (2026-09-30): no local model tested so far is accurate enough to
> trust.** Tried: `qwen3:8b`, `qwen2.5-coder:7b` and `gemma4:26b-a4b-it-qat`.
> Across three PRs with known-good answers, not one finding survived checking.
> The misses were judgement, not speed or anchoring: findings that re-raise
> problems the author's own comments already handle, or fixes that would break
> the code. Treat the output as a reading aid, and verify every finding before
> posting. This skill is kept so the next model can be tried without rebuilding
> it.

A single-source review of a pull request by a local model, sharing prr's setup
and posting scripts. It lives inside the prr skill directory on purpose: Claude
Code only discovers top-level skills, so it sees prr and not this; an agent
harness running a local model (prime-agent) lists this path explicitly and does
not load prr, whose procedure is far too long for a local context window.

The review runs as a deterministic pipeline, not as a procedure you follow: it
splits the diff per file, asks the local model about one file at a time under a
JSON schema, validates every line anchor against the diff, and derives the
verdict from the findings.

**Do not review the diff yourself.** Reading a whole PR through a local model
costs more than the pipeline does and produces worse anchors. Run the script.

## Review

```
python3 ~/.claude/skills/prr/scripts/prr-local.py <PR-url-or-number> --save-only
```

Add `--silent` to suppress chat signals. The default model is
`gemma4-26b-a4b-32k:latest` (override with `--model` or `$PRR_LOCAL_MODEL`).
It makes one call per changed file, so time scales with the file count: expect
seconds to a minute per file, plus ~15 s if the model has to load first. Stream
the output rather than buffering it, and do not start a second heavy request
alongside it.

Read the `reviewing N file(s)` line first. A PR you already reviewed with no
commits since runs in re-review mode against an empty diff, reviews zero files,
and still proposes APPROVE. That result means nothing.

The script prints the findings, the summary body, and a proposed verdict, then
saves `/tmp/pr-<N>-review.json` and stops. It posts nothing.

Relay its report to the user close to verbatim. It is one pass by a quantized
local model, so flag it as lower confidence than a normal prr review and do not
add findings of your own on top.

## Posting

Nothing reaches GitHub until a human reads the report and says yes **in this
conversation**. Their approval is the only thing that authorizes the next
command, so never run it pre-emptively, and never infer consent from an earlier
message about a different PR.

```
python3 ~/.claude/skills/prr/scripts/prr-local.py <PR> --post-saved APPROVE
```

Valid choices: `APPROVE`, `APPROVE_BARE` (approve with no inline comments),
`REQUEST_CHANGES`, `COMMENT`. This runs no inference; it posts the saved
review and cleans up. If commits landed since the review, it refuses rather
than posting a stale review, which is correct: re-run the review.

If the user declines, discard the review and remove the worktree:

```
~/.claude/skills/prr/scripts/post-review.sh <PR>
```

Do not hand-edit the saved file, and do not post with `gh` directly. The script
builds the payload from it, and post-review.sh validates the head sha.
