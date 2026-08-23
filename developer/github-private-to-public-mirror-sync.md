# 私有仓库的文章如何自动同步到公开镜像站

> 📍 本文首发于 [MagicTools 码农早餐](https://tools.cooconsbit.com/zh/articles/github-private-to-public-mirror-sync?utm_source=github&utm_medium=referral)。镜像仓库仅收录预览，**[点此阅读全文 →](https://tools.cooconsbit.com/zh/articles/github-private-to-public-mirror-sync?utm_source=github&utm_medium=referral)**

> 一次实战记录：用 GitHub Actions 把私有仓库的文章自动推送到公开镜像站，实现内容分发三角的最后一环。

---

## 背景

我有一个 [MagicTools](https://tools.cooconsbit.com) 项目，代码托管在 GitHub 的私有仓库里。项目里有 150+ 篇 Markdown 文章（AI 开发教程、Claude 指南、工具评测等）。

问题是：**私有仓库的文章，外面的人看不到**。

而我们的内容分发策略是构建一个「内容三角」：

```
Reddit 讨论 → YouTube 视频 → GitHub 文章
                ↑                  ↓
                ←—— 互相引流 ——————←
```

GitHub 上的文章是这个三角的重要支点——GitHub 页面 Google 收录好、SEO 权重高、分享 URL 干净。但如果仓库是私有的，这一切都是零。

解决思路很直接：**建一个公开的「影子仓库」，只放文章，不放代码。然后用 GitHub Actions 自动同步。**

---

## 方案总览

```
┌──────────────────────────┐         ┌──────────────────────────┐
│  私有仓库 magictools       │  push   │  公开仓库 magictools-     │
│                           │────────→│  articles                │
│  所有代码 + 文章          │ Actions │  只包含 articles/ 目录    │
│  articles/ 目录           │  自动   │  + README 索引            │
│                           │  同步   │                          │
└──────────────────────────┘         └──────────────────────────┘
```

**两个仓库的关系**：

| 项目 | 私有仓库 (magictools) | 公开仓库 (magictools-articles) |
|------|----------------------|-------------------------------|
| 可见性 | 私有 | 公开 |
| 内容 | 全部代码 + 文章 | 仅文章 |
| 更新方式 | 手动 push | GitHub Actions 自动推送 |
| 用途 | 开发 & 部署 | 内容分发 & SEO |

---

## 实施步骤

### 第一步：创建公开镜像仓库

```bash
gh repo create coocon/magictools-articles --public \
  --description "MagicTools 文章镜像 · AI 开发教程、Claude 指南、工具评测"
```

首次初始化：把私有仓库里 `articles/` 目录的内容全部复制过来，加上一个索引 README：

```bash
git clone https://github.com/coocon/magictools-articles.git
cd magictools-articles
cp -r /path/to/magictools/articles/* .
git add -A && git commit -m "feat: 初始文章导入"
git push
```

...

---

**[👉 继续阅读全文：私有仓库的文章如何自动同步到公开镜像站：GitHub Actions 跨仓库推送方案](https://tools.cooconsbit.com/zh/articles/github-private-to-public-mirror-sync?utm_source=github&utm_medium=referral)**

更多文章：[tools.cooconsbit.com/articles](https://tools.cooconsbit.com/zh/articles?utm_source=github&utm_medium=referral)
