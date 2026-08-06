---
title: "openclaw 一半命令能跑一半不能——这不是权限在拦，是网关根本没起来"
slug: openclaw-gateway-not-classifier
category: developer
locale: zh
translationSlug: openclaw-gateway-not-classifier-en
tags: [openclaw, launchd, 故障排查, CLI 工具, 权限管理, 开发者工作流]
summary: "同一台机器、同一个用户、同一份全局配置，两个 Claude Code 窗口里跑同一个 CLI，一个通一个不通。最像的解释是权限分类器把其中一个拦了——毕竟它确实会拦。但真相在另一层：守护进程的 plist 装好了却没被 launchd 加载，于是所有要连网关的子命令全挂，纯本地的子命令照常输出。这种「一半能跑一半不能」的形状，正是把人骗向权限假设的东西。本文记录完整排查：三个假设怎么被逐个推翻，包括我自己误读日志的那一次。"
status: published
---

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

这个假设很好证伪，两条命令：

```bash
$ xxd .claude/settings.local.json | head -1
00000000: 207b 2020 0a20 2020 2022 7065 726d 6973   {  .    "permis

$ python3 -c "import json; json.load(open('.claude/settings.local.json')); print('OK')"
OK
```

文件确实有点脏——第一个字节是 `0x20`，也就是说 `{` 前面有个空格，行尾还有两个。但没有 BOM（不是 `EF BB BF` 开头），JSON 解析器也照收不误。

**假设二出局。** 前导空白是合法 JSON 的一部分，看着别扭，不影响任何事。

## 假设三：日志里的孤立记录说明命令被拒了

这台机器上挂了 hook，把每次工具调用记进 `~/tools/logs/claude-hooks-*.jsonl`。我按 session 分组，两个窗口清清楚楚：

```
('159f3a23', '/Users/duoduo/4khz/tingease')            37 条   ← 窗口 A
('cb28e90b', '/Users/duoduo/4khz/tingease/AudioCore')  41 条   ← 窗口 B
```

翻窗口 A 的记录，我注意到一件事：好几条 `PreToolUse` **没有配对的 `PostToolUse`**。

```
[00:50:09] PreToolUse       ← 无配对
[00:50:33] PreToolUse       ← 无配对
[00:50:53] PreToolUse
[00:50:53] PostToolUse      ← 配对完整
[00:51:03] PreToolUse       ← 无配对
```

推理链条当时看着很硬：`PostToolUse` 只在工具真正执行后触发，所以孤立的 `PreToolUse` 就是被权限拦下的调用。这下不但坐实了权限假设，还有了精确到秒的证据。

**这个推理是错的**，而且错在一个很典型的地方：`PostToolUse` 缺失只能说明「这次调用没有正常走完」，它不区分是被权限拒绝、命令执行报错、超时，还是用户中途打断。我把一个多解的信号当成了单解的证据——恰好那个解还正中我的预设。

真正能分辨的实验，是去那个窗口的工作目录直接跑一次：

```bash
$ cd /Users/duoduo/4khz/tingease && openclaw gateway status
CLI version: 2026.7.1-2 (/opt/homebrew/bin/openclaw)
Gateway version: 2026.7.1-2
Runtime: running (pid 80719, state active)
Connectivity probe: ok
Capability: admin-capable
```

**假设三出局。** 同一个目录、同一条命令，完全正常。cwd 无关，权限无关。

## 决定性证据是时间线，不是状态

上面那条命令还暴露出一个更关键的问题：**它现在是好的**。而窗口 A 当时是坏的。状态快照回答不了"当时发生了什么"，能回答的只有时间线。

把 hook 日志的时间戳和我接手时的实测状态并排放：

```
00:49:58  [窗口 A]  "你来看看系统的 openclaw 有啥问题 修复好"
00:52:36  [窗口 A]  "启动失败 请修复 openclaw"
01:12:10  [窗口 A]  "openclaw gateway status"
01:1x     [窗口 B]  我接手，第一条诊断命令 → 见下
```

我接手跑的第一条是 `openclaw daemon status`，输出是这样的：

```
Service: LaunchAgent (not loaded)
Runtime: unknown (Bad request.
  Could not find service "ai.openclaw.gateway" in domain for user gui: 501)
Connectivity probe: failed
  connect ECONNREFUSED 127.0.0.1:18789
```

