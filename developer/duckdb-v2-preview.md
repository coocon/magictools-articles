# DuckDB v2.0 预览：嵌入式数据库长出了 Server 模式

> 📍 本文首发于 [MagicTools 码农早餐](https://tools.cooconsbit.com/zh/articles/duckdb-v2-preview?utm_source=github&utm_medium=referral)。镜像仓库仅收录预览，**[点此阅读全文 →](https://tools.cooconsbit.com/zh/articles/duckdb-v2-preview?utm_source=github&utm_medium=referral)**

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

...

---

**[👉 继续阅读全文：DuckDB v2.0 预览：嵌入式数据库长出了 Server 模式](https://tools.cooconsbit.com/zh/articles/duckdb-v2-preview?utm_source=github&utm_medium=referral)**

更多文章：[tools.cooconsbit.com/articles](https://tools.cooconsbit.com/zh/articles?utm_source=github&utm_medium=referral)
