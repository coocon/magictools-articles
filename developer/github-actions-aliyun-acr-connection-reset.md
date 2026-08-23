# GitHub Actions 秒挂 connection reset：境外 runner 拉阿里云 ACR 的跨境陷阱

> 📍 本文首发于 [MagicTools 码农早餐](https://tools.cooconsbit.com/zh/articles/github-actions-aliyun-acr-connection-reset?utm_source=github&utm_medium=referral)。镜像仓库仅收录预览，**[点此阅读全文 →](https://tools.cooconsbit.com/zh/articles/github-actions-aliyun-acr-connection-reset?utm_source=github&utm_medium=referral)**

## 一、问题描述

一次很平常的 push 触发了 Docker 镜像构建，GitHub Actions 启动约 2 秒后直接失败：

```
ERROR: failed to authorize: failed to fetch oauth token:
Post "https://dockerauth.cn-hangzhou.aliyuncs.com/auth": read: connection reset by peer
```

日志里报错位置指向 Dockerfile 的这一行：

```dockerfile
FROM registry.cn-hangzhou.aliyuncs.com/<namespace>/node:22-pm2
```

第一反应很容易跑偏：是不是基础镜像 tag 写错了？是不是 ACR 密码过期了？是不是刚改的 Dockerfile 有问题？

都不是。**判据只有一条：同一份 Dockerfile、同一套凭证，前一次构建成功，后一次失败，中间没有任何相关改动。** 代码没变而结果变了，问题就不在代码。

## 二、环境

| 项目 | 详情 |
|------|------|
| CI | GitHub Actions，GitHub 托管 runner（境外机房） |
| 构建 | docker/build-push-action，multi-platform（amd64 + arm64） |
| 镜像仓库 | 阿里云 ACR（杭州 region） |
| 基础镜像 | 自定义 node:22-pm2，也存放在同一 ACR |

## 三、根因

链路拆开看：

1. GitHub 托管 runner 在境外
2. `FROM` 一个 ACR 私有镜像，buildx 在 "load metadata" 阶段要先向 `dockerauth.cn-hangzhou.aliyuncs.com` 拿 OAuth token
3. 这条**境外 → 阿里云杭州**的跨境 TCP 链路质量不稳定，偶发被中间设备直接 RST——表现就是 `connection reset by peer`

...

---

**[👉 继续阅读全文：GitHub Actions 秒挂 connection reset：境外 runner 拉阿里云 ACR 的跨境陷阱](https://tools.cooconsbit.com/zh/articles/github-actions-aliyun-acr-connection-reset?utm_source=github&utm_medium=referral)**

更多文章：[tools.cooconsbit.com/articles](https://tools.cooconsbit.com/zh/articles?utm_source=github&utm_medium=referral)
