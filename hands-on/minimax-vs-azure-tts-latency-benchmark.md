# MiniMax T2A v2 vs Azure Neural TTS 实测：同一段中英文本，延迟差了 6~10 倍

> 📍 本文首发于 [MagicTools 码农早餐](https://tools.cooconsbit.com/zh/articles/minimax-vs-azure-tts-latency-benchmark?utm_source=github&utm_medium=referral)。镜像仓库仅收录预览，**[点此阅读全文 →](https://tools.cooconsbit.com/zh/articles/minimax-vs-azure-tts-latency-benchmark?utm_source=github&utm_medium=referral)**

我那套 [Sun Tzu at Work 视频工厂](/zh/articles) 里，TTS 是流水线上最不起眼、却最容易卡住整条线的一环。一集视频要合成十几段中英文旁白，如果每段等好几秒，`npm run bingfa tts` 这一步就能拖成一杯咖啡的时间。

工厂里同时接了两家：**MiniMax T2A v2**（境内 `api.minimaxi.com`）和 **Azure Neural TTS**（微软 `eastasia` 区）。切换只改一个 `TTS_PROVIDER` 环境变量，输出都被统一成 24kHz / 单声道 / 16bit 的 WAV，下游的 Remotion 合成完全不区分来源。用了一段时间，主观感受是 MiniMax 明显更快，但"明显"值多少、慢的到底是模型还是网络，一直没测过。这次把它钉死。

先给结论：**在我这台大陆直连的机器上，MiniMax 合成中位延迟 1.0~2.2 秒、快于实时 5~12 倍；Azure 中位 6~21 秒、勉强追平实时，长尾能抖到 27 秒。** 这不是"快一点"，是量级差距。但差距里有一半是网络的锅——这点后面用抓包说清楚。

## 问题背景：TTS 延迟不是玄学，它直接决定流水线节奏

视频工厂是批处理，不是实时对话，所以严格来说"首包延迟"对我不是刚需——我不需要边合成边播。真正卡我的是**整段音频返回的墙钟时间**：一集十几段旁白串行合成，每段慢 5 秒，累计就是一分多钟的干等。

更麻烦的是**方差**。如果延迟稳定，我还能预估;可要是这一段 3 秒、下一段 27 秒，重试逻辑和超时阈值就没法设——设短了误杀慢请求，设长了卡死不报错。所以这次实测我不光看中位数，min/max 一起记，专门盯长尾。

两家 API 的调用形态也不同，值得先说清楚：MiniMax T2A v2 我走的是**非流式同步**接口（`/v1/t2a_v2`，一次请求返回整段十六进制 PCM）；Azure 走的是 **REST 一次性合成**（SSML 进、`riff-24khz-16bit-mono-pcm` 出）。两边都是"发一次请求、拿整段音频"，度量口径一致，可比。

## 问题分析：延迟由两段构成，得分开归因

一次 TTS 合成的墙钟延迟，粗看是两段之和：

1. **服务端合成耗时**——模型把文本变成波形要多久。这是模型和推理架构决定的，跟你在哪儿无关。
2. **网络往返 + 传输**——请求打过去、音频拉回来的路上花的时间。这跟你的机器到服务端的物理距离强相关。

如果只测总延迟，MiniMax 快就快了，但你说不清是"模型快"还是"网络近"。而这两个归因导向完全相反的结论：如果是模型快，那换到海外服务器 Azure 也快不起来；如果纯是网络，那把服务部署到 Azure 同机房就能抹平差距。所以这次我**总延迟和网络 RTT 分开测**——总延迟用 5 轮取中位，网络单独用 `curl` 抓 connect/TLS 握手时间。

## 技术方案与选型：复现范围怎么划

这类"两家对比"最容易翻车的是量纲不统一——采样率不同、声道不同、量化位深不同，比出来的字节数和音质都没意义。所以我把范围严格收敛：

- **复用生产 TTS 抽象层**（`src/lib/bingfa/tts.ts`），不为测试单写调用代码。两家在这一层已经被统一成 **24kHz / 单声道 / 16bit PCM**，输出格式完全一致——这是可比的前提。
- **同一批文本**：中英各两段，短句 + 长段，取自《孙子兵法》原文（正好是工厂真实旁白的语料风格）。中文按去空白字符数、英文按词数计。
- **每 case 跑 5 轮**，取中位数抗抖动，同时保留 min/max 看长尾。
- **音色固定**：MiniMax 用工厂默认的 `speech-02-turbo`；Azure 中文 `zh-CN-YunxiNeural`、英文 `en-US-GuyNeural`（都是工厂在用的音色）。
- **明确排除音质主观评分**（排除项）。音质是主观维度，一个人打分没有统计意义；我只客观记录**格式是否对齐**（都 24kHz/mono/16bit）和**波形形态**。字节数我记了但**不当质量指标**——WAV 是无压缩定长格式，字节数只跟时长成正比，跟"好不好听"无关，谁把它当音质比谁就错了。
- **单价只做定性对照**（排除项）。MiniMax 境内计费口径几经调整、且分按 token 与按字符两种，我拿不到当前权威单价就不硬报数字，只给能确认的量级。

...

---

**[👉 继续阅读全文：MiniMax T2A v2 vs Azure Neural TTS 实测：同一段中英文本，延迟差了 6~10 倍](https://tools.cooconsbit.com/zh/articles/minimax-vs-azure-tts-latency-benchmark?utm_source=github&utm_medium=referral)**

更多文章：[tools.cooconsbit.com/articles](https://tools.cooconsbit.com/zh/articles?utm_source=github&utm_medium=referral)
