---
name: council
description: Trigger this if the user says 'council'.
---

# Council

Before an audit, address outstanding PR comments, fix known issues, and run
relevant checks. Give council the updated implementation; do not defer known
fixes until reviews return. If the user wants to discard the implementation
and explore new ideas, skip fixes to code being replaced.

Bring in three delegates to help with the user's request, usually for review
or feedback. Use one strong model from each of three different providers, at
high reasoning effort. If the user's instructions name council members, use
those. Otherwise pick the latest flagship model from each provider, for
example Claude Opus, GPT Sol, and Gemini Pro.

Start the delegates through the headless CLIs, so this works the same from
Claude Code, Codex, pi, or Cursor's `agent`:

| Member | Command |
| --- | --- |
| Claude | `claude -p --model <id> --effort high --permission-mode plan --output-format json "<brief>"` |
| GPT | `codex exec -m <id> -c model_reasoning_effort=high -s read-only -C <repo> --json -o <out-file> "<brief>" </dev/null` |
| Grok | `agent -p --model <id> --mode plan --trust --workspace <repo> --output-format json "<brief>" </dev/null` |
| Any other (e.g. Gemini) | `pi -p --model <provider/id>:high --exclude-tools edit,write --session-id <uuid> "<brief>"` |

- Resolve model IDs at run time: `pi --list-models <search>` lists what each
  provider exposes, and `agent --list-models` lists Cursor's. That list can
  lag the TUI: a newer bare ID such as `grok-<version>` may work with
  `--model` before it shows up, so try it before you fall back to an older
  listed one. Bare IDs may not accept an `[effort=...]` override; if so, run
  them at their default effort and say so. Do not hardcode IDs.
- Run the three in parallel: one shell command that starts each in the
  background with output to its own file under a temp dir, then `wait`. In
  Claude Code, Bash `run_in_background` works too. Give the call a long
  timeout.
- Save each session ID for later rounds: `session_id` from Claude's JSON,
  `thread_id` from Codex's `thread.started` event, `session_id` from
  `agent`'s JSON, and the `--session-id` you chose for pi. Follow up with
  `claude -p --resume <id>`, `codex exec resume <id>`,
  `agent -p --resume <id>`, or `pi -p --session-id <id>`.
- For your own provider you may use the harness's native subagent instead.
- Redirect stdin from `/dev/null` for `codex exec` and `agent -p`, or they
  wait for input. Never pass `--force`/`--yolo` to `agent`; `--mode plan`
  keeps it read-only.
- Delegates get no history. Put the repo path, base, diff command, and intent
  in the brief.

If you cannot reach three providers, or a named member is unavailable, report
it instead of substituting.

Give each delegate the request and relevant context. For review or feedback,
keep their work read-only. Collect their responses and summarize the useful
findings, disagreements, and recommended next steps.

For a code review, ask each delegate for a score out of 10.
Count each full review pass as one round.
A fresh review after fixes starts a new round.

If the work ends in a pull request, put the results in the PR description.
Put this section before the model and harness line at the end:

```markdown
## Council review

Rounds: 2

| Delegate | Round 1 | Round 2 |
| --- | --- | --- |
| Claude Opus | 7/10 | 9/10 |
| GPT Sol | 8/10 | 10/10 |
| Gemini Pro | 7/10 | 9/10 |
```

If a delegate gives no score, write `no score`. Do not invent one.
If the PR is open, edit the description. Do not post a comment for this.
If there is no PR, put the same results in the final reply only.

If you agree with their recommendations, make the changes and request a fresh
review after. Continue this process until all delegates are satisfied with the
outcome.

Do not let repeated reviews result in scope creep. It is your job to fight to
keep scope limited.
