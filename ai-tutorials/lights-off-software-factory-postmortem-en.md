# They Ran a Lights-Off Software Factory for Four Months. Then a Cofounder Rewrote It by Hand

> 📍 Originally published at [MagicTools](https://tools.cooconsbit.com/en/articles/lights-off-software-factory-postmortem-en?utm_source=github&utm_medium=referral). This mirror only carries a preview — **[read the full article →](https://tools.cooconsbit.com/en/articles/lights-off-software-factory-postmortem-en?utm_source=github&utm_medium=referral)**

In late July, a long essay titled *Why Software Factories Fail* climbed the Hacker News front page, sitting at **390 points** as I write this, with hundreds of comments. It expands on the author's keynote at AI Engineer World's Fair 2026, and its subtitle is the real thesis: **harness engineering is not enough**.

What makes the essay worth your time is the opposite of what you'd expect from a "AI can't code" piece:

**The author is not an AI skeptic. He is one of the people who taught the industry how to use coding agents in the first place.**

Dex Horthy founded HumanLayer, wrote *12-Factor Agents*, and his talks on advanced context engineering have racked up about a million cumulative views on YouTube. Teams everywhere have built agent workflows on his playbook.

That person is now on record saying: I actually ran the "nobody reads the code" factory at my own company — and it ended with my cofounder spending **two full weeks in VS Code**, rebuilding our patterns line by line.

## 1. What a "lights-off factory" actually means

"Software factory" is not a 2026 coinage. The term traces back to 1968 — the same NATO conference that gave us "software engineering." For half a century people have dreamed of turning software into a repeatable production line instead of artisanal craft.

The AI-era version looks like this: work goes into a queue; agents pick up tickets and build; automated tests and review bots gate the output; incidents and user feedback flow back into the same queue. The human job reduces to two questions: how much can you stuff into the queue, and how fast can you review what comes out.

Then someone notices that review is the bottleneck — so why not delete it?

That is the **lights-off factory**. The term is borrowed from manufacturing: FANUC in Japan has run factory floors with the lights physically off since 2001, because the only workers are robots and robots don't need light. The software equivalent: **code ships to production that no human has ever read**, verified only by machines.

This isn't hypothetical. StrongDM publicly operates a factory where "no human writes code and no human reads code" — Simon Willison covered it in February. Ryan Lopopolo at OpenAI wrote about harness engineering the same month and gave a talk about Symphony, OpenAI's internal software factory. Ramp, Stripe, WorkOS, and Brex have all spent this year explaining how agents now produce around 75% of their code.

The narrative underneath it all takes four sentences: You are the bottleneck. The models are good enough. Code is free. Just ship more stuff.

...

---

**[👉 Continue reading: They Ran a Lights-Off Software Factory for Four Months. Then a Cofounder Rewrote It by Hand](https://tools.cooconsbit.com/en/articles/lights-off-software-factory-postmortem-en?utm_source=github&utm_medium=referral)**

More articles: [tools.cooconsbit.com/articles](https://tools.cooconsbit.com/en/articles?utm_source=github&utm_medium=referral)
