---
title: "X 开源推荐算法：看得见，不等于看得懂"
slug: x-algorithm-open-source
summary: "8 月 13 日 xAI 把 For You 信息流的排序、过滤、打标签系统一次性推上 GitHub，代码量翻了十几倍，连 Phoenix 排序模型的训练代码和合成数据都给了。第二天他们又提交了一次——不是改逻辑，是给权重加注释，因为有人把 -234 读成了「一个举报抵 468 个赞」。"
category: ai-tutorials
tags: [推荐算法, X, Twitter, xAI, 开源, Rust, 排序模型, 算法透明度]
coverImage: ""
status: draft
locale: zh
source: authored
translationSlug: x-algorithm-open-source-en
---

# X 开源推荐算法：看得见，不等于看得懂

> "This is the kind of thing that I think people will be fairly shocked that we are releasing."
> —— X 产品 VP Keith Coleman，TechCrunch 专访，2026 年 8 月 13 日

`xai-org/x-algorithm` 这个仓库不新：2026 年 1 月 19 日建库，1 月 20 日推第一版。真正值得读的是 8 月 13 日和 14 日这两次提交——排序权重的具体数值、决定「能不能显示」的可见性过滤系统、Phoenix 模型的训练代码和合成数据生成，一次性全进来了。TechCrunch 引述 X 的说法是代码量涨了 10 到 15 倍。

截至本文写作（2026-08-15），这个仓库 29,347 stars、4,918 forks、54 个 open issue，主语言 Rust，Apache-2.0。作为对照，2023 年那个 Scala 写的 `twitter/the-algorithm` 至今 73,738 stars——热度更高，但那次给的东西少得多。

下面七条是我读完 README、`param.rs`、`brazil_2026_election_filter.rs`、`docs/BIDIRECTIONAL_BOOST_CHANGE.md` 和 `phoenix/QUICKSTART.md` 之后，觉得真正有信息量的地方。

---

## 1. 和 2023 年那次的实质区别：这回模型能跑起来

> "所有代码都可检视，其中一部分甚至设计成可以端到端运行——例如 Phoenix 打分模型的训练和推理。"
> —— README，*What's not in this repo?*

2023 年那次开源，Twitter 给了 heavy ranker 的工程代码，但没给训练流程，外面的人拿到手只能读、不能跑。这次 `phoenix/` 目录里是完整的：Cargo workspace、`pyproject.toml`、`QUICKSTART.md`、合成数据生成脚本，照着走一遍可以训一个小模型并起服务。TechCrunch 还提到，发布前 X 找了几位外部推荐系统研究者提前试跑，确认能独立训练并运行 Phoenix。

但 `QUICKSTART.md` 开头自己就把线划清楚了：

> "It is not a production-quality model or a production-scale setup. Production data, checkpoints, orchestration, and scale are not included."

而且门槛不低——Linux + NVIDIA GPU + CUDA 12 + Rust 工具链，教程里的示范训练只跑 6 步，文档还专门写了句「不要把六步当成模型质量的证据」。

**我的看法**：「可运行」和「可复现」是两件事，这次给的是前者。你能验证这套架构确实能学起来，但复现不了线上那个模型的行为——数据和 checkpoint 都不在。这依然是巨大进步：一个跑得起来的骨架，比一堆读不出所以然的类定义有用得多。只是别把 demo 当审计工具。

---

## 2. 它不预测「你会喜欢吗」，它预测「你会做什么」

> ```
> Final Score = Σ (weight_i × P(action_i))
> ```
> —— README，*Scoring and Ranking*

Phoenix 对每条候选帖预测的不是一个 relevance 分，而是二十多个动作各自的概率：点赞、回复、转推、引用、私信分享、复制链接分享、点开图片/视频/链接、点头像、停留时长、活跃秒数、关注作者，以及负向的不感兴趣、静音、拉黑、举报、没停留。然后用一组写在代码里的权重加权求和。

这个拆法的价值在于：**模型只负责估概率，价值判断被隔离到一组人类可读的常量里**。你想知道 X 觉得「引用转推」比「点赞」重要多少，不用去逆向 embedding，`home-mixer/params/param.rs` 里写着 `QuoteWeight = 5.0`、`FavoriteWeight = 0.5`。

顺带一个容易被忽略的设计（README 的 Key Design Decisions 第 2 条）：transformer 推理时候选帖之间不互相 attend，只 attend 用户上下文。所以一条帖子的分数跟同批次里有哪些帖子无关——分数稳定、可缓存、可单独复算。

**我的看法**：这是整套系统里最值得别的团队抄的一条。多目标预测 + 显式权重的组合，把「模型学到了什么」和「我们想要什么」在架构上切开了。调价值观不用重训模型，改一个 f64 就行——这既是可解释性的来源，也正是下面第 5 条要说的麻烦所在。

---

## 3. 排序和可见性是两套系统，这是「限流不禁言」的技术底层

