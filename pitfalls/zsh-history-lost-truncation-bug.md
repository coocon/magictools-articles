# Zsh 历史记录莫名丢失？一个潜伏 10 年的 bug，改配置没用，升级 5.9.2 才是解法

> 📍 本文首发于 [MagicTools 码农早餐](https://tools.cooconsbit.com/zh/articles/zsh-history-lost-truncation-bug?utm_source=github&utm_medium=referral)。镜像仓库仅收录预览，**[点此阅读全文 →](https://tools.cooconsbit.com/zh/articles/zsh-history-lost-truncation-bug?utm_source=github&utm_medium=referral)**

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

...

---

**[👉 继续阅读全文：Zsh 历史记录莫名丢失？一个潜伏 10 年的 bug，改配置没用，升级 5.9.2 才是解法](https://tools.cooconsbit.com/zh/articles/zsh-history-lost-truncation-bug?utm_source=github&utm_medium=referral)**

更多文章：[tools.cooconsbit.com/articles](https://tools.cooconsbit.com/zh/articles?utm_source=github&utm_medium=referral)
