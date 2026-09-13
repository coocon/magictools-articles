# 上周教你手写 git worktree，这周发现有人做成了一条命令：worktrunk 实测

> 📍 本文首发于 [MagicTools 码农早餐](https://tools.cooconsbit.com/zh/articles/worktrunk-hands-on?utm_source=github&utm_medium=referral)。镜像仓库仅收录预览，**[点此阅读全文 →](https://tools.cooconsbit.com/zh/articles/worktrunk-hands-on?utm_source=github&utm_medium=referral)**

上一篇讲 `git worktree`，结尾留了个尾巴：新开的目录是一份干净源码，`node_modules` 得再装一遍。我给的解法是「留一个常驻目录别删」，或者换 pnpm。发出去第二天就看到 [worktrunk](https://github.com/max-sixty/worktrunk) 上了热榜：一个专门管 worktree 的命令行工具，7.4k 星，Rust 写的，作者 Max Sixty（也是 PRQL 的作者）。

它的卖点一句话：**让 worktree 像分支一样好用**。原生 git 开一个 worktree 要把分支名打三遍（`git worktree add -b feat ../repo.feat && cd ../repo.feat`），它只要 `wt switch -c feat`。

好用的包装很多，我更关心两件事：它到底省掉了哪几步，以及那个 `node_modules` 的尾巴它怎么收。于是在我自己的项目里跑了一遍，仓库 1.9 GB 的 `node_modules`，APFS 文件系统，worktrunk 版本 0.77.0。

## 安装：多一步「shell 集成」，原因值得知道

```bash
brew install worktrunk && wt config shell install
```

第二段命令很多人会跳过，然后发现 `wt switch` 不切目录。原因是一个 Unix 常识：**子进程改不了父进程的工作目录**。`wt` 是个二进制，它自己 `cd` 了，你的 shell 还在原地。所以它要在你的 `.zshrc` 里注册一个同名 shell 函数，包住真正的二进制：二进制把目标路径写进一个临时文件，函数读出来再 `builtin cd`。

我没装集成直接跑，它会明确告诉你目录没切，并给出路径。命令本身照常执行。如果你只在脚本或 CI 里用，可以不装。

## 建、列、删：确实是一条命令

在我的仓库里开一个新分支的 worktree：

```
$ wt switch --create wt-trial
✓ Created branch wt-trial from main and worktree @ ~/4khz/magictools.wt-trial
```

0.65 秒。目录放在仓库旁边，命名是 `仓库名.分支名`。分支名只打了一次。路径规则是个模板，可以改成放在 `.worktrees/` 里或集中到 `~/worktrees/` 下，在配置文件里改一行。

`wt list` 是我觉得比原生好最多的地方。原生 `git worktree list` 只给路径和提交号，这个给状态：

```
  Branch    Status  HEAD±  main↕  main…±  Remote⇅  Path                    Commit   Age  Message
@ main          ^|                            |     .                       4f02fb1  16m  chore: bump 1.4.230
+ wt-trial      _                                   ../magictools.wt-trial  4f02fb1  16m  chore: bump 1.4.230
```

`@` 是当前所在，`HEAD±` 是未提交改动的行数，`main↕` 是领先落后主分支几个提交，`Remote⇅` 是有没有没推的。开着五六个目录时，一眼能看出哪个该合、哪个该删。

...

---

**[👉 继续阅读全文：上周教你手写 git worktree，这周发现有人做成了一条命令：worktrunk 实测](https://tools.cooconsbit.com/zh/articles/worktrunk-hands-on?utm_source=github&utm_medium=referral)**

更多文章：[tools.cooconsbit.com/articles](https://tools.cooconsbit.com/zh/articles?utm_source=github&utm_medium=referral)