> "Ranking decides the order. Visibility filtering decides whether a post can be shown at all. Different services, different inputs, different rules."
> —— README，*Key Design Decisions #4*

`visibility-filtering/` 对每一组（帖子，读者）给三选一的答案：ALLOW、INTERSTITIAL（挡一层，用户可点穿）、DROP。它跑在排序**之后**，输入是打标签路径产出的标签（`scarecrow/`、`botmaker/`、`agatha/`、`bdsm/`、`user-cred-v2/`）加上读者自己的拉黑/静音/关注关系、国家和设置。

最能说明问题的是这句：

> "Some rules drop a post only when it is a recommendation from an account the viewer does not follow — spam caught at high recall, for instance. The same post is allowed to a follower."

同一条帖子，对关注者放行，对陌生人丢弃。这就是多年来大家争论的 shadowban 在代码层面的样子——它不是一个封禁开关，是一套「推荐可见性」和「订阅可见性」分开判定的规则集，规则按顺序求值，第一条判 DROP 就结束。

**我的看法**：开源这套，比开源排序权重更有意义。排序是产品口味问题，可见性过滤是权力问题。设计本身很合理（高召回反垃圾只作用于陌生人推荐，误伤成本低），但正因为合理才更要能被外部审查——`visibility-filtering/rules/registry.rs` 里那份按求值顺序排列的规则表，是这次发布里我最想让监管机构去读的文件。

---

## 4. 468 倍那件事：开源代码 ≠ 公众能读懂

> "One common misinterpretation is that you can read these weight ratios as count equivalences, e.g. the incorrect statement that 'one report cancels 468 likes'."
> —— `home-mixer/params/param.rs`，8 月 14 日新增注释

先说数字是真的。`param.rs` 里 `FavoriteWeight = 0.5`、`ReportWeight = -234.0`，234 ÷ 0.5 = 468，这个比值就在代码里。

问题出在怎么读它。权重乘的是**预测概率**，不是**已发生的计数**。举报的基线概率比点赞低一千倍以上（注释里明说了），所以给它一个大权重只是为了让这个预测在最终分里还能起作用——不是「一个真实举报抵消 468 个真实点赞」。而且推荐是个性化的：坏人的举报主要影响与坏人相似的那批用户看到的排序，不是全局压制。

8 月 14 日的提交把这段解释同时加进了 `param.rs` 和 `ranking_scorer.rs`，README 里给的理由很有意思——"so that LLMs or people reading it are more likely to understand it correctly"。

这不是杞人忧天。8 月 14 日 explainx.ai 发的文章标题就是《X open-sources For You ranking weights: a Report costs 468 likes of reach》，摘要里写着「a Report costs 468 likes」。X 加注释和这篇文章基本是同一天的事。

**我的看法**：这条暴露了透明度的真实成本——代码公开只是第一步，之后你得持续跟误读赛跑，而且现在你的读者里有一半是 LLM。另外注释本身也是有立场的表述：X 说协同举报压不了 reach，这话对不对，得靠 `agatha/`、`bdsm/` 的实际行为验证，注释只是一面之词。

---

## 5. 你读到的权重是快照，而且是三个月一次的快照

> "On July 13, 2026, after seeing great initial results from the A/B test, we rolled out a boost value of 20 to many users... Then, on July 24, 2026... we set the bidirectional follow reply boost value to 15 instead of 20."
> —— `docs/BIDIRECTIONAL_BOOST_CHANGE.md`

这份文档是我认为整个仓库里被低估的一个文件。它完整复盘了一个参数的生命周期：7 月 10 日开 A/B，把 `BidirectionalFollowReplyWeightBoost` 随机分配成 5/10/15/20（多数用户是 0）；7 月 13 日效果好，放大到 20；7 月 24 日——理由写得很具体，世界杯期间用户反馈说看不到足够多的相关讨论，因为很多帖子来自没关注的账号——从 20 降到 15。仓库里就是一行 diff：`-20.0` / `+15.0`。

README 也承认，真实的可调值是从配置系统读的，代码里的默认值靠 cron 脚本定期刷成生产主值；实验只保证「跑到 10% 以上流量的」会体现在仓库里。

再看仓库的实际更新节奏。用 GitHub API 拉 commit 列表，从建库到现在总共 **6 次提交**：2026-01-20、2026-05-15（两次）、2026-08-13（两次）、2026-08-14。1 月宣布时的说法是每 4 周更新一次；实际是 1 月 → 5 月 → 8 月。

**我的看法**：所以「读代码就知道算法」要打折。参数每天在动，仓库三四个月同步一次，你读到的是被 cron 刷新过的快照，不是实时状态。`BIDIRECTIONAL_BOOST_CHANGE.md` 恰恰证明他们**有能力**把变更讲清楚——那真正该追问的就不是「开不开源」，而是「多久同步一次」。开源是承诺，节奏才是履约。

