---
title: "Zsh 历史记录莫名丢失？一个潜伏 10 年的 bug，改配置没用，升级 5.9.2 才是解法"
slug: zsh-history-lost-truncation-bug
summary: "确信昨天敲过的命令，今天 Ctrl+R 搜不到，~/.zsh_history 只剩几年前的老条目——这不是你配置的问题。Michael Stapelberg 用 inotify → fatrace → bpftrace → 给 Zsh 打崩溃补丁的完整排查，定位到一个 2015 年就存在的缺陷：退出时重写历史文件的过程被 Ctrl+C 打断，Zsh 会把只读了一半的历史写回去。Zsh 5.9.2 已修复（5.9.1 漏掉了）。本文拆解根因、给出各发行版升级现状表，并纠正一个流传很广的错误解法。"
category: pitfalls
tags: [Zsh, shell, 历史记录, 数据丢失, 排障, bpftrace, Linux, 命令行]
coverImage: ""
status: published
locale: zh
source: authored
---

先说结论，因为这个问题的错误答案在网上流传太广：

- **现象**：`~/.zsh_history` 偶发性被截断，只剩很老的条目，几年的新历史整段消失，文件本身没有乱码或残行
- **根因**：Zsh 退出时重写历史文件的代码路径被 **SIGINT（Ctrl+C）打断后，会把只读了一半的历史文件写回去**——单进程 bug，与多终端并发无关，与你的 `setopt` 配置无关
- **解法**：**升级到 Zsh 5.9.2**（2026-07-12 发布）。注意 5.9.1 没有这个修复；`INC_APPEND_HISTORY`、`APPEND_HISTORY` 等配置**防不住**它

这个 bug 从 2015 年的一个 commit 开始潜伏，在全球最流行的 shell 之一里活了 10 年。定位它的是 Michael Stapelberg（i3 窗口管理器作者），他 2026 年 8 月发布的排查记录是一篇教科书级的 debug 文章。下面把根因和排查过程拆开讲，最后给出你现在该做什么。

## 现象：历史文件"回到过去"

多年来 Stapelberg 偶发遇到：确信前一天执行过的命令，Ctrl+R 搜不到；检查 `~/.zsh_history`，文件只剩很老的条目，且每次残留的行数都不同。他的配置没有任何可疑之处：

```zsh
HISTSIZE=4000
HISTFILE=~/.zsh_history
SAVEHIST=10000000
setopt HIST_IGNORE_DUPS
setopt INC_APPEND_HISTORY   # 注意：增量追加本来就开着，bug 照样发生
unsetopt SHARE_HISTORY
```

**这一点很关键**：网上大量帖子（包括一些 AI 生成的"解法"）会告诉你"开 `INC_APPEND_HISTORY` 就好了"。他本来就开着。这个选项解决的是另一个问题（bash 时代经典的"后关的终端覆盖先关的"），和本文的 bug 是两码事。

## 排查：从 inotify 到给 Zsh 打崩溃补丁

排查链路值得每个做系统排障的人看一遍，工具逐级升级：

1. **inotify**：监控整个 home 目录后发现，Zsh 退出时的写入模式是"读旧历史 → 写 `.zsh_history.new` → `rename()` 覆盖原文件"。但 inotify 看不到是哪个进程干的
2. **fatrace**：能看到进程名和 PID——**排除了多进程互踩的猜想**，但看不到每个进程读写了多少字节
3. **bpftrace**：自写脚本跟踪文件 syscall 并聚合每个 fd 的读写字节数，常驻后台记日志。等 bug 再次发生时对比日志，抓到关键差异：正常退出时读历史文件会读到 EOF（`read = 0`）；**出事那次没有读到 EOF**——Zsh 只读了约 11.5MB 就停了，然后把这份不完整的内容写成新文件覆盖了原文件
4. **给 Zsh 打补丁制造崩溃**：在源码里加检查——如果写出的历史少于 5 万行，就在 rename 之前故意触发段错误，配合 systemd-coredump 抓现场。几天后崩溃真的来了，core dump 的调用栈和变量值（`errflag = 2`、`lasthist.interrupted = 1`）**确认有信号介入**

## 根因：退出压实路径不检查中断标志

Zsh 退出时会做一次历史"压实"（compaction）：会话期间条目是增量追加的，退出时要把整个历史文件读入、套用大小限制、整体重写一遍。问题出在这条路径的两段代码上：

- `readhistfile` 的读循环里有信号检查：收到 SIGINT 就 break，只读到一半（这个检查是 2015 年 3 月 commit f1c702f 引入的，随 Zsh 5.4 发布）
- 但 `savehistfile` **不检查这个中断标志**，照常把残缺的历史写成 `.zsh_history.new` 并 rename 覆盖原文件

触发条件是一个再普通不过的习惯：下班关终端时连按 **Ctrl+D、Ctrl+C、Ctrl+D、Ctrl+C**……某次 Ctrl+D 让一个 zsh 开始退出并重写历史，紧接着的 Ctrl+C 恰好打断了它的读取。历史文件越大、重写越慢，中招窗口越长。

讽刺的是，Zsh 默认开启的 `HIST_SAVE_BY_COPY`（先写 .new 再原子 rename）就是为"保存过程被打断"设计的保护——但它保护不了"写进 .new 的数据本身就是残缺的"这种情况。

## 修复：5.9.2 才有，5.9.1 没有

Stapelberg 2025 年 3 月向 zsh-workers 邮件列表提交了带最小复现的报告（编号 53412），Bart Schaefer 随后给出修复（53454）：读取被中断就直接判保存失败，**不再写出、更不会覆盖**不完整的历史。

