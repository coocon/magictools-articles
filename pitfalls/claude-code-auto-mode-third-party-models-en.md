# Claude Code Auto Mode with DeepSeek, GLM or Kimi: What Each \"temporarily unavailable\" Reason Means and When Retrying Is Pointless (Fault-Injection Test)

> 📍 Originally published at [MagicTools](https://tools.cooconsbit.com/en/articles/claude-code-auto-mode-third-party-models-en?utm_source=github&utm_medium=referral). This mirror only carries a preview — **[read the full article →](https://tools.cooconsbit.com/en/articles/claude-code-auto-mode-third-party-models-en?utm_source=github&utm_medium=referral)**

> **Short answers first**
>
> - **Why does the error name `glm-5.3`, `deepseek-v4-pro` or `k3`?** Before every action with side effects (Bash, file writes, and so on), auto mode sends an extra safety-check request using **your session model** to **the backend you configured**. The name in the error is that classifier, which is simply your own session model.
> - **Look at the parentheses first.** `(server error)`, `(rate-limited)`, `(overloaded)` and `(connection failed)` mean the backend had a temporary problem; Claude Code already tried 5 times. `(timed out)` means the check got no response within 60 seconds.
> - **No parentheses is the one to watch.** The backend rejected the classifier request outright (a 4xx). Claude Code tries only once, and **retrying will give the same result every time**. Check your backend configuration.
> - **`Auto mode could not evaluate this action`**: the backend answered, but the model did not return a verdict in the expected format. After three of these in a row, auto mode falls back to manual approval.
> - **Want a separate model for the classifier?** We could not find a way. `CLAUDE_CODE_AUTO_MODE_MODEL` has no effect in 2.1.285. The only way to change the classifier is to change the session model.

## Background

Our guide [Claude Code "temporarily unavailable, so auto mode cannot determine the safety of bash": how to fix it](/en/articles/claude-code-auto-mode-temporarily-unavailable-fix-en) is the most-searched page on this site over the last 28 days. Many of the search queries reaching it name a model that is not Claude: `glm-5.3`, `glm5.2`, `glm-5.3-flash`, `claude-glm-5.3`, some with a `cc switch` prefix. In other words, a lot of people hit this error after pointing Claude Code at Zhipu, Kimi, DeepSeek or another third-party backend through tools like cc-switch or claude-code-router.

That guide covered this case in one line: "switch to a healthy backend." What these users actually want to know is:

1. Can a third-party model run auto mode at all? What does the classifier request look like, and what does it cost?
2. Which failure does each reason in the parentheses map to? Which ones go away if you wait, and which ones never will?
3. Can you give the classifier its own faster or cheaper model?

## How the check works

Auto mode replaces manual approval with one model call: before an action with side effects runs, a model judges whether it is safe (details in [our teardown of the auto mode classifier](/en/hands-on/claude-code-auto-mode-classifier-prompt-en)).

...

---

**[👉 Continue reading: Claude Code Auto Mode with DeepSeek, GLM or Kimi: What Each \"temporarily unavailable\" Reason Means and When Retrying Is Pointless (Fault-Injection Test)](https://tools.cooconsbit.com/en/articles/claude-code-auto-mode-third-party-models-en?utm_source=github&utm_medium=referral)**

More articles: [tools.cooconsbit.com/articles](https://tools.cooconsbit.com/en/articles?utm_source=github&utm_medium=referral)
