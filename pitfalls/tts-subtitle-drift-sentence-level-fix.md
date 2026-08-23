# AI 视频的字幕越到后面偏得越多：别再修对齐算法了，问题出在「先生成整段音频」这一步

> 📍 本文首发于 [MagicTools 码农早餐](https://tools.cooconsbit.com/zh/articles/tts-subtitle-drift-sentence-level-fix?utm_source=github&utm_medium=referral)。镜像仓库仅收录预览，**[点此阅读全文 →](https://tools.cooconsbit.com/zh/articles/tts-subtitle-drift-sentence-level-fix?utm_source=github&utm_medium=referral)**

## 现象

给耳鸣科普项目做自动化视频：文稿 → TTS 配音 → 素材混剪 → 烧字幕，管线基于 MoneyPrinterTurbo（下称 MPT），TTS 用 MiniMax speech-02-turbo。

第一版产出的视频，字幕前 30 秒和语音严丝合缝，一分钟后开始肉眼可见地慢半拍，到结尾已经错位好几秒。打开生成的 `.srt` 一看，更离谱：**尾部一串字幕条目的时间戳全是 `00:00:00`**——不是慢了，是对齐过程在半路直接断链了。

## 原方案在做什么

MPT 这类工具的默认做法是三段式：

1. 把**整篇文稿一次性**送 TTS，得到一个几分钟长的音频文件；
2. 用 whisper 把这个音频**转写回文字**，拿到带时间戳的词级结果;
3. 把转写结果和原稿做**相似度匹配**（MPT 里的 `correct()`），给原稿的每一句找到对应的时间区间，生成 srt。

逻辑上说得通：TTS 不返回时间戳，那就用语音识别把时间戳「找回来」。

## 断链的直接原因

中文文稿里写的是「4000 赫兹」，whisper 转写出来是「四千赫兹」；原稿有书名号、破折号，转写没有；语气词、停顿处的标点也对不上。相似度匹配靠的是两段文字长得像，**数字和标点的系统性差异会让相似度骤降**——匹配器在某一句上找不到足够像的候选，锚点一丢，后面整段都跟着失锚，最后退化成 `00:00:00` 的兜底值。

第一反应自然是修匹配：把数字归一化成同一种写法、剥掉标点再比、把相似度阈值调松一点。改完这一版，断链消失了，但慢性漂移还在——因为漂移和断链根本不是同一个问题。

## 真正的根因：时序信息走了一条会丢失的路

退一步看这个架构：文稿是唯一的事实来源，音频由它生成，字幕也由它生成。但**字幕的时间戳却不是从生成过程里拿的，而是靠「音频 → 转写 → 匹配」重建出来的**。这条重建链路上每一环都有误差：

...

---

**[👉 继续阅读全文：AI 视频的字幕越到后面偏得越多：别再修对齐算法了，问题出在「先生成整段音频」这一步](https://tools.cooconsbit.com/zh/articles/tts-subtitle-drift-sentence-level-fix?utm_source=github&utm_medium=referral)**

更多文章：[tools.cooconsbit.com/articles](https://tools.cooconsbit.com/zh/articles?utm_source=github&utm_medium=referral)
