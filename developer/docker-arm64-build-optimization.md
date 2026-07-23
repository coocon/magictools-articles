---
title: "Docker 镜像从 arm64+amd64 砍到单平台：一次 CI 磁盘爆炸的踩坑复盘"
slug: docker-arm64-build-optimization
category: developer
locale: zh
tags: [Docker, CI/CD, GitHub Actions, arm64, 构建优化, 踩坑, DevOps]
summary: "GitHub Actions 构建 Docker 镜像频繁报 No space left on device，排查发现 arm64 镜像从来没用过、基础镜像用了完整的 Debian、CJK 字体装了全套 150MB。砍掉 arm64 后构建从 28 分钟降到 10 分钟，镜像瘦身 120MB+。这篇复盘整个排查和优化过程。"
status: published
---

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

但问题在于：**生产环境不需要的东西，CI 里也不应该有。**

## arm64 构建的真实代价

在 GHA hosted runner 上，arm64 构建是通过 **QEMU 模拟**完成的，不是原生执行。这意味着：

| 维度 | amd64 | arm64 (QEMU) | 影响 |
|------|-------|-------------|------|
| `npm ci` | ~2 分钟 | ~5 分钟 | 3 倍慢 |
| `next build` | ~3 分钟 | ~8 分钟 | 2.6 倍慢 |
| Docker 层缓存 | ~2GB | ~2GB | 磁盘翻倍 |
| 推送镜像体积 | ~250MB | ~250MB | ACR 存储翻倍 |

**arm64 构建单独吃掉的时间几乎等于 amd64 的 2 倍，磁盘占用直接翻倍。**

而且 GHA 的 `cache-to: type=gha,mode=max` 会把**所有构建层的中间缓存**都存下来。双平台 + max 缓存模式 = 磁盘爆炸的完美配方。

## 不止 arm64：顺便做了镜像瘦身

既然在动 CI 配置，索性把 Dockerfile 也审了一遍。

### 问题 1：构建阶段用了完整 Debian

```dockerfile
# 改前
FROM node:22.18.0-bookworm AS deps      # ~370MB
FROM node:22.18.0-bookworm AS builder    # ~370MB
```

`npm ci` 和 `next build` 只需要 Node.js 运行时，不需要 systemd、man page、文档。切到 slim：

```dockerfile
# 改后
FROM node:22.18.0-bookworm-slim AS deps      # ~240MB
FROM node:22.18.0-bookworm-slim AS builder    # ~240MB + openssl
```

slim 缺 openssl（Prisma engine 运行时依赖），builder 阶段补一个 `apt-get install openssl` 即可。deps 阶段只跑 `npm ci`，什么都不用补。

### 问题 2：CJK 字体装了全套

基础镜像里这行：

```dockerfile
RUN apt-get install -y fonts-noto-cjk
```

`fonts-noto-cjk` 是 Noto 中日韩**全套**字体（Sans + Serif + Mono，多字重），约 **150MB**。而项目里用字体的场景只有一个：公众号封面通过 sharp 渲染中文标题和摘要。

```dockerfile
# 改后
RUN apt-get install -y fonts-noto-sans-sc fonts-noto-serif-sc
```

只装简体中文的衬线和无衬线体，约 36MB，减了 114MB。

### 问题 3：QEMU 步骤多余

去掉 arm64 后，`Set up QEMU (multi-platform)` 这个 step 也不需要了。QEMU 安装本身不慢，但少一个 step 就是少一个可能出问题的点。

## 改了什么

总共 4 个文件：

| 文件 | 改动 |
|------|------|
| `.github/workflows/build.yml` | `platforms` 去掉 arm64，删 QEMU step |
| `Dockerfile` | deps/builder 切 bookworm-slim，builder 补 openssl |
| `docker/base/Dockerfile` | `fonts-noto-cjk` → `fonts-noto-sans-sc` + `fonts-noto-serif-sc` |
| `docker/base/README.md` | 同步文档 |

完整 diff 只有 26 行增、22 行删。

## 效果对比

改完 push，CI 自动触发：

| 指标 | 优化前 | 优化后 | 变化 |
|------|--------|--------|------|
| 构建时间 | ~28 分钟 | **~10 分钟** | **-64%** |
| 构建架构 | amd64 + arm64 | **仅 amd64** | 减半 |
| 基础镜像 | bookworm (~370MB) | **slim (~240MB)** | -130MB |
| 字体体积 | CJK 全套 (~150MB) | **仅 SC (~36MB)** | -114MB |
| 磁盘不足 | ❌ 偶发 | ✅ 根除 | — |
| 部署 | 2 台全成功 | 2 台全成功 | 不变 |

**构建 + 部署全过程不到 11 分钟**，而且再也不会 "No space left on device"。

## 经验总结

### 1. "多平台兼容"不是免费的午餐

arm64 听起来高级，但如果你的服务器全是 amd64，它就是一个纯粹的成本项：构建时间、磁盘空间、调试复杂度。**只为实际部署的目标构建镜像。**

### 2. 别在 CI 里预支"以后可能用"

"万一以后换 ARM 服务器呢" — 真到了那一天，加一行 `platforms` 只需要 30 秒。但在此之前，每次构建都在为不存在的场景买单。**YAGNI (You Ain't Gonna Need It) 在 CI 配置里同样适用。**

### 3. 磁盘告警先查根源，再打补丁

这次的第一反应是加 `Free disk space` 步骤，它确实有用（现在也保留着），但它只是把问题往后推。真正的解决办法是去掉不必要的构建产物。**治标可以应急，治本才能放心。**

### 4. Docker 基础镜像选 slim

`node:bookworm` vs `node:bookworm-slim` 差了 ~130MB，而构建阶段只需要 Node.js 运行时。如果你不确定需要什么，先选 slim，缺什么补什么（用 `apt-get` 装一个包远比拖一个完整镜像便宜）。

### 5. 字体按需安装

`fonts-noto-cjk` 装了日文、韩文、繁体中文全套——如果你的服务只有简体中文用户，这就跟给只做中餐的厨房配了全套日料刀具一样。**装你用得到的，而不是"包里的全部"。**

### 6. commit message 写清楚 why

`fix: 平台兼容` 这种 message，4 个月后回来看，完全不知道为什么加了 arm64。如果当时写的是 "ci: 添加 arm64 支持，本地 M 芯片开发调试用"，今天的排查只需要 1 分钟。

---

**一句话：CI 配置是代码，不是愿望清单。为今天的需求写，不为明天的可能性买单。**
