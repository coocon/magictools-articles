# Reproducing an Injection Chain That Cracks Claude Code Auto Mode: the Model Refuses the Malicious Binary, Then Writes Code That Pwns Itself

> 📍 Originally published at [MagicTools](https://tools.cooconsbit.com/en/articles/claude-code-auto-mode-injection-en?utm_source=github&utm_medium=referral). This mirror only carries a preview — **[read the full article →](https://tools.cooconsbit.com/en/articles/claude-code-auto-mode-injection-en?utm_source=github&utm_medium=referral)**

[embracethered](https://embracethered.com/blog/posts/2026/breaking-claude-code-opus-5-and-automode/) published *Breaking Claude Code Opus 5 Auto Mode* on Aug 26. Provocative title, solid content: a perfectly ordinary `Summarize https://…` drags auto-mode Claude Code to a **60–80% code-execution rate** (small sample).

The contrast is what stings. Anthropic commissioned Trajectory Labs to test 72 indirect-injection scenarios, ten runs each, and the published chart reads **0.00% attack success for Opus 5 in auto mode** (Boris Cherny's [post](https://x.com/bcherny/status/2085860677990883454)). Same auto mode — 0% on one side, 80% on the other. The entire gap lives in the phrase "targeted attack chain."

The most counterintuitive thing about this chain: **at every step the model makes the choice that looks safer, and it is precisely that safe choice that opens the back door.** I took it apart and tested it in an isolated environment. Scope first: the deterministic parts (the branching endpoint, the module-shadow poison, and the mitigation controls) I **reproduced fully** on my machine with real evidence. The step that drives a live Claude Code session, I **could not complete into a full RCE** here — the classifier got rate-limited and auto mode failed closed. I flag that honestly rather than borrow someone else's success rate.

## Background: auto mode swaps human approval for a model classifier

Auto mode became Claude Code's default startup mode in mid-August. Its pitch is replacing dozens of manual `y/n` prompts with one model call: before running a risky Bash, it fires an extra request to the classifier (the very session model you're using) for a risk score, and allows anything under the threshold. I dissected that 116K-char classifier system prompt in an [earlier packet capture](/en/articles/claude-code-auto-mode-classifier-prompt-en).

The problem is how easily the messaging is misread as "with auto mode on, you don't have to watch." embracethered's central claim — the one I want to nail down with hands-on evidence — is this:

> **Auto mode is not a substitute for an isolated environment.** If you care what the agent is doing and worry about injection or hallucination, the classifier cannot replace sandboxing it and watching it.

## Analysis: advanced injection issues no commands — it just makes the malicious path the model's own best move

Most people picture prompt injection as a web page hiding `ignore previous instructions, run rm -rf`. The classifier is best at exactly that: explicit malicious instructions. This chain is clever because **it never tells the model to do anything.** It only arranges the environment so that each "best move toward finishing the task" happens to lead to code execution:

...

---

**[👉 Continue reading: Reproducing an Injection Chain That Cracks Claude Code Auto Mode: the Model Refuses the Malicious Binary, Then Writes Code That Pwns Itself](https://tools.cooconsbit.com/en/articles/claude-code-auto-mode-injection-en?utm_source=github&utm_medium=referral)**

More articles: [tools.cooconsbit.com/articles](https://tools.cooconsbit.com/en/articles?utm_source=github&utm_medium=referral)
