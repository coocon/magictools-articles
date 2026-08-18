---
title: "DuckDB v2.0 预览：嵌入式数据库长出了 Server 模式"
slug: duckdb-v2-preview
summary: "DuckDB 官方放出 v2.0 预览：CONNECT 语句让 in-process 引擎第一次有了客户端/服务器模式，VARIANT 类型转正，SQL 解析器整个换掉。网上疯传的「40 倍提速」是真的，但场景是递归 CTE，不是聚合查询——这篇把每个数字对回原文。"
category: developer
tags: [DuckDB, 数据库, SQL, OLAP, 数据分析, 开源]
coverImage: ""
status: published
locale: zh
source: authored
translationSlug: duckdb-v2-preview-en
---

# DuckDB v2.0 预览：嵌入式数据库长出了 Server 模式

2026 年 8 月 17 日，DuckDB 两位创始人 Mark Raasveldt 和 Hannes Mühleisen 发了一篇《A Preview of DuckDB v2.0》。正式版要等到今年秋天，但预览构建已经能跑今天说的大部分功能。

自 3 月的 v1.5 以来，这个项目积累了超过 10,000 个 commit。v2.0 的代号叫 "Cyanoptera"（桂红鸭），而它做的事情，比一个大版本号更激进：**一个以「in-process、零依赖」立身的嵌入式数据库，正式长出了 server 模式。**

先把一件事说清楚。这条新闻在中文圈传播时普遍带着两个数字：「聚合查询提速 40 倍」和「压缩率提升 20%」。我们把官方博客逐段核对了一遍——40 倍是真的，但场景不是聚合；20% 这个数字，**原文从头到尾没有出现过**。下面按事实重新过一遍。

---

## 40 倍是真的，但主角是递归 CTE

官方博客全文只给了三处性能倍数，最大的那个来自**递归 CTE**（recursive CTE）：一张 100 万条边的图上跑单源可达性查询，v1.5.4 用时 4.90 秒，v2.0 预览版 0.12 秒——原文自己都加了感叹号："about 40× faster (!)"。

这背后是递归查询执行器的重写，对做图分析、层级遍历、BOM 展开这类工作负载是实打实的收益。但如果你的管道里全是普通的 GROUP BY 和窗口函数，请不要按 40 倍做预算。

聚合方向确实有改进，只是官方一个倍数都没给：partial aggregate 可以下推到 join 之下、冗余聚合会被复用、聚合超内存时可以落盘（spill to disk）。另外两处有数字的提升是：Windows CLI 多线程结果物化约 2.2 倍；自研时区/排序规则替换 ICU 之后，2500 万行的时区转换从 0.24 秒到 0.11 秒（2.2 倍）。

**怎么读 vendor benchmark 的通用姿势，这次又验证了一遍：找到数字对应的原始场景，再决定它跟你有没有关系。** 拿你线上最慢的三个查询在预览版上实测，比转发任何标题都可靠。

## 真正的头条：CONNECT 语句和 Quack 协议

SQLite 一系的产品有一条几十年的教条：嵌入式数据库不做网络。v2.0 把这条教条拆了。

新的 `CONNECT` / `DISCONNECT` 语句配合转正的 `quack` 扩展（DuckDB 原生网络协议），让你可以从一个 DuckDB 实例连到远端的另一个 DuckDB——甚至 `CONNECT 'postgres://...'` 直连 PostgreSQL，配合新的 remote pushdown 优化器把 SQL 推到远端执行。

Hacker News 上有条评论（531 分主帖下）说出了很多人的观感：过去一年 DuckDB 的演进，像是从 in-process 分析引擎滑向「云数仓的地基」。另一位用户举的场景更接地气：他们团队一直把多 GiB 的 `.duckdb` 文件当运行时产物传来传去，client/server 模式让这种「单文件当库用」的部署终于可以像正经数据库一样集中管理。

官方的说法是，DuckDB 从第一天就有完整的 MVCC 和事务隔离，只是单用户场景用不上——现在这些基础设施终于对外营业了。

