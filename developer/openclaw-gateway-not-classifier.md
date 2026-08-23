# openclaw 一半命令能跑一半不能——这不是权限在拦，是网关根本没起来

> 📍 本文首发于 [MagicTools 码农早餐](https://tools.cooconsbit.com/zh/articles/openclaw-gateway-not-classifier?utm_source=github&utm_medium=referral)。镜像仓库仅收录预览，**[点此阅读全文 →](https://tools.cooconsbit.com/zh/articles/openclaw-gateway-not-classifier?utm_source=github&utm_medium=referral)**

两个 Claude Code 窗口，同一台 Mac，同一个用户，同一份全局 `settings.json`。窗口 A 在 `~/4khz/tingease`，窗口 B 在 `~/4khz/tingease/AudioCore`。同样的 `openclaw` 命令，B 跑得好好的，A 反复失败。

用户给出的判断很自然：「你的分类器是好的，另一个窗口的却不能执行。」

这个怀疑有充分理由——Claude Code 的 auto 权限模式确实会在放行 Bash 前调一次模型做安全判定，判定链一断就全面拦截。同样的坑我们写过一篇（见文末）。但这次不是。真相在完全不同的一层，而把人引向权限假设的，恰恰是故障本身的形状。

## 反直觉的地方：不是全挂，是挂了一半

先看现象。这两条命令在同一个窗口里，前一条正常输出：

```
$ openclaw --help
OpenClaw 2026.7.1-2 (0790d9f) — All your chats, one OpenClaw.
Usage: openclaw [options] [command]
...
```

后一条失败：

```
$ openclaw health
[openclaw] Could not start the CLI.
[openclaw] Reason: gateway closed (1006 abnormal closure): no close reason
Gateway target: ws://127.0.0.1:18789
```

**同一个二进制，有的子命令通，有的不通。** 这个形状极具误导性：如果是网络故障或者进程崩了，应该全军覆没；能出结果的那部分说明二进制本身、PATH、执行权限统统正常。剩下的解释里，「有个东西在逐条筛选命令」听起来最合理。

但换个切法就通了。openclaw 的子命令天然分两类：

| 类型 | 例子 | 是否连网关 |
|------|------|-----------|
| 纯本地 | `--help`、`daemon status`、`plugins list` | 否，直接读文件和配置 |
| 走网关 | `health`、`gateway status`、`channels`、`agent` | 是，连 `ws://127.0.0.1:18789` |

**通的全是第一类，不通的全是第二类。** 这不是筛选，是一条共享依赖断了——所有依赖它的命令一起倒下，不依赖的毫发无伤。分界线不在命令的危险程度上，在它连不连那个 WebSocket 上。

这条线索本该一步到位，但当时我并没有先看它。下面三个假设是我实际走过的弯路。

## 假设一：两个窗口的权限配置不一样

最直接的解释：窗口 A 所在的项目目录有额外的权限限制。查两处配置。

全局 `~/.claude/settings.json` 的 `permissions.allow` 里有这么一条：

```json
"allow": ["Bash", "Read", "Edit", ...]
```

一条**裸的 `Bash`**——不带任何括号限定，放行所有 Bash 命令。这条对两个窗口一视同仁，因为全局配置本来就不区分项目。

再查窗口 A 那个项目的 `.claude/settings.local.json`：

```json
{
  "permissions": {
    "allow": ["Bash(cd:*)", "Bash(swift build:*)", "Bash(git status:*)", ...]
  }
}
```

只有 `allow`，没有 `deny`。而 Claude Code 的项目配置是**叠加**到全局上的，不是覆盖——一个只有 allow 的文件不可能把已经放行的东西收回去。

**假设一出局。** 权限层面两个窗口不存在任何差异。

## 假设二：那个配置文件本身坏了

退一步想：如果 `settings.local.json` 解析失败呢？带 BOM、有不可见字符、JSON 语法错——解析一挂，整个项目的权限体系可能降级到"事事确认"。

...

---

**[👉 继续阅读全文：openclaw 一半命令能跑一半不能——这不是权限在拦，是网关根本没起来](https://tools.cooconsbit.com/zh/articles/openclaw-gateway-not-classifier?utm_source=github&utm_medium=referral)**

更多文章：[tools.cooconsbit.com/articles](https://tools.cooconsbit.com/zh/articles?utm_source=github&utm_medium=referral)
