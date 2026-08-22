---
title: "跨境网站页面从 2.7s 降到 0.8s：MySQL binlog 免费搭只读副本 + Prisma 读写分离全记录"
slug: mysql-cross-region-read-replica-prisma
category: developer
tags: [MySQL, 只读副本, binlog, GTID, Prisma, 读写分离, 阿里云RDS, 跨境延迟]
summary: "网站按地域 DNS 分流到国内和海外两台服务器，但数据库只有北京一个 RDS，海外机每条查询跨太平洋往返，页面实测 2.7-4.2 秒。本文完整记录用 MySQL 原生 binlog/GTID 复制（零成本）在海外机房搭只读副本、Prisma 应用层读写分离的全过程：包括 5 个带报错原文的真实踩坑（时区表 1298、RDS 心跳让 GTID 永不静止、DOCKER-USER 链防火墙等），以及降级演练拿到的 3.3 倍提速 A/B 数据。"
status: published
locale: zh
source: authored
---

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

1. **登录会话**：NextAuth 的 database session 是"写入 session 后下一个请求立刻回读"，副本哪怕只有半秒延迟，用户登录就会闪断
2. **写后立读**：`create` 之后马上 `findMany` 同一张表的接口，复制延迟会让刚写的记录"消失"

反过来，「update 的返回值当读结果」这种模式天然安全——update 本身走主库，返回的就是主库数据。

## 只想让一台机器启用：主机白名单门禁

两台服务器共享同一份 `.env` 文件（同一个 CI Secret 渲染），不能靠"发不同配置"区分。做法是三段式：

1. 部署时 `docker run -e DEPLOY_HOST=<本机IP>`，容器知道自己在哪台机器
2. 入口脚本按白名单 + TCP 探测决定变量去留（**fail-safe 方向永远是降级直连主库**）：

```sh
if [ -n "$DATABASE_REPLICA_URL" ]; then
  if host_in_whitelist "$DATABASE_REPLICA_HOST"; then
    node -e "TCP 探测副本 3 秒" || unset DATABASE_REPLICA_URL   # 副本挂了→回主库
  else
    unset DATABASE_REPLICA_URL                                   # 国内机→行为零变化
  fi
fi
```

3. 应用代码只看变量存不存在

这个设计同时就是**降级开关**：副本宕机时重启 app 容器即自动回退主库；彻底摘除只需删 Secret 里两行。

## 五个真实踩坑（附报错原文）

### 坑 1：`lower_case_table_names` 只能在初始化时设置

阿里云 RDS 默认 `lower_case_table_names=1`，Linux 下 MySQL 默认 0。ORM 建的表名带大写（`Article`、`User`），两边不一致复制直接断。**这个参数只在初始化 datadir 时生效**，副本起容器前就必须写进 my.cnf，事后改 = 删数据目录重来。

### 坑 2：mysqldump 要 RELOAD 权限，而 RDS 心跳让 GTID 永不静止

初始全量同步的标准姿势 `mysqldump --set-gtid-purged=ON` 直接报错：

```
mysqldump: Couldn't execute 'FLUSH TABLES': Access denied;
you need (at least one of) the RELOAD or FLUSH_TABLES privilege(s)
```

RDS 普通账号没有也给不了 RELOAD。退而求"dump 前后各取一次 `gtid_executed`，一致就采信"——重试 8 次全失败：**阿里云 RDS 内部心跳表（`mysql.ha_health_check`）秒级写 GTID，静止窗口根本不存在**。

最终解法：`--set-gtid-purged=OFF` + 手工 `SET GLOBAL gtid_purged='<dump前的GTID>'`，配合**响亮失败验证**——dump 前的 GTID 与快照点间隙只有 1-2 秒，若业务写恰好落在里面，重放时必触发 1062/1032 报错（响亮、可检测，绝不静默错数据），报错就重建重试。心跳事务则被下一条过滤规则挡掉，重放无害。

### 坑 3：RDS binlog 携带命名时区，官方镜像时区表是空的

复制刚跑起来就断：

```
Error 1298: Unknown or incorrect time zone: 'Asia/Shanghai'
```

RDS 的会话时区是命名时区并写进 binlog 事务头；mysql 官方 Docker 镜像的 `mysql.time_zone*` 表默认全空。my.cnf 写 `+08:00` 只能对齐服务器默认值，**防不住 binlog 里携带的命名时区**。解法一行：

```bash
docker exec mysql-replica sh -c "mysql_tzinfo_to_sql /usr/share/zoneinfo | mysql -uroot mysql"
```

灌完时区表无需重建，`START REPLICA` 原地恢复（失败的事务是原子回滚的）。

