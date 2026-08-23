# Docker 镜像从 arm64+amd64 砍到单平台：一次 CI 磁盘爆炸的踩坑复盘

> 📍 本文首发于 [MagicTools 码农早餐](https://tools.cooconsbit.com/zh/articles/docker-arm64-build-optimization?utm_source=github&utm_medium=referral)。镜像仓库仅收录预览，**[点此阅读全文 →](https://tools.cooconsbit.com/zh/articles/docker-arm64-build-optimization?utm_source=github&utm_medium=referral)**

## 背景：CI 又挂了

2026 年 7 月 22 日晚，magictools 项目的 CI 连续两次构建失败。日志显示：

```
No space left on device
```

失败发生在 `Build and push Docker image` 阶段——多架构 Docker 构建把 GitHub Actions hosted runner 的 ~14GB 可用磁盘吃满了。

当时的第一反应是：加一个清理磁盘的步骤。于是 push 了这行代码：

```yaml
- name: Free disk space
  run: |
    sudo rm -rf /usr/share/dotnet /usr/local/lib/android /opt/ghc \
      /opt/hostedtoolcache/CodeQL /usr/local/share/boost \
      /usr/local/.ghcup /usr/share/swift
    sudo docker image prune -af
```

能释放 ~25GB，暂时把问题压住了。但这是治标不治本——为什么一个 Next.js 项目的 Docker 镜像会吃这么多磁盘？

## 排查：arm64 镜像到底是谁在用？

看 CI 配置：

```yaml
platforms: linux/amd64,linux/arm64
```

两台服务器，两套架构。看到 `arm64` 时我愣了一下——我们有 ARM 服务器吗？

查 deploy.yml 的部署目标：

```yaml
matrix:
  include:
    - host: 182.92.161.253    # 阿里云 ECS，x86_64
    - host: 179.253.232.64    # DMIT 洛杉矶，x86_64
```

**两台全是 amd64**。arm64 镜像从来没有部署过。

那 arm64 是怎么来的？翻 git 历史：

```bash
$ git log --oneline -- .github/workflows/build.yml
82e61fe chore: Docker 镜像瘦身 P1-P4 — 去 arm64
b22b97a fix: CI 构建前清理 runner 磁盘
...
c9409f6 fix: 平台兼容              # ← 这一行
3247bb4 fix: 构建和部署分开          # ← 最初的 build.yml
```

`3247bb4` 是最初的 build.yml（2026-02-12），**只有 amd64**。当天下午，`c9409f6` 加上了 `arm64 + QEMU`，commit message 只有四个字：

```
fix: 平台兼容
```

没有 body，没有说明原因，没有链接 issue。就是一个"我觉得应该这样"的改动。

我推测当时的思路是：

1. **"多平台总比单平台好"** — Docker 官方文档里确实推荐多架构构建，看起来更"专业"
2. **"万一以后用 Apple Silicon Mac 跑呢"** — 开发机是 M 系列芯片，本地测试用 arm64 方便
3. **"反正 CI 不要钱"** — GitHub Actions 免费额度够，多跑一个架构没什么感觉

...

---

**[👉 继续阅读全文：Docker 镜像从 arm64+amd64 砍到单平台：一次 CI 磁盘爆炸的踩坑复盘](https://tools.cooconsbit.com/zh/articles/docker-arm64-build-optimization?utm_source=github&utm_medium=referral)**

更多文章：[tools.cooconsbit.com/articles](https://tools.cooconsbit.com/zh/articles?utm_source=github&utm_medium=referral)