---

## 6. 藏起来的那部分，用「你自己查」来换

> "To reduce the risk of this, there are a limited set of files not currently published in the repository, e.g.: Grox prompts... Some botmaker rules"
> —— README，*What's not in this repo?*

留在外面的是两类：Grox（内容分类系统）里那些具体的 LLM prompt（j2 文件），以及部分 botmaker 规则。理由是防 gaming——知道 prompt 就知道怎么绕过垃圾内容识别。TechCrunch 引述的说法是「防止坏人拿这个绕开规则、往网络里灌垃圾」。

X 给的补偿方案是 Under the Hood：不给你规则，给你**规则作用在你身上的结果**。用户可以下载一份 JSON，看到上个自然月里自己账号和帖子被打了哪些影响可见性的标签。目前是试点，随机抽取、账号需满一年、上月至少发过 10 条帖。TechCrunch 那篇还提了个很实际的用法：非技术用户可以把 JSON 丢给 LLM，让它对着 GitHub 仓库解读。

**我的看法**：这个权衡我认可，但要点名边界。「代码 + 输出」能验证的是「规则怎么作用于我」，验证不了「规则是否公平地作用于所有人」——你只能看到自己的标签，看不到分布。真正的算法审计需要跨账号聚合数据，而 Under the Hood 按设计就是逐账号的。

---

## 7. Brazil2026ElectionFilter：算法里写死的法律

> "Application providers that use a recommendation system for users must exclude from the results the channels and profiles reported to the Electoral Court..."
> —— `home-mixer/filters/brazil_2026_election_filter.rs` 文件头注释（巴西选举法条文）

打开这个文件，是一份 665 个 user id 的硬编码列表——测试里直接断言 `BRAZIL_2026_ELECTION_USER_IDS.len() == 665`。每个 id 上面还带一行注释写着对应的 @用户名，文件里明说了原因：「User ids below are obfuscated; usernames are included for transparency.」甚至还有一句「OmarAzizSenador deleted his account at the time this code was written.」

逻辑很简单：这些账号的帖子不进 For You，除非读者明确关注了他们。README 对此的表述是：

> "A benefit of open-source is that you can see that changes like this exist, and exactly how they work."

**我的看法**：这条我完全同意 X，它可能是整次发布里最有说服力的论据。合规性内容限制每个平台都有，区别只在于你能不能看见。在这里，一部选举法变成了一个 Rust 文件、665 个整数和一句 `unless the viewer follows`。你可以质疑名单怎么来的、有没有超出法律要求、退出机制是什么——但你至少有具体的东西可以质疑。开源没让审查消失，它让审查变成了可以逐行 review 的对象。

---

## 写在最后

整体判断：这次发布在技术上是慷慨的，在制度上还差一段。

慷慨的部分很实在——多动作预测 + 显式权重的架构、排序与可见性分离、能真跑起来的训练代码、一份把参数变更讲清楚的复盘文档。对任何做推荐系统的团队都有直接参考价值，和「PR 稿式开源」不是一回事。

差的那段是可验证性。仓库三个多月更新一次，生产 checkpoint 不在，Under the Hood 只给逐账号视图，最关键的一点——公开的代码和线上跑的东西是否一致——依然只能靠信任。Coleman 说他们的梦想是「任何人都能评估帖子在平台上是怎么分发的，验证这是个公平的赛场」。代码是必要条件，不是充分条件。

不过 Meta、YouTube、TikTok 到今天一个权重都没公开过。评价这次发布的标准应该是「离可审计还有多远」，而不是「比闭源好多少」——但这两句都得说。

---

**参考来源**

- [xai-org/x-algorithm](https://github.com/xai-org/x-algorithm) —— README、`home-mixer/params/param.rs`、`home-mixer/scorers/ranking_scorer.rs`、`home-mixer/filters/brazil_2026_election_filter.rs`、`docs/BIDIRECTIONAL_BOOST_CHANGE.md`、`phoenix/QUICKSTART.md`（仓库状态与提交记录经 GitHub API 于 2026-08-15 核对）
- [X open sources its ranking algorithm, letting users see if they've been 'shadowbanned'](https://techcrunch.com/2026/08/13/x-open-sources-its-ranking-algorithm-letting-users-see-if-theyve-been-shadowbanned/) —— TechCrunch，2026-08-13
- [X open-sources For You ranking weights: a Report costs 468 likes of reach](https://explainx.ai/blog/x-algorithm-for-you-timeline-open-source-ranking-weights-august-2026) —— explainx.ai，2026-08-14（作为「权重被误读」的实例引用）
- [X shares new insights into transparency and shadowbanning](https://www.socialmediatoday.com/news/x-shares-new-insights-into-transparency-and-shadowbanning/827858/) —— Social Media Today
- [twitter/the-algorithm](https://github.com/twitter/the-algorithm) —— 2023 年 3 月版本，用于对照
