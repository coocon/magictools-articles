# 改到一半要修线上 bug？别 git stash，用 git worktree 再开一个目录

> 📍 本文首发于 [MagicTools 码农早餐](https://tools.cooconsbit.com/zh/articles/git-worktree-instead-of-stash?utm_source=github&utm_medium=referral)。镜像仓库仅收录预览，**[点此阅读全文 →](https://tools.cooconsbit.com/zh/articles/git-worktree-instead-of-stash?utm_source=github&utm_medium=referral)**

写到一半，群里一句「线上挂了，你看下」。你手里是一堆没提交的改动，测试还没跑通，现在要切到 `main` 上修一个 bug。

肌肉记忆告诉你：`git stash`，切分支，修完，切回来，`git stash pop`。这条路我走了很多年，每一步都踩过坑。后来发现 Git 早就给了另一条路：**同一个仓库，再开一个目录，检出另一个分支**。改到一半的东西原地不动，修 bug 去隔壁做。这就是 `git worktree`。

## stash 到底哪里不对

先说清楚，stash 不是坏东西，是被用错了场景。它设计出来是「临时把工作区收起来」，不是「在两个任务之间来回切」。用它切任务，有四个具体的坑：

**第一，默认不带未跟踪文件。** 你新建的文件不在 stash 里，切到 `main` 后它们还躺在工作区。修 bug 时一不留神就把它们一起提交进去。要带上得加 `-u`，要连 `.gitignore` 里的一起带得加 `-a`，很少有人每次都记得。

**第二，pop 回来会冲突。** 你在 `main` 上修的那几行，恰好也是你 feature 分支改过的地方，`git stash pop` 直接给你一屏冲突标记。这时候 stash 条目还在（pop 失败不会删），你得手动解冲突再 `git stash drop`，不少人在这一步把改动弄丢过。

**第三，stash 是本地的，不进远端。** 换台机器、仓库重新 clone、误删目录，stash 全没了。它不像分支可以 push。

**第四，忘掉的 stash 会堆成坟场。** 跑一下 `git stash list`，很多人能翻出十几条「WIP on feature-xxx」，没人记得里面是什么，也没人敢删。

这四条的根源是同一个：**stash 试图在一个工作区里装两件事**。解法不是把 stash 用得更熟，是再开一个工作区。

## worktree 是什么

`git worktree` 让一个仓库同时拥有多个工作目录。每个目录检出各自的分支，有各自的 `HEAD`、各自的暂存区，但共享同一份对象库和分支列表。你在任何一个目录里提交，其他目录立刻能看到这个提交。

一句话记：**clone 是复制仓库，worktree 是复制工作区。** 对象库只有一份，不占双倍磁盘，也不用来回 fetch。

回到开头的场景，现在的做法是：

```bash
# 在主目录（feature 分支，一堆没提交的改动）里执行
git worktree add ../myproj-hotfix main
```

...

---

**[👉 继续阅读全文：改到一半要修线上 bug？别 git stash，用 git worktree 再开一个目录](https://tools.cooconsbit.com/zh/articles/git-worktree-instead-of-stash?utm_source=github&utm_medium=referral)**

更多文章：[tools.cooconsbit.com/articles](https://tools.cooconsbit.com/zh/articles?utm_source=github&utm_medium=referral)
