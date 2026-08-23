# 跨境网站页面从 2.7s 降到 0.8s：MySQL binlog 免费搭只读副本 + Prisma 读写分离全记录

> 📍 本文首发于 [MagicTools 码农早餐](https://tools.cooconsbit.com/zh/articles/mysql-cross-region-read-replica-prisma?utm_source=github&utm_medium=referral)。镜像仓库仅收录预览，**[点此阅读全文 →](https://tools.cooconsbit.com/zh/articles/mysql-cross-region-read-replica-prisma?utm_source=github&utm_medium=referral)**

## 业务问题：同一个网站，两种速度

我们的站点按访问者地域做 DNS 分流：大陆用户解析到阿里云（北京），海外用户解析到洛杉矶的 VPS。应用是无状态的 Next.js，两边跑同一份镜像——但数据库只有一个：阿里云 RDS MySQL，在北京。

结果就是一个很典型的"分了流量、没分数据"的架构：

- 大陆用户：app 和数据库同地域，单条查询几毫秒，页面百毫秒级
- 海外用户：app 在洛杉矶，每条 SQL 都要跨太平洋往返一次，RTT 约 150-200ms

SSR 页面通常要串行跑多条查询（文章 + 相关推荐 + 标签 + 分类），延迟被逐条放大。**实测同一个文章页：国内机 0.1-0.5 秒，海外机 2.7-4.2 秒**。用户的直观感受就是"这网站在国外卡得没法用"。

## 选型：为什么是原生 binlog 复制

写流量很小（内容站，读写比悬殊），要解决的只是"读"的物理距离。备选方案对比：

| 方案 | 成本 | 问题 |
|------|------|------|
| 云厂商只读实例 | 按实例付费 | 只能建在同厂商地域内，出不了境，解决不了跨洋 |
| DTS / 数据同步服务 | 按量付费 | 为 30MB 的库付订阅费不划算 |
| 应用层缓存（Redis） | 新组件 | 缓存失效逻辑侵入业务，且首查仍跨洋 |
| **MySQL 原生 binlog/GTID 复制** | **0 元** | 需要自己踩运维的坑（本文价值所在） |

MySQL 复制是标准能力，RDS 作为源端不收任何费用。副本落在与海外 app **同机房**的一台 2C2G 小 VPS 上（Docker 跑 MySQL 8），读延迟从 150ms 变成本机房 1ms 级。

动手前先对源库做**前提核查**，五个必查项（在 RDS 上执行）：

```sql
SELECT @@version;          -- 副本版本必须 ≥ 源库（我们是 8.0.36）
SELECT @@gtid_mode;        -- 必须 ON，才能用 SOURCE_AUTO_POSITION
SELECT @@binlog_format;    -- 期望 ROW
SHOW VARIABLES LIKE 'binlog_expire_logs_seconds';  -- 断链重建窗口（我们是 30 天）
SHOW GRANTS FOR CURRENT_USER;  -- 有没有 REPLICATION SLAVE 权限
```

意外收获：阿里云 RDS 的高权限账号默认自带 `REPLICATION SLAVE`，不需要单独建复制账号。

## 架构：写回北京，读在本地

```
阿里云 RDS 北京（主库，唯一写入点）
    │  binlog/GTID 原生复制，SSL 加密，SOURCE_AUTO_POSITION=1
    ▼
洛杉矶 VPS：MySQL 8 只读副本（Docker，与海外 app 同机房）
    ▲  读 ~1ms
海外 app ──写───────────────────→ 仍直写北京主库
国内 app ──读写──→ 北京主库（完全不变）
```

应用层用 Prisma 官方扩展 [@prisma/extension-read-replicas](https://github.com/prisma/extension-read-replicas) 做路由，核心是**双导出**：

```ts
// src/lib/db.ts（简化）
export const prismaPrimary = new PrismaClient({ /* 主库 */ });

export const prisma = process.env.DATABASE_REPLICA_URL
  ? prismaPrimary.$extends(readReplicas({ url: process.env.DATABASE_REPLICA_URL }))
      as unknown as PrismaClient   // cast 保型，全站 200+ 调用点零改动
  : prismaPrimary;                 // 没配副本 = 同一实例，行为与改造前一字不差
```

扩展的路由规则：`find*/count/aggregate/groupBy` 走副本；**所有写、`$transaction`、`$queryRaw` 自动走主库**。绝大多数代码不用动，但有两类场景必须手工固定主库（import `prismaPrimary`）：

...

---

**[👉 继续阅读全文：跨境网站页面从 2.7s 降到 0.8s：MySQL binlog 免费搭只读副本 + Prisma 读写分离全记录](https://tools.cooconsbit.com/zh/articles/mysql-cross-region-read-replica-prisma?utm_source=github&utm_medium=referral)**

更多文章：[tools.cooconsbit.com/articles](https://tools.cooconsbit.com/zh/articles?utm_source=github&utm_medium=referral)
