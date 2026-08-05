---
title: "GitHub Actions 秒挂 connection reset：境外 runner 拉阿里云 ACR 的跨境陷阱"
slug: "github-actions-aliyun-acr-connection-reset"
category: developer
locale: zh
source: authored
tags:
  - github-actions
  - docker
  - aliyun
  - acr
  - ci-cd
  - troubleshooting
summary: "GitHub Actions 构建启动两秒就挂在 load metadata，报 failed to fetch oauth token: connection reset by peer，报错行还指向 Dockerfile 的 FROM。别改代码——这是境外 runner 到阿里云 ACR 鉴权服务的跨境链路偶发中断。本文给出判据、带自动重试的 workflow 写法和结构性根治思路。"
coverImage: ""
status: published
scheduledAt: ""
---

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

所以这是**网络层的偶发故障**，不是鉴权配置问题（鉴权错误会返回 401/403 的 HTTP 响应，而不是 TCP 连接被重置），更不是 Dockerfile 问题（报错指向 FROM 行只是因为失败发生在解析该行的元数据阶段）。

### 一条值得记住的诊断口诀

> **先看失败发生在第几秒。秒级失败 = 网络 / 鉴权链路；分钟级失败才可能是编译 / 依赖问题。**

2 秒就挂，连依赖都还没开始装，把时间花在检查 npm 包或代码上是南辕北辙。

## 四、修复：workflow 里加一次自动重试

跨境链路的偶发 RST 无法从我们这端消除，但它的特点是**重试大概率成功**。改造 build 步骤：

```yaml
- name: Build and push
  id: build
  continue-on-error: true        # 第一次失败不终止 job
  uses: docker/build-push-action@v6
  with:
    context: .
    platforms: linux/amd64,linux/arm64
    push: true
    tags: ${{ env.IMAGE_TAGS }}

- name: Wait before retry
  if: steps.build.outcome == 'failure'
  run: sleep 30                  # 给链路一点恢复时间

- name: Build and push (retry)
  if: steps.build.outcome == 'failure'
  uses: docker/build-push-action@v6
  with:                          # ⚠️ 参数必须与上面逐字一致
    context: .
    platforms: linux/amd64,linux/arm64
    push: true
    tags: ${{ env.IMAGE_TAGS }}
```

两个实操注意点：

- **GitHub Actions 的 YAML 不支持锚点**（`&anchor` / `*ref`），两个步骤的 `with` 块没法复用，只能手工保持逐字同步——以后改参数时两处都要改，最好在文件里留注释提醒。
- `continue-on-error: true` 会让第一步失败时步骤状态显示绿色勾（外层 job 不失败），要靠 `steps.build.outcome` 判断真实结果，别被界面颜色骗了。

上线这套重试后，同样的偶发 reset 都被第二次构建接住了，没有再出现整条流水线红掉的情况。

## 五、结构性根治（可选）

重试只是止血。想让"境外 runner 拉境内镜像"这个跨境依赖彻底消失，思路是**把基础镜像镜像一份到境外可稳定访问的 registry**：

1. 把自定义基础镜像同步推一份到 GHCR（`ghcr.io`）
2. Dockerfile 的 `FROM` 改走 GHCR——拉取侧不再跨境
3. 构建产物照旧推 ACR（推送侧的跨境写入实测比鉴权拉取稳定，且失败会被上面的重试兜住）

代价是多维护一份镜像同步。如果偶发频率不高，重试方案已经够用；等失败频率影响到发布节奏再上这套。

## 六、排查清单（遇到同类报错时按序过一遍）

1. 失败发生在**第几秒**？秒级 → 网络/鉴权方向，分钟级 → 编译/依赖方向
2. 报错是 **TCP 层**（connection reset / timeout）还是 **HTTP 层**（401/403/404）？前者查链路，后者才查凭证
3. 同样的配置**之前成功过吗**？成功过且无相关改动 → 偶发环境问题，先重试再说
4. 报错指向的代码行（比如 FROM）只是**失败时正在处理的位置**，不等于那一行有问题
