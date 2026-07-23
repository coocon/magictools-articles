---
title: "私有仓库的文章如何自动同步到公开镜像站：GitHub Actions 跨仓库推送方案"
slug: github-private-to-public-mirror-sync
summary: "如果你有一个私有 GitHub 仓库存放代码和文章，但又希望文章内容公开可访问，可以通过公开镜像仓库 + GitHub Actions 自动同步来实现。本文记录了一次完整的实施过程，包括跨仓库 Token 配置、同步工作流设计和常见注意事项。"
category: developer
tags: [GitHub, GitHub Actions, CI/CD, 公开镜像, 自动同步, 内容分发]
coverImage: ""
status: published
locale: zh
source: authored
---

# 私有仓库的文章如何自动同步到公开镜像站

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

**重点**：README 里要包含文章分类导航、热门文章推荐、主站链接，让访客有清晰的入口。

### 第二步：创建 GitHub Actions 同步工作流

在**私有仓库**（magictools）里创建 `.github/workflows/sync-articles-mirror.yml`：

```yaml
name: Sync Articles Mirror

on:
  push:
    branches: [main]
    paths:
      - 'articles/**'    # 只有 articles/ 变动才触发

jobs:
  sync:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Clone mirror repo
        run: |
          git clone --depth 1 \
            https://x-access-token:${{ secrets.ARTICLES_MIRROR_TOKEN }}@github.com/coocon/magictools-articles.git \
            /tmp/mirror

      - name: Copy articles to mirror
        run: |
          rm -rf /tmp/mirror/*
          cp -r articles/* /tmp/mirror/

      - name: Push to mirror
        run: |
          cd /tmp/mirror
          git config user.email "github-actions[bot]@users.noreply.github.com"
          git config user.name "GitHub Actions"
          git add -A
          if git diff --staged --quiet; then
            echo "No changes"
          else
            git commit -m "sync: from magictools@${GITHUB_SHA:0:7}"
            git push origin main
          fi
```

**关键设计点**：

1. **`paths: articles/**`** — 只在文章目录变动时触发，改代码不触发，节省 CI 时间
2. **`--depth 1`** — 浅克隆镜像仓库，不需要完整历史
3. **`rm -rf /tmp/mirror/*`** — 全量替换，保证两边完全一致（删文章也能同步）
4. **`git diff --staged --quiet`** — 如果没变动就跳过（避免空 commit）

### 第三步：配置跨仓库 Token

同步工作流需要往公开仓库推送代码，这需要跨仓库写权限。

1. 打开 https://github.com/settings/tokens  
2. 点 **Generate new token (classic)**  
3. 勾选 `repo` 权限（包含对仓库的读写）  
4. 复制生成的 `ghp_xxx...` token  
5. 在私有仓库的 Settings → Secrets → Actions 里新增：  
   - **Name**: `ARTICLES_MIRROR_TOKEN`  
   - **Value**: 刚才复制的 token

**Token 安全提醒**：这个 Token 有完整的 repo 权限，不要泄露。如果担心风险，可以创建更细粒度的 fine-grained token，只授权到 `magictools-articles` 这一个仓库。

### 第四步：验证

push 一篇文章到私有仓库的 `articles/` 目录，去 Actions 看工作流是否自动触发，再到公开仓库确认文章已同步。

---

## 注意事项

### 1. 清理敏感文件

私有仓库的 `articles/` 目录里如果有草稿（`_drafts/`）、模板文件等不适合公开的内容，要在同步前排除。

**方案**：在工作流里加 `.gitignore` 过滤：

```yaml
- name: Copy articles to mirror
  run: |
    rm -rf /tmp/mirror/*
    cp -r articles/* /tmp/mirror/
    # 排除草稿和模板
    rm -rf /tmp/mirror/_drafts/ /tmp/mirror/**/_template* 2>/dev/null || true
```

或者在镜像仓库的 `.gitignore` 里声明：

```
_drafts/
*_template*
```

### 2. README 保持区别

两个仓库的 README 应该是不同的：
- 私有仓库 README：面向开发者，讲技术栈、部署方式
- 公开仓库 README：面向读者，讲文章分类、热门推荐

**方案**：在同步工作流里排除 README：

```yaml
- name: Copy articles to mirror
  run: |
    rm -rf /tmp/mirror/*
    cp -r articles/* /tmp/mirror/
    rm -f /tmp/mirror/README.md  # 保留镜像仓库自己的 README
```

### 3. Token 过期

GitHub 的 classic token 默认不过期，但 fine-grained token 有有效期。建议设一个日历提醒，定期检查 token 是否还有效。

### 4. 双向同步问题

这个方案是**单向同步**（私有 → 公开）。如果有人在公开仓库提 PR 改文章，不会自动合并回去。

如果文章有翻译版本（比如中英文双语），确保同步时 `translationSlug` 等元数据保持一致。这对 markdown frontmatter 来说不是问题，但如果元数据存在数据库里，需要额外处理。

### 5. 首次同步时的大批量文件

如果有 100+ 篇文章首次同步，一次 commit 166 个文件是正常的。后续增量同步通常只有 1-2 个文件变动。

### 6. Actions 用量

这个工作流非常轻量——clone 浅层仓库、复制文件、commit 和 push。每次运行不到 30 秒，远在 GitHub Actions 免费额度（2000 分钟/月）之内。

### 7. 安全边界

这不是安全问题，而是设计选择：**公开仓库里的内容就是可被任何人访问的**。确保你放出去的内容不包含：
- 内部链接（指向未公开的 API）
- 硬编码的密钥或 Token
- 未发布的草稿
- 个人隐私信息

---

## 效果

设置完成后，每次在私有仓库写新文章并 push：

1. GitHub Actions 自动检测到 `articles/` 变动
2. 30 秒内同步到公开镜像仓库
3. 无需任何手动操作

访客可以直接通过 GitHub URL 阅读文章：

```
https://github.com/coocon/magictools-articles/blob/main/ai-tutorials/deepseek-founder-4h-interview.md
```

而且 GitHub 原生渲染 Markdown，阅读体验不输博客。

---

## 总结

这套方案的核心是两个简单原则：

1. **私有仓库 = 完整项目（代码 + 文章）**，不改变开发流程
2. **公开仓库 = 内容镜像（只有文章）**，自动从私有仓库同步

零额外成本（GitHub Actions 免费），零维护负担（全自动），但多了一个公开的内容分发渠道——可以直接分享到 Reddit、放在 YouTube 简介里、被 Google 索引。

如果你的项目也是「代码私有 + 内容想公开」的模式，这个方案可以直接复制使用。

---

*本文同步发布于 [magictools-articles 公开镜像站](https://github.com/coocon/magictools-articles)*
