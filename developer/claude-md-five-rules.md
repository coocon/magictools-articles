# CLAUDE.md 不是 README：给 Claude Code 写规则的 5 条硬规矩

> 📍 本文首发于 [MagicTools 码农早餐](https://tools.cooconsbit.com/zh/articles/claude-md-five-rules?utm_source=github&utm_medium=referral)。镜像仓库仅收录预览，**[点此阅读全文 →](https://tools.cooconsbit.com/zh/articles/claude-md-five-rules?utm_source=github&utm_medium=referral)**

Claude Code 启动时会读项目根目录的 `CLAUDE.md`，把内容塞进每一轮对话的上下文。很多人第一反应是：那我把 README 贴进去，再把架构文档贴进去，越全越好。

然后发现它该守的规矩不守，不该动的文件照动，跑测试时总用错命令。

问题不在 Claude Code，在于 **CLAUDE.md 不是给人看的文档，是给模型的常驻指令**。文档追求完整，指令追求命中。按写文档的思路写它，字越多命中率越低。下面五条规矩，是我在自己的项目里改了几十版之后留下来的。

## 规矩一：只写代码里推不出来的东西

模型能读代码。目录结构、函数签名、依赖列表，它自己 `ls` 和 `grep` 一下就知道，你写进 CLAUDE.md 只是浪费上下文，还会在代码变化后变成过期信息误导它。

该写的是**从代码里推不出来的事实**：

- 构建、测试、类型检查用什么命令。`npm test` 还是 `npm run test:unit`，跑全量还是只跑单个文件，模型猜不到。
- 环境的怪癖。比如「访问 GitHub 必须走本机代理 127.0.0.1:7897」「数据库迁移只能在容器里跑」。
- 团队约定。ES 模块不用 CommonJS、接口统一返回 `{ code, msg, data }`、分页排序由后端负责前端不翻转。
- 红线。不动 `.env`、不 `git push`、不删文件。

一个判断方法：这句话删掉之后，模型靠读代码能不能得出同样的结论？能，就删。

## 规矩二：越短越管用

CLAUDE.md 的每一个字都在每一轮对话里重复出现。它不只是花钱，更重要的是**稀释注意力**。三百行的 CLAUDE.md 里藏一条「禁止修改 schema」，和三十行里放同一条，模型的遵守率差得很远。

Anthropic 自己的建议是：保持简洁、人类可读、像调提示词一样反复迭代。他们还提了一个实用技巧，对真正重要的规则加 `IMPORTANT` 或 `YOU MUST` 这类强调，能明显提高遵守率。反过来说，如果你每条都加强调，等于没加。

...

---

**[👉 继续阅读全文：CLAUDE.md 不是 README：给 Claude Code 写规则的 5 条硬规矩](https://tools.cooconsbit.com/zh/articles/claude-md-five-rules?utm_source=github&utm_medium=referral)**

更多文章：[tools.cooconsbit.com/articles](https://tools.cooconsbit.com/zh/articles?utm_source=github&utm_medium=referral)