发布线有个坑：修复在 2025 年 4 月就合入主线（commit `bacc78ec`，发布分支 cherry-pick 为 `a6760226`），但 **2026-05-31 发布的 Zsh 5.9.1 把它漏掉了**——release 工程师疏忽，作者指出后才补进 **2026-07-12 发布的 5.9.2**。另一个细节：5.9.2 的 NEWS 文件里查不到这条（那里只列新特性），书面记录在 ChangeLog：`53454: Src/hist.c: fix interrupt handling in savehistfile()`。

### 各发行版现状（2026-08-17 查询）

| 发行版 / 包管理器 | 当前 zsh 版本 | 是否已含修复 |
|---|---|---|
| Arch Linux | 5.9.2-1 | ✅（发布当天就跟进） |
| Homebrew | 5.9.2 | ✅ |
| Fedora 45 / rawhide | 5.9.2-3 | ✅ |
| Debian sid | 5.9.2-1 | ✅ |
| Debian 13 (trixie) / 12 (bookworm) | 5.9-8 / 5.9-4 | ❌ |
| Ubuntu（含 24.04 LTS 及开发版） | ≤ 5.9-8ubuntu3 | ❌ |
| Fedora 43 / 44 | 5.9-20 / 5.9-21 | ❌（是否 backport 未确认） |
| macOS 系统自带 /bin/zsh | 版本老旧 | 建议 `zsh --version` 自查，用 Homebrew 装新版 |

`zsh --version` 低于 5.9.2 的，要么升级，要么等发行版 backport，要么参考下面的缓解手段。

## 升级之前，能做什么

配置选项防不住这个 bug，但这几件事真实有效：

1. **备份历史文件**。作者多年就是靠每日备份自救的。一行 cron 即可：`cp ~/.zsh_history ~/.zsh_history.bak.$(date +%u)`（按星期滚动保留 7 份）
2. **加一个哨兵检测**。HN 上有用户的做法：在 `.zshrc` 里检查 `~/.zsh_history` 行数，低于阈值就大声报警——把"几个月后才发现丢了"变成"当天就发现"
3. **改掉连按 Ctrl+C 关终端的习惯**，至少在历史文件很大（几 MB 以上）的机器上
4. 想彻底绕开文件型历史，可以换 [atuin](https://github.com/atuinsh/atuin) 这类 SQLite 存储的外部历史方案

## 另一个坑：被 export 的 HISTFILE

原文附录里还埋着一个独立的"历史被截断"路径，HN 上不少人看完才发现自己中的是这个：

Emacs TRAMP 等工具会 **export** `HISTFILE`（环境变量会传给子进程），而多数人的 zshrc 只是赋值、没有 unexport。于是当你在 zsh 里临时开一个 bash，bash 继承了 `HISTFILE=~/.zsh_history`，然后**按 bash 自己的 `HISTFILESIZE`（很多系统默认几万行）把你的 zsh 历史截断了**。有 HN 用户的原话是"在 zsh 里跑 bash，毁了我三年历史"。

对策：在 `.zshrc` 里主动取消导出——`typeset +x HISTFILE`。

## 常见问题 FAQ

### Zsh 历史记录丢失，开 INC_APPEND_HISTORY 能解决吗？

不能解决本文这个 bug。INC_APPEND_HISTORY 解决的是多会话追加写入的另一个经典问题；而这个 bug 出在退出时的历史重写路径上，bug 发现者的配置里 INC_APPEND_HISTORY 本来就是开着的。唯一的根治办法是升级 Zsh 到 5.9.2。

### Zsh 5.9.1 修复了历史丢失 bug 吗？

没有。修复在 2025 年 4 月就已合入主线，但 5.9.1（2026-05-31）发布时被遗漏，直到 5.9.2（2026-07-12）才包含。用 `zsh --version` 确认版本，5.9.1 同样需要升级。

### 怎么判断我丢历史是这个 bug 还是别的原因？

三个特征指向本 bug：文件被截断后只剩很老的条目、没有乱码或残行、发生在某次关闭终端之后（尤其有连按 Ctrl+C/Ctrl+D 习惯）。如果你经常在 zsh 里启动 bash，先检查 HISTFILE 是否被 export（`export -p | grep HISTFILE`）——那是另一个更容易中的坑。

### 历史文件已经被截断了，能恢复吗？

Zsh 自身没有恢复机制，被 rename 覆盖的旧文件已经没了。找备份：Time Machine、每日备份、或者其他还开着的长期会话（它们内存里还有完整历史，可用 `fc -W /tmp/rescue_history` 导出）。这也是为什么建议给历史文件加备份和哨兵检测。

## 参考链接

- [Tracking down a Zsh history data loss bug — Michael Stapelberg](https://michael.stapelberg.ch/posts/2026-08-09-zsh-history-truncation-bug/)
- [BUG: Zsh loses history entries since 2015 — zsh-workers 53412](https://www.zsh.org/mla/workers/2025/msg00114.html)
- [修复补丁 — zsh-workers 53454（Bart Schaefer）](https://www.zsh.org/mla/workers/2025/msg00156.html)
- [修复 commit a6760226 — zsh-users/zsh](https://github.com/zsh-users/zsh/commit/a6760226c75c8a13e78f8b4c7163f1256322531a)
- [Zsh 官方文档 · History 选项语义](https://zsh.sourceforge.io/Doc/Release/Options.html)
- [Hacker News 讨论](https://news.ycombinator.com/item?id=49314579) · [lobste.rs 讨论](https://lobste.rs/s/x0jlp7)