## VARIANT 转正："imagine if JSON were fast"

半结构化数据类型 VARIANT 在 v2.0 成为一等公民：shredded 存储可以直读执行、extraction 谓词能下推、Parquet 的 shredded VARIANT 读写都齐了，还配了一族 `variant_*` 函数。官方还预告 v2.0 之后要让普通 JSON 类型直接坐上 VARIANT 的底座——也就是说现有 JSON 查询会免费变快。

对日志、埋点、API 响应这类「schema 天天变」的数据，这是最值得试的新特性。

## 其他值得记的

- **触发器完整交付**：BEFORE/AFTER、FOR EACH ROW/STATEMENT、transition tables 都有了。
- **SQL 解析器整个换掉**：弃用 PostgreSQL 派生解析器，改自研 PEG 解析器，扩展可以注入自己的语法；首个方言兼容模式是 `SET dialect_compatibility_mode = 'spark'`——意图很明显，冲着 Spark SQL 迁移市场去的。
- **异步 I/O 贯穿引擎**：Parquet、CSV、自有格式全部异步化，新增 MMAP 和 DIRECT_IO 模式，对象存储读取显著提速。
- **去 ICU**：时区/日历/排序规则自己实现，IANA 时区数据压到约 45 kB。HN 上有人对着 "We reimplemented ICU" 发了尖叫表情——这是出名的坑区，勇气可嘉，风险自负。
- **稳定 C API**：扩展「写一次永远能用」，API 由 YAML 声明式规范生成，还支持自建签名扩展仓库。

## 新存储格式：该说的和不该传的

存储格式 v2.0.0 的真实卖点是：ART 索引改为 buffer-managed（不再常驻内存，带大索引的表可以秒开）、列元数据懒加载、字符串压缩 DICT_FSST 默认启用、删除记录紧凑存储。官方总结是「打开更快、内存占用低得多」——**没有任何「压缩率提升 20%」的表述**，这个数字请不要再传了。

兼容性方面：DuckDB 自 v0.10 起保证新版本永远能读旧文件，所以「旧文件打不开」不成立；反过来，v2.0 新格式写出的文件旧版本可能读不了，跨版本协作的团队可以用 `storage_compatibility_version` 钉住旧格式。官方也明说 v2.0 有一小批破坏性变更（新默认存储格式、lambda 语法迁移完成），细节留给正式发布公告。

## 没给的东西，和该做的事

HN 上最尖锐的一条批评是：DuckDB 至今没有**增量物化视图**——评论者称之为「ClickHouse 最好的功能」，并指出实现所需的零件（聚合状态导出/finalize）DuckDB 其实都有，看起来像是刻意回避。v2.0 的功能列表确实也没有它。

给正在用 DuckDB 的团队三句话：

1. 预览版可以装，生产别急——正式版秋天到，破坏性变更清单还没出全。
2. 拿自己的慢查询实测，尤其是有递归 CTE 和大 VARIANT/JSON 负载的，收益可能远超平均。
3. 如果你在等它替代 ClickHouse，先确认你不依赖增量物化视图。

一个 2019 年才开源的项目，五年冲到 v2.0，半年 10,000 个 commit（HN 上已经有人怀疑有多少是 AI 写的）。不管答案是什么，这个迭代速度本身，就是嵌入式分析这个赛道最吓人的竞争力。

---

*资料来源：*
*DuckDB 官方博客：[A Preview of DuckDB v2.0](https://duckdb.org/2026/08/17/duckdb-20-highlights)*
*DuckDB 官方博客：[Asynchronous I/O in DuckDB](https://duckdb.org/2026/07/31/asynchronous-io)*
*Hacker News 讨论（531 分）：[A Preview of DuckDB v2.0](https://news.ycombinator.com/item?id=49330781)*
*DuckDB 存储兼容性文档：[Storage Versions and Format](https://duckdb.org/docs/stable/internals/storage)*
