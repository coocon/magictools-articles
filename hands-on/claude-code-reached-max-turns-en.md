# Claude Code \"Error: Reached max turns (1)\": when the headless guardrail fires, your file may already be written

> 📍 Originally published at [MagicTools](https://tools.cooconsbit.com/en/articles/claude-code-reached-max-turns-en?utm_source=github&utm_medium=referral). This mirror only carries a preview — **[read the full article →](https://tools.cooconsbit.com/en/articles/claude-code-reached-max-turns-en?utm_source=github&utm_medium=referral)**

This is part 3 of the Claude Code error-message series. Part 1 was "auto mode temporarily unavailable" and part 2 was [MCP "Failed to connect"](/en/articles/claude-code-mcp-failed-to-connect-en). This one covers a new pair of errors, from the two guardrails in headless mode (`claude -p`):

```text
Error: Reached max turns (1)
Error: Exceeded USD budget (0.01)
```

With `--output-format json` you don't get those lines at all. Instead you get `"subtype": "error_max_turns"`, `"terminal_reason": "max_turns"`, and a `result` field that **doesn't exist**.

![The two guardrail errors from claude -p; xxd shows they go to stdout with no trailing newline](https://cdn.tools.cooconsbit.com/uploads/articles/2026-09-25-max-turns/01-two-guardrail-errors.png)

## Background

If you run Claude Code in CI, cron or a batch script, you normally put limits on it:

- `--max-turns N` caps how many turns it takes, so the model can't loop on tool calls forever.
- `--max-budget-usd X` caps what it spends, so one job can't run up a large bill.

The [official CLI reference](https://code.claude.com/docs/en/cli-reference) (fetched 2026-09-25) gives each flag one sentence:

- `--max-turns`: "Limit the number of agentic turns (print mode only). Exits with an error when the limit is reached. No limit by default."
- `--max-budget-usd`: "Maximum dollar amount to spend on API calls before stopping (print mode only)."

That leaves out what a script author actually needs to know:

- Which stream does the error go to?
- What exit code do you get?
- What does the json look like?
- Does work the model already did still count when it gets cut off?
- What exactly does "1 turn" count?

I measured all of it.

## Analysis

Every run used the same three-step task:

1. Read the number on line 3 of `data.txt` (42).
2. Write that number times 2 into `out.txt`.
3. Reply with the final number.

When it succeeds, `out.txt` contains `84`. The task needs at least 3 model calls (Read → Write → reply), so it's easy to see what happens when the cut lands at each point.

Here's the short version. These guardrails behave in four ways you might not expect:

1. **The error goes to stdout, not stderr.** `2>/dev/null` won't hide it. It also has no trailing newline, so it runs into the next line of your log.
2. **In json mode there is no `result` key.** `jq -r .result` prints the four letters `null`, and jq itself exits 0. A downstream script will pass the string `"null"` along as if it were the model's answer. This is the least visible way these errors can break a pipeline.
3. **A failure doesn't mean nothing happened.** Tool calls issued on the Nth model call still run. When `--max-turns 2` fires, the file has already been written.
4. **The budget is checked after the fact.** Claude Code compares total spend only after an API call returns, so it can go over the cap by up to one call. That last call may have finished the job, and you still get exit 1.

...

---

**[👉 Continue reading: Claude Code \"Error: Reached max turns (1)\": when the headless guardrail fires, your file may already be written](https://tools.cooconsbit.com/en/articles/claude-code-reached-max-turns-en?utm_source=github&utm_medium=referral)**

More articles: [tools.cooconsbit.com/articles](https://tools.cooconsbit.com/en/articles?utm_source=github&utm_medium=referral)
