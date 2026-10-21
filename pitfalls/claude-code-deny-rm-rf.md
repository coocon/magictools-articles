# Claude Code 禁止 rm -rf 怎么配：deny 规则两周拦下 14 次，实测还有 7 种写法照样能删，一个 hook 补齐

> 📍 本文首发于 [MagicTools 码农早餐](https://tools.cooconsbit.com/zh/articles/claude-code-deny-rm-rf?utm_source=github&utm_medium=referral)。镜像仓库仅收录预览，**[点此阅读全文 →](https://tools.cooconsbit.com/zh/articles/claude-code-deny-rm-rf?utm_source=github&utm_medium=referral)**

## 问题背景

Claude Code 跑着跑着，一条命令被挡下，模型收到的是这样一行：

```
Permission to use Bash with command cd /tmp && rm -rf smc2 && mkdir smc2 && cd smc2 && unzip -q ... has been denied.
```

没有理由，没有出处。如果你和我一样，曾经在 `~/.claude/settings.json` 里亲手写过这么一条，那它就是你自己加的：

```json
{
  "permissions": {
    "allow": ["Bash"],
    "deny": ["Bash(rm -rf *)"]
  }
}
```

加它的目的很朴素：让 Claude Code 随便跑命令，但别让它一个 `rm -rf` 把东西删没了。这篇回答三个问题：这条规则实际拦下了什么，它漏了什么，怎么把漏洞补上又不把模型卡死。

## 结论先说

- **这条 deny 规则只认字面上的 `rm -rf `。**`rm -fr`、`rm -r -f`、`rm -Rf`、`rm -r` 在三种权限模式下都能删掉目录。
- **模型很守规矩。**本机两周 14 次被拒，模型没有一次换写法绕过去，代价是留下一堆 `xxx2` 目录和没删掉的旧产物。
- **还有一道你没配过的闸。**Claude Code 内置的 dangerous-rm 检查会拦 `$D/*` 这类「变量为空就变成删根目录」的写法，bypass 模式也拦，交互模式下会停下来等人批准。
- **要补齐，用 PreToolUse hook。**它在三种模式下都先于其他规则生效，还能把「为什么不让删、该怎么办」直接告诉模型。

## 两周的真实记录：拦下 14 次，0 次绕过

我翻了本机 10 月 7 日到 21 日 5 个项目的 Claude Code 会话记录，找出所有带 `rm` 且被拒的工具调用，再看模型下一步干了什么。

| 被拒的命令（节选） | 模型下一步 |
|---|---|
| `rm -rf $S && git checkout-index -a --prefix=$S/` | 换成 `snap2/` 重来 |
| `rm -rf .venv && uv venv -p 3.11 .venv` | 新建 `.venv311` |
| `cd /tmp && rm -rf smc2 && mkdir smc2 && unzip …` | 解压到项目 tmp 下的新目录 |
| `rm -rf wtt-film && … pipeline.py init wtt-film` | 改用 `wtt-film2` |
| `rm -rf $D; mariadb-install-db --datadir=$D` | 换成 `tmp/mariadb` |
| `rm -rf tmp/lt/patchtest && mkdir -p tmp/lt/patchtest …` | 目录名加上时间戳 |
| `rm -rf /tmp/pc && mkdir -p /tmp/pc …` | 改用 `/tmp/pc2` |
| `cd /tmp && rm -rf hyperframes && git clone …` | clone 到另一个位置 |
| `rm -rf node_modules package-lock.json; npm i …` | 不删了，先看当前状态 |
| `rm -rf dial.mp4.frames sheet.png …` | 跳过删除，继续后面的活 |

完整统计：

| 模型反应 | 次数 |
|---|---|
| 换一个新目录名或新位置重来 | 8 |
| 放弃删除，只做命令里的其他部分 | 6 |
| 换成 `rm -fr` / `find -delete` / `/bin/rm` 等写法绕过 | 0 |

我又在这些会话的全部 Bash 调用里搜了 `rm -fr`、`find -delete`、`/bin/rm -r`、`shutil.rmtree`、`git clean -f`，一条都没有。

这说明主人的意图被尊重了。但有两个代价：

1. **整条复合命令都不执行。**被拒的命令大多是「删旧目录 + 重建 + 跑测试」连在一起的，`rm` 只是其中一段，结果后面的构建和测试也一起没跑，模型要多花一轮拆开重发。
2. **垃圾目录越积越多。**`snap` 和 `snap2`、`.venv` 和 `.venv311`、`wtt-film` 和 `wtt-film2` 并排躺着。被拒文案里没有任何指引，模型只能自己想办法，最省事的办法就是换个名字。

## 实验：22 种删除写法，逐字回放

光看真实记录只能说明「模型没去绕」，说明不了「规则能不能被绕」。我在 `/tmp` 下搭了一个隔离实验：

- **模型由本地 stub 扮演。**它把指定的 Bash 命令原样作为 `tool_use` 发给 Claude Code，再把 Claude Code 回给模型的 `tool_result` 原文记下来。客户端行为是真实的，模型回复是模拟的。
- **每次全新环境。**空的 `CLAUDE_CONFIG_DIR`、假 API key、外连一律拦截，删的是每次新建的沙箱目录 `s/`，里面放 `a/x.txt`、`a/sub/y.txt`、`b.txt`、`t_1.jpg`、`t_2.jpg`。
- **用磁盘结果判对错。**跑完列出沙箱剩下什么，不看模型说了什么。
- **三种权限模式。**同一份 settings（allow `Bash` + deny `Bash(rm -rf *)`），分别用手动模式（`--permission-mode default`）、auto 模式、bypass 模式（`--dangerously-skip-permissions`）跑。

...

---

**[👉 继续阅读全文：Claude Code 禁止 rm -rf 怎么配：deny 规则两周拦下 14 次，实测还有 7 种写法照样能删，一个 hook 补齐](https://tools.cooconsbit.com/zh/articles/claude-code-deny-rm-rf?utm_source=github&utm_medium=referral)**

更多文章：[tools.cooconsbit.com/articles](https://tools.cooconsbit.com/zh/articles?utm_source=github&utm_medium=referral)
