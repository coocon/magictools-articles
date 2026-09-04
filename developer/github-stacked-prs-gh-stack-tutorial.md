# GitHub 原生 Stacked PR 上手教程：gh stack 命令怎么用、怎么合并、老分支怎么转

> 📍 本文首发于 [MagicTools 码农早餐](https://tools.cooconsbit.com/zh/articles/github-stacked-prs-gh-stack-tutorial?utm_source=github&utm_medium=referral)。镜像仓库仅收录预览，**[点此阅读全文 →](https://tools.cooconsbit.com/zh/articles/github-stacked-prs-gh-stack-tutorial?utm_source=github&utm_medium=referral)**

先把搜索者最常问的几件事直接说清：

- **这是什么**：Stacked PR（堆叠式 PR）= 把一个大改动拆成一串有依赖顺序的小 PR，每个 PR 只含自己那一层的 diff，逐层评审，最后可以一键把整串合入。
- **什么时候上线的**：GitHub 于 **2026 年 7 月 30 日**将其作为原生功能开启公测（public preview），逐步推送到**所有仓库**——在此之前你只能靠 Graphite、spr、git-branchless 这类第三方工具模拟。
- **收费吗**：不收费，不是付费计划专属功能。CLI 扩展 `gh-stack` 免费安装。
- **在哪用**：github.com 网页、`gh` 命令行、GitHub 移动端都支持；还有配套的 AI agent skill。

## 一、五分钟上手：从零建一个 stack

前提：装好 [GitHub CLI](https://cli.github.com/)（v2.0+，Windows/macOS/Linux 都有官方安装包），然后装官方扩展：

```sh
gh extension install github/gh-stack
```

核心心智模型：一个 **stack** 是一列有序分支，每层建立在下一层之上，最底层基于主干（通常是 `main`）：

```
frontend      → PR #3 (base: api-endpoints) ← 顶层
api-endpoints → PR #2 (base: auth-layer)
auth-layer    → PR #1 (base: main)          ← 底层
─────────────
main (trunk)
```

完整流程：

```sh
# 1. 建 stack（创建并切到第一个分支）
gh stack init

# ...在第一层提交代码...

# 2. 在上面叠一层
gh stack add api-endpoints
# ...继续提交...

# 3. 推送所有分支
gh stack push

# 4. 查看整个 stack
gh stack view

# 5. 一次性开出整串 PR（每层一个，base 自动指向下一层）
gh stack submit
```

`submit` 之后，GitHub 会把这串 PR 链接成一个 Stack 对象：每个 PR 页面顶部出现 **stack map**，评审者能看到当前层在整串改动中的位置，且每个 PR 只显示本层的 diff。

...

---

**[👉 继续阅读全文：GitHub 原生 Stacked PR 上手教程：gh stack 命令怎么用、怎么合并、老分支怎么转](https://tools.cooconsbit.com/zh/articles/github-stacked-prs-gh-stack-tutorial?utm_source=github&utm_medium=referral)**

更多文章：[tools.cooconsbit.com/articles](https://tools.cooconsbit.com/zh/articles?utm_source=github&utm_medium=referral)
