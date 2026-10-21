# Blocking rm -rf in Claude Code: A Deny Rule Stopped 14 Deletes in Two Weeks, 7 Other Spellings Still Get Through, and a Hook That Closes the Gap

> 📍 Originally published at [MagicTools](https://tools.cooconsbit.com/en/articles/claude-code-deny-rm-rf-en?utm_source=github&utm_medium=referral). This mirror only carries a preview — **[read the full article →](https://tools.cooconsbit.com/en/articles/claude-code-deny-rm-rf-en?utm_source=github&utm_medium=referral)**

## The problem

Mid-task, a Claude Code command gets blocked and the model receives this:

```
Permission to use Bash with command cd /tmp && rm -rf smc2 && mkdir smc2 && cd smc2 && unzip -q ... has been denied.
```

No reason, no source. If you, like me, once wrote this into `~/.claude/settings.json`, you put it there yourself:

```json
{
  "permissions": {
    "allow": ["Bash"],
    "deny": ["Bash(rm -rf *)"]
  }
}
```

The intent is simple. Let Claude Code run whatever it needs, but don't let a stray `rm -rf` wipe something out. This post answers three questions. What does the rule actually block? What does it miss? How do you close the gap without leaving the model stuck?

## The short version

- **The deny rule only recognizes a literal `rm -rf `.** `rm -fr`, `rm -r -f`, `rm -Rf` and `rm -r` delete the directory in all three permission modes.
- **The model plays by the rules.** In two weeks it was denied 14 times and never once tried a different spelling. The cost is a pile of `xxx2` directories and stale output nobody removed.
- **There is a second gate you never configured.** Claude Code has a built-in dangerous-rm check for targets like `$D/*` that would hit the root if the variable were empty. It fires even in bypass mode, and in an interactive session it waits for a person to approve.
- **To close the gap, use a PreToolUse hook.** It runs before the other rules in all three modes, and it can tell the model why the delete was refused and what to do instead.

## Two weeks of real sessions: 14 blocked, 0 workarounds

I went through my Claude Code session logs from October 7 to 21, across 5 projects. I pulled every denied tool call that contained `rm` and looked at what the model did next.

| Denied command (excerpt) | What the model did next |
|---|---|
| `rm -rf $S && git checkout-index -a --prefix=$S/` | Started over in `snap2/` |
| `rm -rf .venv && uv venv -p 3.11 .venv` | Created `.venv311` |
| `cd /tmp && rm -rf smc2 && mkdir smc2 && unzip …` | Unzipped into a new directory under the project's tmp |
| `rm -rf wtt-film && … pipeline.py init wtt-film` | Switched to `wtt-film2` |
| `rm -rf $D; mariadb-install-db --datadir=$D` | Used `tmp/mariadb` instead |
| `rm -rf tmp/lt/patchtest && mkdir -p tmp/lt/patchtest …` | Added a timestamp to the directory name |
| `rm -rf /tmp/pc && mkdir -p /tmp/pc …` | Switched to `/tmp/pc2` |
| `cd /tmp && rm -rf hyperframes && git clone …` | Cloned somewhere else |
| `rm -rf node_modules package-lock.json; npm i …` | Skipped the delete and checked the current state |
| `rm -rf dial.mp4.frames sheet.png …` | Skipped the delete and moved on |

...

---

**[👉 Continue reading: Blocking rm -rf in Claude Code: A Deny Rule Stopped 14 Deletes in Two Weeks, 7 Other Spellings Still Get Through, and a Hook That Closes the Gap](https://tools.cooconsbit.com/en/articles/claude-code-deny-rm-rf-en?utm_source=github&utm_medium=referral)**

More articles: [tools.cooconsbit.com/articles](https://tools.cooconsbit.com/en/articles?utm_source=github&utm_medium=referral)