`ECONNREFUSED` 意味着 18789 端口上压根没有进程在监听。窗口 A 从 00:49 挣扎到 01:12 的整段时间里，网关就没起来过。

而窗口 B 之所以"能跑 openclaw"，唯一原因是我第一步就执行了：

```bash
$ openclaw daemon start
Gateway LaunchAgent was installed but not loaded; re-bootstrapped launchd service.
```

不是我的调用方式更聪明，是我把服务修好了。**同一时刻的两个窗口从来没有被区别对待过——它们只是在不同时刻访问了同一个共享服务，而那个服务的状态在中间被改变了。**

## 根因：装了，但没被加载

那句提示已经说明一切：`installed but not loaded`。

launchd 管理服务分两步，plist 文件躺在 `~/Library/LaunchAgents/` 只完成了第一步；还得被 bootstrap 进用户的 launchd domain 才真正受管。这次是第一步成了、第二步没成，于是形成一个很尴尬的中间态：

- `openclaw daemon status` 能读到 plist，报告"服务已安装"
- launchctl 里查不到这个 label，报 `Could not find service ... in domain for user gui: 501`
- 端口无人监听，所有走网关的子命令 `ECONNREFUSED`

`openclaw daemon start` 做的就是补上第二步。

修完顺手确认了持久化配置，避免下次重启复发：

```bash
$ plutil -p ~/Library/LaunchAgents/ai.openclaw.gateway.plist | grep -E "RunAtLoad|KeepAlive"
  "KeepAlive" => true
  "RunAtLoad" => true

$ launchctl print-disabled gui/501 | grep openclaw
  "ai.openclaw.gateway" => enabled
```

`RunAtLoad` 管登录自启，`KeepAlive` 管崩溃拉起，两个都是 `true`，也没被 `launchctl disable` 过。配置本身是健全的，那次未加载是一次性的意外，不是设计缺陷。

服务是全机共享的，修好之后两个窗口同时恢复——窗口 A 不需要改任何配置。

## 这件事的普遍教训

**同环境不同结果，先查共享的有状态依赖，别先查各自的配置。**

配置是每个窗口自己的，服务是全机共享的。当两个环境的差异是"时间"而不是"设置"时，只有共享的有状态组件解释得通。我把顺序搞反了，先花时间比对两份 settings、验 JSON 合法性，而这些都是无状态的——**无状态的东西不会只在某个时间段坏掉**。这个反问本可以在三分钟内把我推到正确的方向：这两个窗口有什么是共享的，且会随时间变化？

另外三条：

- **"一部分功能能用"是链路信息，不是筛选信息。** A 类命令通、B 类不通，第一反应不该是"有东西在挑"，而是"这两类走的路不一样，分岔点在哪"。找到那条分界线（这次是"连不连 WebSocket"），根因基本就浮出来了。
- **日志的缺失是多解信号。** 假设三的翻车就在这：`PostToolUse` 没出现，可能是被拒、报错、超时、打断中的任何一种。用多解信号去确认一个你已经倾向的假设，是排查里最容易掉进去的坑——它给你的是信心，不是信息。
- **状态快照证明不了历史。** "我现在跑一遍是好的"无法反驳"你刚才是坏的"。要还原当时，得靠带时间戳的记录，并且把它和另一条独立时间线对齐——这次是 hook 日志的 prompt 时间戳，对上我接手时的 `ECONNREFUSED`。

一句话总结：当一个 CLI 有一半命令能跑一半不能，先别怀疑有人在筛选它们，去找那条把它们分成两半的依赖——大多数时候，被筛掉的那一半只是共用了一个已经躺下的服务。

配套阅读：站内的[《Claude Code 的 auto 模式是用模型审模型的》](/zh/articles/claude-code-auto-mode-permission-trap)讲的是权限分类器**真的**故障时长什么样，正好和本文构成一对照——那篇里"读文件正常、跑命令全挂"的半瘫是分类器造成的，这篇里"一半命令能跑"的半瘫跟权限一点关系没有。两篇放在一起看，能训练出区分它们的直觉：**看不通的那些命令是按危险程度分的，还是按依赖分的。**