顺带一条：副本要配 `replicate-wild-do-table=业务库.%` 过滤，否则 RDS 内部心跳表的 ROW 事件在副本找不到对应表，同样会打断复制。

### 坑 4：Docker 发布的端口不走 INPUT 链

限制 3306 只允许 app 机访问，`iptables -A INPUT --dport 3306` 写完毫无作用——Docker 发布端口的流量走 PREROUTING DNAT → FORWARD，根本不进 INPUT。正确姿势是 DOCKER-USER 链 + 匹配 DNAT 前的原始端口：

```bash
iptables -I DOCKER-USER -i eth0 -p tcp -m conntrack \
  --ctorigdstport 3306 --ctdir ORIGINAL ! -s <app机IP> -j DROP
```

### 坑 5：`MYSQL_ROOT_HOST=localhost` 下 root 是空密码

为了不把 root 暴露到公网设了 `MYSQL_ROOT_HOST=localhost`，结果初始化完 `mysql.user` 里 `root@localhost` 的 `authentication_string` 是**空的**（无密码 socket 直入）。暴露面虽然只有容器内，但仍然要手工补一刀 `ALTER USER` 并回查字段非空。

另外两道保险值得写进任何副本的 my.cnf：`super_read_only=ON`（应用层路由 bug 会直接报错而不是把数据写分叉）和 `performance_schema=OFF`（2G 小机省几百 MB 内存）。

## 效果：3.3 倍，而且拿到了干净的 A/B

上线后做降级演练时顺手解决了"没有改造前基线"的问题——把副本容器停掉、app 自动回退直连主库，这个状态就是改造前：

| 状态 | 同一文章页耗时（海外机本机实测） |
|------|--------------------------------|
| 直连北京主库（=改造前） | 2.7 - 4.2 s |
| 读本地副本（=改造后） | **0.81 - 0.95 s** |

约 **3.3 倍**。剩下的 0.8 秒里还包含一次跨洋写（页面浏览计数）和渲染本身，读路径已经不再是瓶颈。

演练同时验证了故障路径：停副本 → app 重启自动降级（站点全程 200 无中断）→ 副本恢复后复制自动续传、延迟归零。日常由 cron 每 5 分钟检查复制线程和延迟，异常发邮件告警。

## 这套方案适合谁

**适合**：读写比悬殊的内容站/工具站；数据库不大（我们只有 30MB，断链重建是分钟级）；已经做了多地域部署、只差数据这一层。

**不适合**：写多读少、或对"写后立读"有强一致要求的业务（要么忍受到处标注主库读，要么直接上分布式数据库）；库大到全量重建很痛的场景要先算好 binlog 保留窗口。

## 常见问题 FAQ

### 副本挂了网站会挂吗？

不会。入口脚本在启动时做 TCP 探测，探测失败自动降级为直连主库；运行中副本宕机则重启一次 app 容器即可回退。fail-safe 的方向永远是"回主库"，最坏情况是海外用户变慢，不是不可用。

### 复制断了怎么办？

先看报错：时区/表结构类问题修复后 `START REPLICA` 原地续传；无法修复或断链超过 binlog 保留期（我们是 30 天）就删数据目录全量重建——库小的话整个过程只要几分钟。

### 为什么不用 Redis 缓存解决？

缓存解决的是"热数据重复读"，这里的问题是"所有读都要跨洋"。缓存需要失效逻辑侵入业务代码，且冷数据首查仍然慢；副本方案对业务代码几乎透明，一次搭建全量加速。

### Prisma 的读写分离扩展稳定吗？

`@prisma/extension-read-replicas` 是 Prisma 官方扩展，但它依赖 Prisma Client 的内部字段，peer 版本声明比较保守。我们把扩展和 prisma 双双锁死精确版本，并留了一个 7 项断言的路由验证脚本，升级 prisma 前必须重跑。

## 参考链接

- [MySQL 8.0 Reference: Replication with GTIDs](https://dev.mysql.com/doc/refman/8.0/en/replication-gtids.html)
- [Prisma 官方读写分离扩展 @prisma/extension-read-replicas](https://github.com/prisma/extension-read-replicas)
- [MySQL 官方 Docker 镜像文档（时区表、MYSQL_ROOT_HOST）](https://hub.docker.com/_/mysql)
- [Docker 官方文档：Packet filtering and firewalls（DOCKER-USER 链）](https://docs.docker.com/engine/network/packet-filtering-firewalls/)
- [阿里云 RDS MySQL 帮助文档](https://help.aliyun.com/zh/rds/apsaradb-rds-for-mysql/)
