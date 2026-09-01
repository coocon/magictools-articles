# PM2 cron_restart 是「先杀再起」：调度进程自杀断更两天的复盘

> 📍 本文首发于 [MagicTools 码农早餐](https://tools.cooconsbit.com/zh/articles/pm2-cron-restart-dispatcher-pitfall?utm_source=github&utm_medium=referral)。镜像仓库仅收录预览，**[点此阅读全文 →](https://tools.cooconsbit.com/zh/articles/pm2-cron-restart-dispatcher-pitfall?utm_source=github&utm_medium=referral)**

## 现象

一条每天早晨定时发布的内容管线（生成 → 发公众号 → 发邮件），08-28、08-29 连续两天全部没发。两天的日志特征一模一样：

- 生成步骤（intel-generate）的输出停在 **07:04:xx**，没有任何报错，就是没有下文了
- **07:05** 整，日志里出现两条 dispatcher 进程的启动头行
- 之后一整天，调度器每次唤醒都打同一句：「今天已有记录（running），跳过」
- 07:40 下游的公众号草稿步骤报 `guard:no-today-queue`——没稿可发

没有 crash 日志，没有 OOM，进程列表里 dispatcher 活得好好的。任务就像凭空蒸发了。

## 根因

这条管线的调度器 pipeline-dispatcher 当时是一个**一次性 PM2 进程**：跑一轮判定、该执行的步骤执行完就退出，靠 PM2 的 `cron_restart: '*/5 * * * *'` 每 5 分钟拉起来一次。

坑就在这里：**`cron_restart` 到点时如果发现实例还在运行，不是「等它跑完」也不是「跳过这轮」，而是先杀再起**——SIGINT 掐掉在跑的进程（连同它 spawn 的子进程一起），然后启动新实例。

平时无感，因为 generate 一轮只要 42~92 秒，永远赶在下一个 5 分钟边界前退出。出事那两天，候选池 24 小时上限打满后老条目在 07:00 前后滚出窗口、配额突然释放，现场补写量暴增（一次 LLM 聚类 1 分 43 秒 + 6 条简报校对），generate 被拖过了 07:05——于是：

1. 07:05 整，PM2 按 cron 表把 dispatcher 连同正在写稿的 generate 子进程一起杀掉
2. 数据库里的执行记录 PipelineRun 停在 `running` 状态，没人给它收尸
3. 幂等键（step + 当天日期）看到「今天已有记录」，**全天不再重试**
4. 下游每一步都等不到上游产物，整条链路静默瘫痪

一句话：**给「每 N 分钟唤醒」的调度进程挂 cron_restart，等于给它所有子任务设了 N 分钟硬超时**——而且这个超时不报错、不告警，杀完还留下一条僵尸 running 记录把重试通道也堵死。

...

---

**[👉 继续阅读全文：PM2 cron_restart 是「先杀再起」：调度进程自杀断更两天的复盘](https://tools.cooconsbit.com/zh/articles/pm2-cron-restart-dispatcher-pitfall?utm_source=github&utm_medium=referral)**

更多文章：[tools.cooconsbit.com/articles](https://tools.cooconsbit.com/zh/articles?utm_source=github&utm_medium=referral)
